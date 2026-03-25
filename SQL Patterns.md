
Here are the **Top 5 [[SQL]] Patterns** for Analytics Engineers ([[dbt]] + Snowflake).

### 1. The "Top N" Per Group (Window Functions)

*   **The Question:** "Find the top-selling product *for each category*."
*   **Why HSE asks:** Standard retail analysis. "What's the #1 Lipstick? What's the #1 Frying Pan?"
*   **The Concept:** You can't just use `MAX()` because you want the *Product Name*, not just the *Sales Number*. You need `RANK()`.

**The Query:**
```sql
SELECT 
    Category,
    Product_Name,
    Sales,
    RANK() OVER (PARTITION BY Category ORDER BY Sales DESC) as Rank_In_Category
FROM Fact_Sales
-- In Snowflake, you can filter this immediately using QUALIFY (Insider Tip!)
QUALIFY Rank_In_Category = 1
```
*   **The Logic:** "I partition the data by Category, sort it by Sales, and assign a rank. Then I just keep Rank #1."

### 2. The "Growth" Calculation (LAG)

*   **The Question:** "Calculate the difference in sales compared to the previous day."
*   **Why HSE asks:** You work with T-1 data. They want to see the delta.
*   **The Concept:** Look at the row *behind* the current one.

**The Query:**
```sql
SELECT 
    Order_Date,
    Total_Sales,
    LAG(Total_Sales) OVER (ORDER BY Order_Date) as Previous_Day_Sales,
    (Total_Sales - LAG(Total_Sales) OVER (ORDER BY Order_Date)) as Daily_Change
FROM Daily_Sales_Table
```
*   **The Logic:** "I use `LAG` to peek at the previous row's value so I can calculate growth purely in [[SQL]] before it even hits PowerBI."

### 3. The "Cleaner" (Deduplication)

*   **The Question:** "We have a bug in the App event stream. Some clicks are recorded twice. How do you fix it?"
*   **Why HSE asks:** Clickstream data (Gen Z app) is messy.
*   **The Concept:** Assign a number to every row. If ID is same, row 1 is the original, row 2 is the duplicate.

**The Query:**
```sql
WITH Deduped AS (
    SELECT 
        Event_ID,
        Event_Timestamp,
        ROW_NUMBER() OVER (PARTITION BY Event_ID ORDER BY Event_Timestamp) as rn
    FROM Raw_Events
)
SELECT * FROM Deduped WHERE rn = 1
```
*   **The Logic:** "I use `ROW_NUMBER()` partitioned by the Event ID. Any row with a number greater than 1 is a duplicate, so I filter them out."

### 4. The "Bucketing" (CASE WHEN)

*   **The Question:** "Group customers into 'High', 'Medium', and 'Low' value based on spend."
*   **Why HSE asks:** This is the exact equivalent of the [[DAX]] `SWITCH` pattern we discussed, but done in the database ([[dbt]]).
*   **The Concept:** Conditional logic.

**The Query:**
```sql
SELECT 
    Customer_ID,
    Total_Spend,
    CASE 
        WHEN Total_Spend > 1000 THEN 'High'
        WHEN Total_Spend > 500 THEN 'Medium'
        ELSE 'Low'
    END as Customer_Segment
FROM Dim_Customer
```

### 5. The "Aggregator" (DATE_TRUNC)

*   **The Question:** "We have timestamp data (YYYY-MM-DD HH:MM:SS) but we want to analyze Monthly cohorts."
*   **Why HSE asks:** Moving from Seconds (Live TV) to Months (Financial Reporting).
*   **The Concept:** Rounding the date down.

**The Query:**
```sql
SELECT 
    DATE_TRUNC('month', Order_Timestamp) as Sales_Month,
    SUM(Sales_Amount) as Total_Revenue
FROM Fact_Sales
GROUP BY 1 -- Group by the first column
```

---

### The "Snowflake Flex" (Use this to impress them)

Since they use **Snowflake**, there is one magic keyword that standard [[SQL]] doesn't have: **`QUALIFY`**.

If you write a query on the whiteboard, use `QUALIFY`.

*   **Old SQL Way:** You have to write a subquery (CTE) to calculate the Rank, and then select from that subquery where Rank = 1.
*   **Snowflake Way:**
    ```sql
    SELECT * 
    FROM Sales
    QUALIFY ROW_NUMBER() OVER (PARTITION BY Category ORDER BY Amount DESC) = 1
    ```
*   **What to say:** *"Since we're on Snowflake, I prefer using `QUALIFY` to filter window functions immediately. It keeps the code much cleaner than writing subqueries."*

### Summary for Tomorrow

1.  **Group By / Sum** (The basics).
2.  **Left Join** (Keep all rows from the left table, match what you can from the right).
3.  **Window Functions** (`RANK`, `LAG`, `ROW_NUMBER`).

If you blank on the syntax, just say:
*"I would use a Window Function here, likely partitioning by Category and ordering by Date, to grab the previous value."*

**Logic > Syntax.** You are ready. Go sleep!