
# 1. DAX (The Engine)
*You must be able to write these blind.*

### A. The "Holy Trinity" ([[Context]] Manipulation)

These are the most important functions in the language.

1.  **`CALCULATE( Expression, Filter1, Filter2... )`**
    *   *Role:* The only function that changes Filter [[Context]].
    *   *Note:* Automatically performs [[Context Transition]] if the expression is a Measure.
2.  **`FILTER( Table, Condition )`**
    *   *Role:* An iterator. Returns a table.
    *   *By Heart:* `FILTER( ALL(Table), ... )` is the standard pattern to keep the table size small.
3.  **`ALL( Table_or_Column )`**
    *   *Role:* Removes filters. Returns the unique values.
4.  **`ALLEXCEPT( Table, Column_to_Keep )`**
    *   *Role:* "Remove all filters from this table *except* the one I explicitly list." Great for "% of Parent" calcs.
5.  **`ALLSELECTED( Table_or_Column )`**
    *   *Role:* Removes filters inside the query but keeps filters from the outside (Slicers).
6.  **`KEEPFILTERS( Expression )`**
    *   *Role:* Forces `CALCULATE` to intersect (AND) with existing contexts instead of overwriting them.
7.  **`REMOVEFILTERS( Table_or_Column )`**
    *   *Role:* Readable alias for `ALL()`. Use this when you don't need to return a table, just clear the [[Context|context]].

### B. Aggregators & Iterators

1.  **`SUM` / `AVERAGE` / `MIN` / `MAX`**
    *   *Role:* The basics. They only accept a Column Name.
2.  **`SUMX( Table, Expression )`**
    *   *Role:* Row-by-row math. `SUMX( Sales, Sales[Qty] * Sales[Price] )`.
    *   *Also:* `AVERAGEX`, `MINX`, `MAXX`.
3.  **`COUNTROWS( Table )`**
    *   *Role:* The most performant way to count.
4.  **`DISTINCTCOUNT( Column )`**
    *   *Role:* Counts unique values. Expensive on large columns.
5.  **`RANKX( Table, Expression, [Value], [Order], [Ties] )`**
    *   *Role:* Ranking logic.
    *   *Memorize:* `RANKX( ALL(Table), [Measure] )` is the standard pattern to rank against the whole group.

### C. Table Manipulation (Virtual [[Tables]])

1.  **`ADDCOLUMNS( Table, "Name", Expression )`**
    *   *Role:* Adds calculated columns to a virtual table.
2.  **`SUMMARIZE( Table, GroupBy_Col )`**
    *   *Role:* Groups data. **Only** use for grouping, not for adding math columns.
3.  **`VALUES( Column )`**
    *   *Role:* Returns unique values visible in current [[Context|context]]. Includes the Blank row (Invalid Relationship).
4.  **`DISTINCT( Column )`**
    *   *Role:* Returns unique values visible in current context. Excludes the Blank row.
5.  **`CROSSJOIN( Table1, Table2 )`**
    *   *Role:* Cartesian product.
6.  **`UNION( Table1, Table2 )`**
    *   *Role:* Stacks [[Tables|tables]] vertically.
7.  **`TREATAS( Table_Expression, Column_to_Filter )`**
    *   *Role:* Creates a **Virtual Relationship**. Connects two [[Tables|tables]] that are not physically connected in the model.

### D. [[Time Intelligence]]

1.  **`DATESYTD( Date_Column )`**
    *   *Role:* Calculates Year-to-Date.
    *   *Also:* `DATESMTD`, `DATESQTD`.
2.  **`SAMEPERIODLASTYEAR( Date_Column )`**
    *   *Role:* Shifts the current context back one year.
3.  **`DATEADD( Date_Column, Number, Interval )`**
    *   *Role:* Generic shifting. `DATEADD(Dates[Date], -6, MONTH)`.
4.  **`DATESINPERIOD( Date_Column, Start_Date, Number, Interval )`**
    *   *Role:* Moving Averages. "Last 3 months including today."
5.  **`CALENDARAUTO()`**
    *   *Role:* Generates a date table automatically based on your data.

### E. Logical & Informational

1.  **`DIVIDE( Numerator, Denominator, [Alternate] )`**
    *   *Role:* Safe division (handles Divide by Zero).
2.  **`IF( Logical, Result_True, Result_False )`**
    *   *Role:* Basic logic.
3.  **`SWITCH( TRUE(), Condition1, Result1, ... )`**
    *   *Role:* Replacing nested IFs.
4.  **`COALESCE( Value1, Value2... )`**
    *   *Role:* Returns the first non-blank value. Faster than `IF(ISBLANK())`.
5.  **`ISINSCOPE( Column )`**
    *   *Role:* Checks if a column is being used to group the visual. Essential for Matrix hierarchies.
6.  **`SELECTEDVALUE( Column, [Default] )`**
    *   *Role:* Returns the value if only one is selected.
7.  **`USERELATIONSHIP( Col1, Col2 )`**
    *   *Role:* Activates an inactive dotted-line relationship.

---

# 2. [[SQL]] (The Source)

*You need to show you can prep data before it hits Power BI.*

### A. The Order of Execution (Know this sequence!)
1.  **`FROM` / `JOIN`**
2.  **`WHERE`**
3.  **`GROUP BY`**
4.  **`HAVING`**
5.  **`SELECT`**
6.  **`ORDER BY`**
7.  **`LIMIT` / `TOP`**

### B. Joins & Unions
1.  **`LEFT JOIN`** (Keep all Left, match Right)
2.  **`INNER JOIN`** (Intersection only)
3.  **`UNION ALL`** (Stack data, keep duplicates, high performance)

### C. Window Functions (The Senior Skill)

*Syntax:* `FUNCTION() OVER (PARTITION BY x ORDER BY y)`
1.  **`ROW_NUMBER()`**
    *   *Use:* Deduplication. "Keep row where row_number = 1".
2.  **`RANK()` / `DENSE_RANK()`**
    *   *Use:* Leaderboards.
3.  **`LAG(Col, 1)` / `LEAD(Col, 1)`**
    *   *Use:* Previous row value (for Month-over-Month logic in [[SQL]]).

### D. Transformation
1.  **`CASE WHEN x THEN y ELSE z END`** (The [[SQL]] version of IF/SWITCH)
2.  **`COALESCE(Col, 0)`** (Handle Nulls)
3.  **`CAST(Col AS Type)` / `CONVERT`** (Fix [[Data Types|data types]])
4.  **`STRING_AGG` / `LISTAGG`** (Combine rows into a comma-separated list).

---

# 3. Power Query / M (The ETL)

*You don't need to write these from scratch usually, but you must know what to type in the "Custom Column" box.*

1.  **`Text.Upper` / `Text.Lower` / `Text.Proper`** (Capitalization)
2.  **`Text.Select`** (Keep only numbers: `Text.Select([Col], {"0".."9"})`)
3.  **`Text.Split`** (Split string by delimiter)
4.  **`Date.Year` / `Date.Month` / `Date.EndOfMonth`**
5.  **`DateTime.LocalNow()`** (Current refresh time)
6.  **`if [Col] > 10 then "High" else "Low"`** (Note: M is lowercase `if`, `then`, `else`).
7.  **`try ... otherwise ...`** (Error handling).

---

# 4. Fabric / [[dbt]] Specifics (The Modern Stack)

1.  **`is_incremental()`** (Jinja macro for [[dbt]])
    *   *Use:* `{% if is_incremental() %} WHERE date > (select max(date) from {{ this }}) {% endif %}`
2.  **`ref('model_name')`** (Jinja)
    *   *Use:* Referencing other tables in [[dbt]].
3.  **`source('source_name', 'table_name')`** (Jinja)
    *   *Use:* Referencing raw data in dbt.

---

### Mnemonic Strategy for practical application

If you get stuck, remember **"C.A.L.M."**:
*   **C**alculate (Do I need to change filters?)
*   **A**ggregate (Do I need to Sum/Count?)
*   **L**ogic (Do I need IF/SWITCH?)
*   **M**odify (Do I need ALL/REMOVEFILTERS?)
