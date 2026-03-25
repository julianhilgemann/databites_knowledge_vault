
### 1. The Core Concept (The Elevator Pitch)
**Query Folding** is the ability of the Power Query (Mashup) Engine to convert your M code transformations into a **single native [[SQL]] statement** and execute it on the source server.

*   **Scenario:** You import a 100-million-row table but filter it to "Last 7 Days."
*   **With Folding:** Power BI sends `SELECT * FROM Table WHERE Date > '2025-01-01'` to the [[SQL]] Server. The server returns only the small slice of data.
*   **Without Folding:** Power BI downloads all 100 million rows, stores them in local temp memory, and *then* filters them. This kills performance and often causes timeout errors.

---

### 2. How to Check It (The Diagnostics)

#### A. The "Right-Click" Method (Basic)
Right-click the last step in Power Query $\to$ **"View Native Query."**
*   **Enabled:** Folding is happening. You can see the [[SQL]].
*   **Greyed Out:** Folding *might* have broken.
    *   *Note:* This is not 100% reliable. Some connectors (like Snowflake or OData) fold but don't show the native query in the UI.

#### B. The "Query Diagnostics" Method (Senior)
Use the **Tools** tab $\to$ **Start Diagnostics**.
*   Run the preview refresh.
*   Stop Diagnostics.
*   Look at the `Detailed` diagnostic report. Look for the `Data Source Query` column. If you see a massive SQL statement wrapping your logic, it folded.

---

### 3. The "Folding Killers" (What Breaks It)

You need to memorize these. These are the transformations that force the engine to switch from SQL to Local Processing.

1.  **Index Columns:** Adding a standard Index Column (1, 2, 3...) usually breaks folding because SQL doesn't have an inherent row order until you query it.
    *   *Fix:* Use `ROW_NUMBER()` in SQL (View) or Window Functions if supported.
2.  **Different Data Sources:** Merging a SQL table with an Excel file.
    *   *Result:* The engine pulls the SQL data locally to join it with the Excel data.
3.  **Specific Transformations:**
    *   `Capitalize Each Word` (SQL generally doesn't have a `PROPER()` function).
    *   `Split Column by Delimiter` (sometimes breaks depending on the driver).
    *   Buffer functions (`Table.Buffer`).
4.  **Changing [[Data Types]]:** Doing this *too early* can prevent folding if the target type doesn't map cleanly to a SQL type.

**The Golden Rule:** **"Push transformations upstream."**
If you need an Index or a Merge that breaks folding, do it in a SQL View or [[dbt]] model *before* Power BI touches it.

---

### 4. Advanced: Forcing Folding (`Value.NativeQuery`)

Sometimes you want to write your own SQL (e.g., a complex CTE) but still allow subsequent Power Query steps to fold into it.

**The Problem:** If you paste SQL into the "SQL Statement" box in the connector, folding often stops immediately.

**The Specialist Fix:** Use `Value.NativeQuery` with the `EnableFolding` flag.

```powerquery
let
    Source = Sql.Database("ServerName", "DBName"),
    RunSQL = Value.NativeQuery(
        Source, 
        "SELECT * FROM Sales WHERE Amount > 100", 
        null, 
        [EnableFolding=true] -- The Magic Flag
    )
in
    RunSQL
```
*   **Why:** This tells Power Query, "Trust me, this SQL is valid subquery material. You can wrap outer `SELECT`s and `WHERE` clauses around it."

---

### 5. Why it is Non-Negotiable

#### A. [[Incremental Refresh]]
[[Incremental Refresh]] **requires** Query Folding.
Power BI needs to dynamically inject `WHERE Date >= RangeStart AND Date < RangeEnd` into the SQL query. If it can't fold, it can't create the partitions, and it downloads the whole table every time.

#### B. DirectQuery
DirectQuery **is** Query Folding.
Every time a user clicks a slicer, Power BI generates a SQL query. If your M transformation doesn't fold, DirectQuery simply breaks (or performs terribly).

---

### Summary Checklist

1.  **"Folding is not binary."**
    (It can fold for the first 5 steps and break on step 6. I always push the "Folding Killers" to the very end of the script).
2.  **"I verify with Diagnostics."**
    (I don't just trust the greyed-out button; I check the actual query sent to the server).
3.  **"It enables Scalability."**
    (Without folding, we cannot use Incremental Refresh, which means we can't handle big data).

### Mnemonic: **"Fold or Fold"**
**"Fold** the query to the server, **or Fold** your cards and go home (because the report will time out)."