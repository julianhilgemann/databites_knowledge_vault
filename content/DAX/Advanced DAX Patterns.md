
### 1. Virtual [[Tables]]: The "Average of Sums" Pattern

This is the most common advanced pattern.
*   **The Problem:** You want the "Average Sales per Customer."
*   **The Trap:** `AVERAGE(Sales[Amount])` gives you the average *Transaction* size, not the average *Customer* spend.
*   **The Solution:** You must virtually group by customer, sum their sales, and *then* average those sums.

```dax
Avg Sales Per Customer = 
VAR CustomerTable = 
    ADDCOLUMNS(
        VALUES( Customer[ID] ),      -- 1. Get distinct customers visible
        "@CustSales", [Total Sales]  -- 2. Calculate sales for each (Context Transition)
    )
RETURN
    AVERAGEX( 
        CustomerTable,               -- 3. Iterate over the virtual table
        [@CustSales]                 -- 4. Average the result
    )
```
**Pro Tip:** *"I always use `ADDCOLUMNS(VALUES(...))` instead of `SUMMARIZE` for these patterns. `SUMMARIZE` works for grouping, but adding calculations inside it creates inefficient query plans (clustering issues). `ADDCOLUMNS` is faster."*

---

### 2. Dynamic Segmentation (The "Burger King" Pattern)

**The Scenario:** You want to slice Customers by "High", "Medium", and "Low" spenders.
**The Twist:** A customer might be "High" in January but "Low" in February. Calculated Columns cannot handle this (they are static). You need a measure.

**The Setup:** Create a disconnected table `Segmentation` with columns: `Segment`, `Min`, `Max`.

**The Pattern:**
```dax
Customers in Segment = 
VAR SegmentMin = MIN( Segmentation[Min] )
VAR SegmentMax = MAX( Segmentation[Max] )

-- 1. Create the Virtual Table of Customer Totals
VAR CustomerStats = 
    ADDCOLUMNS(
        VALUES( Customer[ID] ),
        "@Sales", [Total Sales]
    )

-- 2. Filter that table based on the Disconnected Slicer
VAR FilteredCustomers = 
    FILTER(
        CustomerStats,
        [@Sales] >= SegmentMin && 
        [@Sales] < SegmentMax
    )

-- 3. Count the survivors
RETURN
    COUNTROWS( FilteredCustomers )
```
**Pro Tip:** *"I use the 'Disconnected Table' pattern to solve Dynamic Binning problems. It allows the user to re-classify data on the fly without reloading the model."*

---

### 3. RANKX (Ranking Correctly)

**The Problem:** `RANKX` often returns "1" for every row if you don't manage the [[Context|context]].
**The Rule:** You must rank against **All** items, not just the **Current** item.

```dax
Product Rank = 
IF(
    ISINSCOPE( Product[Name] ), -- Hide Rank on Subtotals
    RANKX(
        ALL( Product[Name] ),   -- The Context: Look at ALL products
        [Total Sales],          -- The Metric
        ,                       -- Value (Leave Empty)
        DESC,                   -- Order
        Dense                   -- Ties: 1, 2, 2, 3 (Not 1, 2, 2, 4)
    )
)
```

**Pro Tip:** *"I always prefer `Dense` ranking over `Skip` ranking because business users get confused when Rank #3 disappears due to a tie at #2."*

---

### 4. Window Functions (The "New" [[DAX]])

Released in late 2022/2023, these functions (`WINDOW`, `OFFSET`, `INDEX`) allow [[SQL]]-style windowing. They are faster and cleaner than the old `FILTER` methods.

#### A. Running Total (The Modern Way)
**Old Way:** `CALCULATE( [Sales], FILTER( ALL('Date'), 'Date'[Date] <= MAX('Date'[Date]) ) )` (Quadratic complexity, slow).
**New Way:**
```dax
Running Total = 
CALCULATE(
    [Total Sales],
    WINDOW(
        1, ABS,             -- Start: Absolute position 1 (Beginning)
        0, REL,             -- End: Relative position 0 (Current Row)
        ALLSELECTED('Date'),-- The Relation/Table
        ORDERBY( 'Date'[Date], ASC )
    )
)
```

#### B. Visual-Independent Comparison (`OFFSET`)

Compare current row to previous row *regardless of what the axis is*.

```dax
Previous Value = 
CALCULATE(
    [Total Sales],
    OFFSET(
        -1,                  -- Go back 1 step
        ALLSELECTED( Product[Name] ),
        ORDERBY( [Total Sales], DESC ) -- Sort by Sales, not Name!
    )
)
```
**Pro Tip:** *"I’ve migrated my Running Totals to use `WINDOW` functions. They reduce the complexity from Quadratic ($N^2$) to Linear ($N$), which fixed our performance issues on the 5-year trend charts."*

---

### 5. TREATAS (Virtual [[Relationships]])

**The Scenario:**
*   **Fact:** Sales (Daily).
*   **Fact:** Budget (Monthly).
*   **Problem:** You can't join them physically because the grain doesn't match.

**The Solution:** Propagate the filter virtually.

```dax
Budget for Selected Days = 
CALCULATE(
    [Total Budget],
    TREATAS(
        VALUES( 'Date'[MonthYear] ), -- Take Month from the Date Table context
        'Budget'[Month]               -- Apply it to the Budget table
    )
)
```
**Pro Tip:** *"I use `TREATAS` to handle mismatched granularities, like comparing Daily Sales vs Monthly Targets, without creating messy Many-to-Many physical [[Relationships|relationships]]."*

---

### 6. The "Magic" Totals Fix (SUMX over VISUAL)

**The Scenario:**
*   Row 1: 10 * 2 = 20
*   Row 2: 10 * 3 = 30
*   Total Row: 20 * 5 = **100** (Wrong!)
*   Expected Total: 20 + 30 = **50** (Sum of Rows)

**The Pattern:** Force the Total logic to iterate the dimension.

```dax
Correct Total = 
SUMX(
    VALUES( Product[ID] ), -- Force iteration over Products
    [Price] * [Qty]        -- Calculate per product
)
```
*   *Note:* In the Total Row, `VALUES` returns *all* products, so it iterates through them all and sums the result. In a single row, `VALUES` returns 1 product, so it runs once.

---

### Summary Table

| Pattern | Usage | The "Pro" Move |
| :--- | :--- | :--- |
| **Virtual [[Tables]]** | Complex Aggregations. | `ADDCOLUMNS` + `VALUES`. Never `SUMMARIZE` for math. |
| **Segmentation** | Dynamic Grouping. | Disconnected Table + `FILTER` logic. |
| **Rank** | Top N logic. | `ALL(Column)` to remove [[Context|context]]; `Dense` for ties. |
| **Windowing** | Running Totals / Moving Avg. | Use `WINDOW` / `OFFSET`. Much faster than legacy `FILTER`. |
| **Data Lineage** | Mismatched Grains. | `TREATAS` to creating virtual joins. |

### Mnemonic: **"A.R.T."**
*   **A**ddColumns (for Virtual [[Tables]]).
*   **R**ankX (for Ordering).
*   **T**reatAs (for Virtual [[Relationships]]).