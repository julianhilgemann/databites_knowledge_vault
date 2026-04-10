
The "Golden Rule" for Calculated Columns in a professional setting:

> **"Calculated Columns are computed during Data Refresh and stored in RAM. Measures are computed at Query Time (CPU). Therefore, I only use Calculated Columns when I need to SLICE by the result (Axis, Slicer, Filter)."**

If you don't need to put it on an Axis, make it a Measure.

Here are the specific best practices for when you *do* need to create them.

---

### 1. The `SWITCH( TRUE() )` Pattern

**[[Context]]:** Creating categorical groups based on logic (e.g., "High Value", "Low Value").
**The Old Way:** Nested `IF` statements. Hard to read, prone to bracket errors.
**The Best Practice:** Use `SWITCH` with `TRUE()` as the first argument.

```dax
-- Categorizing Order Size
Order Priority = 
SWITCH( TRUE(),
    Sales[Amount] > 5000, "High",
    Sales[Amount] > 2000, "Medium",
    Sales[Amount] > 0,    "Low",
    "Unknown" -- Default catch-all
)
```
*   **Why:** Readability. The engine evaluates top-to-bottom and stops at the first match.
*   **Tip:** "I always include a 'Default' result to catch data quality issues (like negative numbers)."

---

### 2. Binning (The Mathematical Approach)

**[[Context]]:** Grouping continuous numbers (Age, Price) into buckets (0-10, 10-20) for histograms.
**The GUI Way:** Right-click column $\to$ "New Group." (Creates a hidden internal logic).
**The Best Practice:** Use `FLOOR` logic in [[DAX]]. It gives you explicit control.

```dax
-- Creates bins: 0, 10, 20, 30...
Age Bin = FLOOR( Customer[Age], 10 )
```
*   **Refinement:** If you want text labels ("10-19"), keep the integer column for sorting, and create a second text column for display.
*   **Why:** Integer grouping compresses better than Text grouping ("10-20" repeats as a string).

---

### 3. Booleans (True/False vs. Yes/No)

**[[Context]]:** Creating flags like `IsLate`, `HasDiscount`.
**The Trap:** Returning Text `"Yes"` or `"No"`.
**The Best Practice:** Return standard `TRUE` / `FALSE` or `1` / `0`.

```dax
-- BAD (High RAM usage, slow filtering)
Is High Value = IF( Sales[Amount] > 1000, "Yes", "No" )

-- GOOD (1 bit storage, instant filtering)
Is High Value = Sales[Amount] > 1000
```
*   **Performance:** A Boolean column is the most efficient storage type in the [[Vertipaq|VertiPaq]] engine.
*   **Usability:** In the visual, you can rename True/False to "Yes/No" in the format pane if the user insists, but keep the model clean.

---

### 4. The "Sort By" Column

**Context:** You have a text column that sorts alphabetically but shouldn't (e.g., "Low", "Medium", "High").
**The Issue:** "High" comes before "Low" alphabetically.
**The Best Practice:** Always create a companion **Integer** column to control the sort.

1.  **Column 1 (Text):** `Priority = "High"`
2.  **Column 2 (Int):** `PrioritySort = 3`
3.  **Action:** Select `Priority` $\to$ Sort by Column $\to$ `PrioritySort`.

*   **Warning:** "Circular Dependency."
    *   If Column A relies on Column B to calculate, and you try to Sort B by A, the engine crashes. You cannot have A calculate from B while B sorts by A.

---

### 5. String Concatenation (The [[Cardinality]] Killer)

**Context:** Creating a composite key or a "Full Name" column.
**The Trap:** `FullName = [FirstName] & " " & [LastName]`
**The Best Practice:**
1.  **Do it in Power Query (ETL):** The compression happens *during* load. If you do it in [[DAX]], the engine stores the two original columns AND the uncompressed new column in RAM.
2.  **Avoid High [[Cardinality]]:** If you combine `[OrderID]` (1 million rows) with `[LineID]`, you create a column with 1 million unique text strings. This effectively disables the compression engine.

---

### 6. Replacing `RELATED` with Variables

**Context:** You need to do math using a number from a related table.
**Old Way:**
```dax
Total Cost = Sales[Quantity] * RELATED( Product[StandardCost] )
```
**Better Way (Variables):**
Technically the performance is similar, but Variables make the Row Context explicit.
```dax
Total Cost = 
VAR CurrentProductCost = RELATED( Product[StandardCost] )
RETURN
    Sales[Quantity] * CurrentProductCost
```
*   **Why:** If you need to use that cost twice (e.g., for logic checks), fetching it into a variable once is more efficient than calling `RELATED()` twice.

---

### 7. Handling Errors (`DIVIDE` and `COALESCE`)

**Context:** Calculating ratios in a column where the denominator might be 0, or dealing with Nulls.
**Best Practice:**

*   **Division:** `Margin % = DIVIDE( Sales[Profit], Sales[Amount], 0 )`
    *   (Never use `/` in a column, or the refresh will fail on a Divide By Zero error).
*   **Nulls:** `Clean Region = COALESCE( Customer[Region], "Unknown" )`
    *   (Replaces `IF( ISBLANK(...) )` pattern. Much faster and cleaner).

---

### 8. Use `DATEDIFF` for Age/Tenure

**Context:** Calculating how many days/years between dates.
**The Trap:** Simply subtracting dates (`[End] - [Start]`) returns a decimal number (days).
**The Best Practice:** Use `DATEDIFF` for precise control over the unit.

```dax
Days To Ship = DATEDIFF( Sales[OrderDate], Sales[ShipDate], DAY )
```
*   **Tip:** "I use `DATEDIFF` with the `SECOND` or `MINUTE` granularity carefully, as it produces high [[Cardinality|cardinality]] integers (millions of unique values), which hurts model size."

---

### Summary Table

| Feature | Bad Practice | Best Practice | Why? |
| :--- | :--- | :--- | :--- |
| **Categorization** | Nested `IF` | `SWITCH( TRUE()... )` | Readability & maintenance. |
| **Flags** | Text "Yes"/"No" | Boolean `TRUE`/`FALSE` | RAM storage & filter speed. |
| **Binning** | GUI Groups | `FLOOR()` or `ROUNDDOWN()` | Explicit control & integer storage. |
| **Sorting** | Manual ordering | Dedicated Integer Sort Column | Required for text like Jan/Feb or Low/High. |
| **Creation** | [[DAX]] | Power Query (M) | Do heavy row-by-row transformations in ETL, not RAM. |
| **Logic** | `EARLIER()` | Variables (`VAR`) | `EARLIER` is legacy and confusing. VAR is standard. |

### Final Mnemonic

**"Power Query for the Row, DAX for the Aggregate."**

If you are writing a Calculated Column that strictly transforms row-level data (like concatenating names or parsing strings), you should probably be doing it in Power Query, not DAX. DAX Calculated Columns are a "last resort" for modeling specific needs (like sorting or dynamic binning).