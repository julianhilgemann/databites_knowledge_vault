
### 1. Dynamic Titles (The "[[Context]] Awareness" Layer)

**The Problem:** A chart shows "Sales by City." The user filters to "Berlin." The chart title still says "Sales by City."
**The Goal:** The title should say "Sales for Berlin."

#### Level 1: The Basic Switch
Use `SELECTEDVALUE` to grab the single selection.
```dax
Title Sales = 
"Sales Report for " & SELECTEDVALUE( 'Dim_Geography'[City], "All Cities" )
```

#### Level 2: The Multi-Select Handler (Senior Level)
What if the user selects Berlin, Paris, and London? `SELECTEDVALUE` returns the default ("All Cities"), which is a lie.

Use `CONCATENATEX` to list them, with a safeguard for "Too Many Selected."

```dax
Title Advanced = 
VAR CountSelected = COUNTROWS( VALUES( 'Dim_Geography'[City] ) )
VAR MaxDisplay = 3
RETURN
    "Sales for: " & 
    IF(
        CountSelected > MaxDisplay,
        "Multiple Cities (" & CountSelected & ")", 
        CONCATENATEX( VALUES('Dim_Geography'[City]), 'Dim_Geography'[City], ", " )
    )
```
**Pro Tip:** *"I prioritize [[Context|context]] in my reports. If a user prints a PDF, the slicers might not be visible. The Dynamic Title ensures the [[Context|context]] is baked into the visual."*

---

### 2. Dynamic Format Strings (The "Polymorphic" Measure)

**The Old Way:** using the `FORMAT()` function inside a measure.
*   *Problem:* `FORMAT([Sales], "$0")` converts the number to **Text**. You can no longer chart it, and sorting breaks (e.g., "$10" comes after "$1000").

**The New Way (Feature):** **Dynamic Format Strings**.
*   *Where:* In the Modeling tab, select the Measure $\to$ Dropdown "Format" $\to$ "Dynamic".
*   *What:* It allows you to write a separate [[DAX]] expression *just for the format*, while keeping the result as a **Number**.

#### Use Case: Currency Conversion
User selects "EUR" or "USD" in a slicer.

**1. Main Measure (The Number):**
```dax
Total Sales = SUM(Sales[Amount]) * SELECTEDVALUE(Currency[Rate], 1)
```

**2. Format String Expression (The Mask):**
```dax
VAR SelectedCode = SELECTEDVALUE(Currency[Code], "USD")
RETURN
    SWITCH( SelectedCode,
        "USD", "$#,0.00",
        "EUR", "#,0.00 €",
        "JPY", "¥#,0",
        "$#,0.00" -- Default
    )
```
**Result:** The chart axis changes symbols automatically, but the bars remain chartable numbers.

---

### 3. Field Parameters (The "Slicer for Axis")

**The Concept:** Allow users to change the **Axis** (X-Axis) or the **Measure** (Y-Axis) using a Slicer.

**How it works (Under the Hood):**
It creates a Calculated Table with a special function: `NAMEOF()`.
```dax
Parameter = {
    ("City", NAMEOF('Geography'[City]), 0),
    ("Country", NAMEOF('Geography'[Country]), 1),
    ("Category", NAMEOF('Product'[Category]), 2)
}
```
*   **`NAMEOF`**: This is critical. It tells Power BI "Don't just give me the string 'City', give me the **lineage pointer** to the actual column objects." This ensures RLS and cross-filtering work correctly.

#### Senior Best Practice: Grouping/Hierarchies
You can add columns to the Field Parameter table to group options (e.g., "Financial Measures" vs "Operational Measures").

**Pro Tip:** *"I use Field Parameters to reduce report bloat. Instead of building 5 duplicate charts (Sales by City, Sales by Country, etc.), I build one chart and give the user a Field Parameter to swap the view. This improves performance and maintainability."*

---

### 4. Conditional Formatting (The "Logic" Layer)

**The Junior Way:** Manually setting "Rules" in the UI (e.g., If > 0 then Green).
*   *Problem:* If you change your brand colors, you have to edit 50 charts manually.

**The Senior Way:** **"Field Value"** Formatting.
You create a specific [[DAX]] measure that returns a Hex Code.

#### Pattern: The "Brand Color" Measure
```dax
CF Sales Color = 
VAR SalesVar = [YoY Growth %]
RETURN
    SWITCH( TRUE(),
        SalesVar < -0.10, "#D0021B", -- Critical Red (Brand Hex)
        SalesVar < 0,     "#F5A623", -- Warning Orange
        "#417505"                    -- Good Green
    )
```
**Implementation:**
1.  Click the `fx` icon next to Bar Color.
2.  Style = **Field Value**.
3.  Select `[CF Sales Color]`.

**Pro Tip:** *"I centralize my color logic. I create dedicated CF (Conditional Formatting) measures returning Hex codes. This way, if Marketing updates the branding, I update one [[DAX]] measure, and the entire dashboard updates instantly."*

---

### Summary Table

| Feature | The Old Way | The Senior/Specialist Way | Why? |
| :--- | :--- | :--- | :--- |
| **Titles** | Static Text | `SELECTEDVALUE` & `CONCATENATEX` | Provides context (e.g., for PDF exports). |
| **Currency** | `FORMAT()` function | **Dynamic Format Strings** (Model Property) | Keeps the data numeric (sortable) but changes the symbol. |
| **Swapping Axis**| Bookmarks (Show/Hide) | **Field Parameters** | Reduces visual count, cleaner DOM, faster rendering. |
| **Colors** | UI Rules | **Field Value** (Hex Measure) | Centralized maintenance, theme compliance. |

### Mnemonic

**"Context, Colors, Controls."**
*   **Context:** Dynamic Titles (User knows where they are).
*   **Colors:** Hex Measures (Centralized Logic).
*   **Controls:** Field Parameters (User flexibility).