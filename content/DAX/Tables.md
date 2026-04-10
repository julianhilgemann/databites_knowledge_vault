
# 1. Calculated Tables (The "Physical" Tables)

### Definition

A Calculated Table is a table created using [[DAX]] that is **computed during Data Refresh** and physically stored in the model.

*   **Storage:** Consumes RAM and Disk space.
*   **Update Frequency:** Only updates when the dataset refreshes. It is **static** during user interaction (Slicers do NOT affect the content of a Calculated Table).

### Best Practice Use Cases

You should generally avoid Calculated Tables and prefer Power Query (M) or [[SQL]]. However, use them for:

1.  **Date Tables:** `Date = CALENDARAUTO()` (If you don't have a [[SQL]] Date dimension).
2.  **Role-Playing / Clones:** If you need a separate "Manager" table but only have "Employees," you can do: `Manager = 'Employee'`.
3.  **Parameter Tables:** Creating disconnected tables for "What-If" analysis.
    ```dax
    Discount Scenario = GENERATESERIES(0, 0.50, 0.05)
    ```
4.  **Debugging:** Great for testing a complex Virtual Table script to see what it actually returns.

### The Warning

**Q:** "Can I create a Calculated Table that changes based on the user's Slicer selection?"
**A:** "No. Calculated Tables are computed at **Process Time** (Refresh), not **Query Time**. To do dynamic table logic, you must use **Virtual Tables** inside a Measure."

---

# 2. SUMMARIZE & SUMMARIZECOLUMNS (The Grouping Engines)

These functions behave like `GROUP BY` in [[SQL]].

### A. `SUMMARIZE`

**Syntax:** `SUMMARIZE( Table, GroupBy_Column1, [GroupBy_Column2], ... )`

*   **Function:** Returns a table of unique combinations of the specified columns.
*   **The Golden Rule:** **"Use `SUMMARIZE` only for Grouping. Never for Math."**
    *   *Bad Practice:* `SUMMARIZE(Sales, Product[Color], "Total", SUM(Sales[Amount]))`.
    *   *Why?* It generates suboptimal query plans and technically creates "Clustering" issues in the engine.

### B. The "Best Practice" Pattern: `ADDCOLUMNS` + `SUMMARIZE`

If you need to Group By something *and* Add a Calculation (e.g., Sales by Color), do this:

```dax
Table = 
ADDCOLUMNS(
    SUMMARIZE( Sales, Product[Color] ),  -- Step 1: Group
    "Total Sales", [Total Sales]         -- Step 2: Math
)
```
*   **Why:** This preserves **Data Lineage** correctly and is highly optimized by the [[Vertipaq|VertiPaq]] engine.

### C. `SUMMARIZECOLUMNS` (The Modern Standard)

*   **[[Context]]:** This is the function Power BI generates natively when you drag visuals onto the canvas. It is faster and more efficient.
*   **Limitation:** Historically, it had issues inside complex **[[Context]] Transitions** (measures).
*   **Final Verdict:** "For querying ([[DAX]] Query View), I use `SUMMARIZECOLUMNS`. For complex measure logic inside `CALCULATE`, I prefer the robustness of `ADDCOLUMNS(SUMMARIZE(...))`."

---

# 3. Virtual Tables (The "Hidden" Tables)

### Definition
A Virtual Table is a temporary table created **inside a Measure** (usually in a Variable). It exists only for the milliseconds it takes to run the query and is then discarded.

*   **Storage:** None (CPU only).
*   **Power:** This is how you solve "Complex Aggregation" problems (e.g., "Average of Sums").

### The Workflow
1.  **Define:** Create the table in a `VAR`.
2.  **Iterate:** Pass that variable into an Iterator function (`SUMX`, `COUNTROWS`, `MAXX`).

### Real-World Example: "High Value Customers"

*Requirement: Count how many customers have bought more than $1,000.*

```dax
High Value Customer Count = 
-- 1. Create the Virtual Table
VAR CustomerStats = 
    ADDCOLUMNS(
        VALUES( Customer[CustomerID] ),  -- Get unique list of customers visible in current filter
        "CustSales", [Total Sales]       -- Calculate sales for each
    )

-- 2. Filter the Virtual Table
VAR BigSpenders = 
    FILTER( 
        CustomerStats, 
        [CustSales] > 1000 
    )

-- 3. Count the result
RETURN
    COUNTROWS( BigSpenders )
```

### Key Functions for Virtual Tables

| Function | Description |
| :--- | :--- |
| **`VALUES(Col)`** | Returns unique list of values (Respects current filters). Includes the "Blank" row if RI is broken. |
| **`DISTINCT(Col)`** | Returns unique list (Respects current filters). **Excludes** the "Blank" row (usually). |
| **`ALL(Col)`** | Returns unique list (Ignores filters). |
| **`CALCULATETABLE(...)`** | The table equivalent of `CALCULATE`. Filters a table expression. |
| **`CROSSJOIN(...)`** | Creates a Cartesian product of two tables. |
| **`UNION(...)`** | Stacks two tables vertically. |

---

# Summary Checklist

1.  **"Calculated Tables are Static."**
    (I try to do this upstream in SQL/[[dbt]]. I only use [[DAX]] tables for Date tables or Parameter generation).
2.  **"Virtual Tables are Dynamic."**
    (This is my primary tool for 'Second Level' aggregations, like calculating the Average Daily Sales per Store).
3.  **"The `ADDCOLUMNS` Pattern."**
    (I never add extension columns inside `SUMMARIZE`. I always wrap `SUMMARIZE` inside `ADDCOLUMNS` for performance and lineage safety).
4.  **"Debugging."**
    (If a measure is wrong, I copy the Virtual Table logic, create a physical Calculated Table with it in Desktop just to see the rows, debug it, then delete the physical table).