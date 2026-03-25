
# 1. The Logical Order of Operations (Execution Order)

This is the **#1 SQL Common Question**.
You write SQL from top to bottom (`SELECT` first), but the database engine executes it in a completely different order.

### The Order
1.  **FROM / JOIN** (Load the [[Tables]])
2.  **WHERE** (Filter Rows)
3.  **GROUP BY** (Aggregate)
4.  **HAVING** (Filter Aggregations)
5.  **SELECT** (Choose Columns & Create Aliases)
6.  **DISTINCT** (Remove Duplicates)
7.  **ORDER BY** (Sort)
8.  **LIMIT / TOP** (Cut off results)

### The "Alias Paradox" (Warning)
**Question:** "Why does this query fail?"
```sql
SELECT TotalAmount * 0.1 AS Tax
FROM Sales
WHERE Tax > 100  -- ERROR!
```
**Answer:**
*   The `WHERE` clause runs at **Step 2**.
*   The `SELECT` (which defines the alias `Tax`) runs at **Step 5**.
*   Therefore, when `WHERE` runs, the column `Tax` **does not exist yet**.
*   *Fix:* Repeat the calculation in the WHERE clause, or use a CTE/Subquery.

### Mnemonic
**"F**red **J**ones **W**ears **G**lasses, **H**aving **S**elected **D**istinct **O**rders **L**ately."
(FROM, JOIN, WHERE, GROUP BY, HAVING, SELECT, DISTINCT, ORDER BY, LIMIT).

---

# 2. Filtering: `WHERE` vs. `HAVING`

This distinction is critical for data manipulation.

| Feature | `WHERE` | `HAVING` |
| :--- | :--- | :--- |
| **Execution Step** | Step 2 (Early) | Step 4 (After Grouping) |
| **Filters...** | **Raw Rows** (Individual records). | **Groups** (Aggregated results). |
| **Can use Aliases?** | No. | Yes (in some DBs like MySQL/Postgres), No (in SQL Server/Oracle). |
| **Can use Aggregates?** | **No.** (`WHERE SUM(x) > 10` fails). | **Yes.** (`HAVING SUM(x) > 10` works). |

---

# 3. Joins (The Venn Diagram)

| Join Type | Description | Handling of Unmatched Rows |
| :--- | :--- | :--- |
| **INNER JOIN** | **Intersection.** Only rows that exist in BOTH [[Tables|tables]]. | Unmatched rows are dropped. |
| **LEFT JOIN** | **Master List.** All rows from Left table, matching rows from Right. | Unmatched Right rows become `NULL`. |
| **RIGHT JOIN** | Inverse of Left. Rarely used (just swap table order and use Left). | Unmatched Left rows become `NULL`. |
| **FULL OUTER JOIN**| **Union.** All rows from Both [[Tables|tables]]. | Lots of `NULLs` on both sides. |
| **CROSS JOIN** | **Cartesian Product.** Every row in A paired with every row in B. | **Danger:** 100 rows x 100 rows = 10,000 rows. |

**Best Practice:**
Always explicit write `INNER JOIN` or `LEFT JOIN`. Never just write `JOIN` (it implies Inner, but explicit is better).

---

# 4. Set Operations (`UNION`)

Combining two queries vertically (stacking them).

*   **`UNION`**: Stacks results AND **removes duplicates**. (Slower, does a sort).
*   **`UNION ALL`**: Stacks results AND **keeps duplicates**. (Faster, just appends).
*   **Best Practice:** Always use `UNION ALL` unless you specifically need to deduplicate. It is much more performant.

---

# 5. Window Functions (The Power User Tool)

This is how you manipulate data "in place" without collapsing rows like `GROUP BY` does.
**Syntax:** `FUNCTION() OVER (PARTITION BY [Col] ORDER BY [Col])`

### Common Functions for Engineering
1.  **`ROW_NUMBER()`**: Generates 1, 2, 3...
    *   *Use Case:* Deduplication. "Keep the row with `ROW_NUMBER() = 1` ordered by Date DESC."
2.  **`RANK()` vs `DENSE_RANK()`**:
    *   *Scenario:* A tie for 1st place.
    *   *Rank:* 1, 1, **3** (Skips 2).
    *   *Dense_Rank:* 1, 1, **2** (No gaps).
3.  **`LAG()` / `LEAD()`**:
    *   Look at the *previous* or *next* row.
    *   *Use Case:* Calculating "Month-over-Month Growth" in SQL.

---

# 6. NULL Handling in SQL

SQL logic uses **Three-Valued Logic** (True, False, Unknown).

*   **The Trap:** `WHERE column = NULL` will **always** fail. Nothing is "equal" to Null, not even another Null.
*   **The Fix:** `WHERE column IS NULL`.
*   **The Function:** `COALESCE(Column, 0)` -> Returns the first non-null value (similar to `IFNULL` or [[DAX]] `COALESCE`).
*   **Aggregation:** `SUM(Column)` ignores NULLs automatically. `COUNT(Column)` ignores NULLs. `COUNT(*)` counts rows regardless of NULLs.

---

# 7. CTEs (Common Table Expressions)

The modern way to write subqueries.

**The "Spaghetti" Subquery (Bad):**
```sql
SELECT * FROM (SELECT * FROM (SELECT ... ) )
```

**The CTE (Good):**
```sql
WITH CleanSales AS (
    SELECT ID, Amount FROM Sales WHERE Amount > 0
),
CustomerTotals AS (
    SELECT ID, SUM(Amount) as Total FROM CleanSales GROUP BY ID
)
SELECT * FROM CustomerTotals
```
*   **Benefit:** Readability. You can read it top-to-bottom.
*   **Reusability:** You can reference `CleanSales` multiple times in the final SELECT.

---

# Summary Checklist for SQL Data Manipulation

1.  **Filtering:** Filter early (`WHERE`) to reduce data size before joining/grouping.
2.  **Joins:** Check for "Fan-out" (Row explosion) when joining 1:Many.
3.  **Aggregates:** Remember that `COUNT(*)` counts rows, `COUNT(Col)` counts non-nulls.
4.  **Formatting:** Use CTEs instead of deep subqueries.
5.  **Windowing:** Use `ROW_NUMBER()` to identify duplicates.

### Mnemonic for Window Function Syntax
**"Pop Over"**
**P**artition, **O**rder, **P**erformed **Over**.
(`PARTITION BY` divides the data, `ORDER BY` sorts it, inside the `OVER()` clause).