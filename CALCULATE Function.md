This is the **Heart of [[DAX]]**. If you master this, you master the language.

# Part 1: The Anatomy of `CALCULATE`

### The Definition

`CALCULATE` evaluates an expression in a **modified Filter [[Context]]**.

### The Execution Order (The Algorithm)

When the engine sees `CALCULATE( [Sales], Filter1, Filter2 )`, it follows this strict 4-step process. **Memorize this.**

1.  **Snapshot:** It takes a snapshot of the *existing* Filter [[Context]] (from slicers/visuals).
2.  **Transition:** If a **Row [[Context]]** exists (e.g., inside `SUMX` or a [[Calculated Column]]), it creates a new Filter Context from the current row ([[Context Transition]]).
3.  **Modification:** It evaluates the Modifier functions (`ALL`, `REMOVEFILTERS`, `KEEPFILTERS`, `USERELATIONSHIP`).
    *   *Crucial:* These remove or alter filters *before* new ones are added.
4.  **Overwrite/Merge:** It evaluates the explicit filters (`Color = "Red"`).
    *   *Default:* **Overwrites** existing filters on that column.
    *   *With `KEEPFILTERS`:* **Intersects** (AND) with existing filters.
5.  **Run:** It calculates the Expression.

---

# Part 2: [[Context Transition]] (The Magic)

### What is it?
**[[Context Transition]]** is the process where the engine takes the **current row** (Row Context) and transforms every value in that row into a **Filter Context**.

### The "Hidden" Rule
Anytime you call a **Measure** inside [[DAX]], it is automatically wrapped in a hidden `CALCULATE()`.

### The Visual Example

Imagine a table `Product`:
| ID | Color | Price |
| :--- | :--- | :--- |
| 1 | Red | 10 |
| 2 | Blue | 20 |

**Scenario A: [[Calculated Column]] WITHOUT Transition**

Formula: `Col = SUM(Sales[Amount])`
*   **Context:** Row Context (I am on Row 1).
*   **Action:** `SUM` ignores Row Context. It only sees Filter Context. Since there are no slicers, it sees the whole table.
*   **Result:** **Grand Total** (e.g., $1,000) on every row.

**Scenario B: [[Calculated Column]] WITH Transition**

Formula: `Col = CALCULATE( SUM(Sales[Amount]) )`
*   **Context:** Row Context (I am on Row 1: ID=1, Color=Red, Price=10).
*   **Transition Triggered:** `CALCULATE` grabs the row.
*   **Transition Action:** It converts the row into a filter: `FILTER(Product, ID=1 AND Color=Red AND Price=10)`.
*   **Result:** Sales for **Red Item #1** only ($50).

---

# Part 3: The "Iterator" Trap (Best Practice)

This is the most common test of Senior [[DAX]] knowledge.

**The Question:**
"I have a measure `[Total Sales]`. I want to calculate the sum of sales for products that sold more than $100."

**Code A (Wrong):**
```dax
SUMX( Product, [Total Sales] > 100 )
```
*   *Why:* `SUMX` needs a number to sum, not a Boolean (True/False).

**Code B (The Trap):**
```dax
COUNTROWS(
    FILTER( Product, SUM(Sales[Amount]) > 100 )
)
```
*   *Why it fails:* `FILTER` creates a Row Context on `Product`. But `SUM(Sales[Amount])` **ignores** Row Context. It calculates the Grand Total ($1,000,000) for every single product.
*   *Result:* Returns the count of *all* products (because 1M > 100).

**Code C (The Solution - Context Transition):**
```dax
COUNTROWS(
    FILTER( Product, [Total Sales] > 100 )
)
```
*   *Why it works:*
    1.  `FILTER` iterates Product Row 1 (Red Hat).
    2.  It calls `[Total Sales]`.
    3.  `[Total Sales]` is a measure, so it has a hidden `CALCULATE`.
    4.  **Context Transition:** The row "Red Hat" becomes a Filter.
    5.  The measure calculates Sales *just* for Red Hat.
    6.  Check: Is $50 > $100? No. Skip.

---

# Part 4: The "Circular Dependency" Risk

This is a specific issue with Context Transition in **Calculated Columns**.

*   **The Scenario:** You create a Calculated Column that calls `CALCULATE` (or a measure).
*   **The Mechanism:** Context Transition turns **every column** in the row into a filter.
*   **The Problem:** If Column A depends on Column B, but Context Transition filters *all* columns (including A) to calculate B, the engine gets confused about which came first.
*   **The Fix:** Use `ALLEXCEPT` or `REMOVEFILTERS` inside the calculation to break the dependency loop on the columns you are currently building.

---

# Part 5: Practice Q&A [[Simulation]]

**Q: Does Context Transition happen automatically?**

**A:** "No. It only happens if `CALCULATE` (or `CALCULATETABLE`) is present. However, because every Measure has a hidden `CALCULATE`, calling a measure inside an iterator triggers it automatically."

**Q: Why is Context Transition expensive?**

**A:** "It forces the engine to switch from 'Batch Mode' (scanning columns fast) to 'Row Mode'. If you iterate 1 million rows, and perform Context Transition 1 million times, you are firing 1 million individual queries against the engine. I always try to filter the table *down* as much as possible before triggering transition."

**Q: What happens if I have duplicate rows in my dimension table during Context Transition?**

**A:** "Chaos. Context Transition filters by *value*, not by internal Row ID.
If you have two customers named 'John Doe' (different IDs) but your context transition filters on `Name="John Doe"`, it will aggregate data for **both** Johns into the result for the first John.
*Best Practice:* Always iterate over a table with a **Primary Key** (Unique ID) to ensure the transition creates a unique filter."

---

# Summary Visual Mnemonic

Think of **Row Context** as a **Flashlight** in a dark room.
*   It shines on one row at a time.
*   It sees the data on that row.
*   But the **Filter Context** (the room's thermostat) doesn't know the flashlight is on. It stays at the temperature set by the Slicers.

Think of **Context Transition** as pressing a button on the flashlight that **syncs it to the thermostat**.
*   Now, whatever the flashlight points at becomes the rule for the whole room.

### The "Senior" One-Liner

> **"Row Context iterates. Filter Context filters. Context Transition is using the current iteration step to modify the filter."**