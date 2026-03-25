

### 1. Variables (`VAR` / `RETURN`) - The #1 Optimization

If you only remember one thing, remember this.
**Why:**
1.  **Performance:** Variables are calculated **once** and stored in memory. If you use a variable 5 times in a `RETURN` statement, the engine only calculates it once. Without variables, it calculates it 5 times.
2.  **Readability:** It breaks complex logic into steps.
3.  **Debugging:** You can return a specific variable to test that part of the code.

**Bad:**
```dax
Total Ratio = 
DIVIDE( SUM(Sales[Amount]), CALCULATE(SUM(Sales[Amount]), ALL(Sales)) ) 
+ 
( SUM(Sales[Amount]) / 1000 )
-- The SUM(Sales[Amount]) is calculated twice!
```

**Good:**
```dax
Total Ratio = 
VAR CurrentSales = SUM(Sales[Amount])
VAR AllSales = CALCULATE(CurrentSales, ALL(Sales))
RETURN
    DIVIDE(CurrentSales, AllSales) + (CurrentSales / 1000)
```

---

### 2. Strict Naming Conventions (The "Pro" Signifier)

This is a visual cue that tells a stakeholder you are experienced.

*   **Measures:** Always use brackets only. `[Total Sales]`
*   **Columns:** Always include the table name. `Sales[Amount]`
*   **Why?** When reading code, you immediately know if [[[[Context]] Transition|[[[[Context]] Transition|[[[[Context]] Transition|context transition]]]]]] is happening.
    *   `CALCULATE( ... , Sales[Color] = "Red")` -> Filtering a **Column**.
    *   `CALCULATE( ... , [Total Sales] > 100)` -> Filtering a **Measure** (Triggers Context Transition).

---

### 3. `DIVIDE` vs. `/`

**The Rule:** Always use the `DIVIDE()` function.
**Why:** The `/` operator crashes (throws an error) if the denominator is zero. `DIVIDE` handles divide-by-zero gracefully (returns BLANK by default, or an alternate result).

**Pro Tip:** *"I use DIVIDE to ensure the visual doesn't throw 'Infinity' errors or crashes during data quality issues."*

---

### 4. Filter Columns, Not [[Tables]]

**The Trap:**
```dax
-- BAD
CALCULATE( [Total Sales], FILTER( Sales, Sales[Color] = "Red" ) )
```
**The Problem:** `FILTER(Sales)` loads the **entire** Sales table (all 50 columns) into memory to check one column.
**The Fix:**
```dax
-- GOOD
CALCULATE( [Total Sales], KEEPFILTERS( Sales[Color] = "Red" ) )
```
**Why:** This only touches the `Color` column dictionary. It is exponentially faster on wide [[Tables|tables]].

---

### 5. `COUNTROWS` vs. `COUNT`

**The Rule:** If you want to count how many transactions happened, use `COUNTROWS`.
*   **`COUNT(Sales[ID])`**: The engine has to check the ID column to see if it is BLANK.
*   **`COUNTROWS(Sales)`**: The engine just looks at the table metadata (internal statistics). It doesn't check any values. It is instant.

---

### 6. Avoid `IFERROR` and `ISERROR`

**The Issue:** These functions force the engine to calculate the expression, catch the error, and then run the alternative. It can break the optimized query plan (SE vs FE).
**The Fix:** Use defensive coding logic (like `DIVIDE` or `IF(ISBLANK())`) to prevent the error from happening in the first place.

---

### 7. [[Time Intelligence]]: Auto vs. Custom

**The Rule:** Never use "Auto Date/Time."
**The Pro Tip:** *"I disable Auto Date/Time immediately to prevent model bloat (hidden [[Tables|tables]]) and ensure I can use a standard 4-4-5 or Fiscal calendar using a dedicated Date Dimension."*

---

### 8. Measure Branching

**The Concept:** Build measures on top of measures. Don't rewrite logic.
1.  Base Measure: `[Total Sales] = SUM(Sales[Amount])`
2.  Branch 1: `[Sales YTD] = TOTALYTD([Total Sales], ...)`
3.  Branch 2: `[Sales Margin] = [Total Sales] - [Total Cost]`

**Why:** If the definition of "Sales" changes (e.g., exclude tax), you update **one** base measure, and the fix propagates to all 50 dependent measures automatically.

---

### 9. `HASONEVALUE` / `ISINSCOPE` (Total Row Logic)

**The Scenario:** You have a measure that subtracts two values. In the visual, the individual rows are correct, but the Grand Total line is wrong (it does the math on the total, rather than summing the individual results).
**The Fix:** Use `SUMX` or `ISINSCOPE` to detect if you are in the Total row.

```dax
-- Preventing a weird ratio on the Total Line
Safe Ratio = 
IF(
    HASONEVALUE(Dim_Product[Name]),
    [MyRatio],
    BLANK() -- Or a specific aggregation for the total
)
```

---

### Summary Checklist

If they ask: **"How do you optimize a slow report?"**
Your answer should be this workflow:

1.  **"Performance Analyzer:** I run this first to see if it's the [[DAX]] query or the Visual rendering."
2.  **"[[DAX]] Studio:** I copy the query there to check the Server Timings."
3.  **"Variables:** I check if the code is calculating the same thing multiple times."
4.  **"Iterators:** I check if `FILTER` is being used on a whole table instead of a column."
5.  **"[[Cardinality]]:** I check if we are filtering on high-[[Cardinality|cardinality]] columns (GUIDs) causing expensive joins."
6.  **"Weak [[Relationships]]:** I check if Bi-directional filters are causing ambiguity."