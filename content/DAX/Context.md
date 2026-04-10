
### 1. Row Context ("The Current Line")

**Definition:**
Row context is the ability of the engine to know "which row I am currently looking at."

**Where does it exist?**
1.  **Calculated Columns:** By default. If you write `[Price] * [Quantity]` in a new column, [[DAX]] knows to grab the price *from this row* and quantity *from this row*.
2.  **Iterator Functions:** Functions that loop (e.g., `SUMX`, `FILTER`).

**The Hard Rule:**
> **Row Context does NOT filter the data model.**

**The Warning:**
*   **Scenario:** You are in a [[Calculated Column]] in the `Product` table. You write `SUM(Sales[Amount])`.
*   **Result:** You get the **Grand Total** of all sales repeated on every single row.
*   **Why?** You have a *Row Context* (you are on the "Red Hat" row), but you have no *Filter Context*. The `Sales` table doesn't know you want to filter by "Red Hat". It just sees "Sum everything."

---

### 2. Filter Context ("The View")

**Definition:**
Filter context is the set of active filters currently applied to the data model. It restricts the [[Tables|tables]] before any math happens.

**Where does it come from?**
1.  **Visuals:** Rows, Columns, Axis, Slicers.
2.  **External:** Filter Pane, Slicers on the page.
3.  **Internal:** `CALCULATE` modifiers inside a measure.

**The Behavior:**
It flows **downhill** (Relationship propagation).

---

### 3. [[Context Transition]] ("The Magic Switch")

**Definition:**
[[Context Transition]] is the process of transforming the **Row Context** (current row values) into an equivalent **Filter Context**.

**The Trigger:**
The function **`CALCULATE`** (and `CALCULATETABLE`).

**Revisiting the Warning:**
*   **Scenario:** You are in a [[Calculated Column]] in `Product`.
*   **Code:** `CALCULATE( SUM(Sales[Amount]) )`.
*   **Result:** You get the correct sales amount **for that specific product**.
*   **What happened?**
    1.  `CALCULATE` looked at the current row (Row Context).
    2.  It saw: `ID=100`, `Name="Red Hat"`.
    3.  It **transitioned** these values into filters: `Filter(Product, ID=100 AND Name="Red Hat")`.
    4.  These filters propagated to the `Sales` table.
    5.  `SUM` ran on the filtered `Sales` table.

---

### 4. Iterator Functions ("The Loops")

**Definition:**
Functions that create a programmable Row Context. They iterate through a table row by row, perform a calculation, and then aggregate the result.

**The Syntax:**
Usually end in **X**. (`SUMX`, `AVERAGEX`, `MINX`, `MAXX`, `CONCATENATEX`).
*   *Exception:* `FILTER` is also an iterator. `ADDCOLUMNS` is an iterator.

**The Standard Pattern:**
`Function( <Table to Loop Over>, <Expression to Run on Each Row> )`

**Common Question:** "Why use `SUMX` instead of `SUM`?"
*   **Answer:** `SUM` aggregates a single column. `SUMX` allows for row-by-row logic *before* the aggregation.
*   **Example:**
    *   `SUM(Sales[Amount])` $\to$ Adds up the column.
    *   `SUMX(Sales, Sales[Price] * Sales[Quantity])` $\to$ Multiplies P*Q for every row, *then* sums the results. (This avoids the "Average of Averages" math error).

---

### Summary Comparison Table

| Feature | Row Context | Filter Context |
| :--- | :--- | :--- |
| **Concept** | "I am looking at **this specific row** right now." | "I have filtered the table to **this subset** of data." |
| **Does it Filter?** | **NO.** (Crucial distinction). | **YES.** |
| **Created By** | Calculated Columns, Iterators (`SUMX`, `FILTER`). | Visuals, Slicers, `CALCULATE`. |
| **How to interact?** | Use `RELATED()` or `EARLIER()` to grab values. | Use `CALCULATE()` to change it. |

---

### The "Must-Know" Real-world Scenarios

#### 1. The "Maximum Value" Logic

**Question:** "How do I calculate the total sales of the *last date* in the current selection?"
**Solution:** You need an iterator and [[Context Transition|context transition]].
```dax
Sales Last Date = 
VAR MaxDate = MAX(Sales[Date]) -- 1. Find the date in the current Filter Context
RETURN
    CALCULATE(
        SUM(Sales[Amount]), 
        Sales[Date] = MaxDate      -- 2. Overwrite Filter Context with this date
    )
```

#### 2. The `FILTER` Function Trap

**Question:** "What is the difference between `CALCULATE(SUM(Sales), Sales[Color]='Red')` and `CALCULATE(SUM(Sales), FILTER(Sales, Sales[Color]='Red'))`?"

*   **Simple Version (`Sales[Color]='Red'`):**
    *   Fast. Optimized.
    *   Replaces any existing filter on Color with "Red".
*   **Iterator Version (`FILTER(...)`):**
    *   Slower (iterates the whole table).
    *   Used when logic is complex (e.g., `Price > Cost * 1.5`).
    *   *Key Distinction:* `FILTER` respects existing context unless you tell it not to (using `ALL`).

#### 3. Measure vs. [[Calculated Column]]
**Question:** "When should you use a Calculated Column vs. a Measure?"
**Answer:**
*   **Measure:** Always preferred. Calculated at query time (CPU). Dynamic based on user slicers.
*   **Calculated Column:** Calculated at refresh time (Storage/RAM). Static.
    *   *Use only if:* You need the result in an Axis, Slicer, or as a field in a Matrix row/column.

---

### Mnemonics for practical application

1.  **"Row Context is blind to Neighbors."**
    (It sees its own row, but it doesn't filter the other table unless you use `CALCULATE`).

2.  **"Context Transition is the Bridge."**
    (It is the only bridge from Row World to Filter World).

3.  **"Iterators are for Line Items."**
    (Use `SUMX` when you need to do math *per line item* before summing).

4.  **"Measures are Wrapped."**
    (Every Measure is automatically wrapped in a hidden `CALCULATE`. This means calling a Measure inside another Measure triggers Context Transition automatically).

### Final "Gotcha" Example for the Expert Level

**Question:** Why does `COUNTROWS(FILTER(Table, Measure > 100))` work, but `COUNTROWS(FILTER(Table, Column > 100))` acts differently?

**Answer:**
*   `Column > 100`: Just checks the value on that row. Simple.
*   `Measure > 100`: Because `Measure` has a hidden `CALCULATE`, it triggers **Context Transition**. For every row `FILTER` iterates over, it turns that row into a filter context, calculates the measure specifically for that row, and checks if it's > 100. This is powerful but expensive on performance.