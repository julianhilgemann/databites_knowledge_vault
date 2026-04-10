
### 1. `SELECTEDVALUE` (The Safe "Getter")

**Definition:**
"If there is exactly **one** value visible in this column, give it to me. If there are multiple values (or zero), give me a default."

**The Syntax:**

`SELECTEDVALUE( Table[Column], "Alternative Result" )`

**Why use it? (Best Practice):**

Before `SELECTEDVALUE` existed, we had to write:
`IF( HASONEVALUE(Column), VALUES(Column), "Multiple" )`
`SELECTEDVALUE` is just syntactic sugar (a shortcut) for that pattern.

**Primary Use Case: Dynamic Titles**

You want a Chart Title to change based on the Slicer.
```dax
Title Measure = 
"Sales Report for: " & SELECTEDVALUE( Dim_Geography[City], "All Cities" )
```
*   If user selects "Berlin" $\to$ "Sales Report for: Berlin"
*   If user selects nothing (or 2 cities) $\to$ "Sales Report for: All Cities"

**Warning:**

*   **Q:** "What does `SELECTEDVALUE` return if the column is filtered to 2 items?"
*   **A:** "It returns BLANK (or the alternate result you provided). It only works if **one single distinct value** is visible."

---

### 2. The "Am I Filtered?" Hierarchy

There are three functions that sound similar but act differently. Knowing the difference is a "Senior Developer" trait.

#### A. `ISFILTERED( Column )` (Direct Filter)

**Question:** "Did the user **directly** interact with this specific column?"
*   **Scenario:** User clicks a "Color" slicer.
*   `ISFILTERED(Product[Color])` = **TRUE**.
*   `ISFILTERED(Product[ID])` = **FALSE** (The user didn't touch ID, they touched Color).

#### B. `ISCROSSFILTERED( Table/Column )` (Indirect Filter)

**Question:** "Is this table getting filtered by **anything**?"
*   **Scenario:** User clicks a "Color" slicer.
*   `ISCROSSFILTERED(Product[Color])` = **TRUE**.
*   `ISCROSSFILTERED(Product[ID])` = **TRUE**. (Because filtering Color reduces the list of IDs).
*   `ISCROSSFILTERED(Fact_Sales)` = **TRUE**. (Because filters flow downhill).

#### C. `ISINSCOPE( Column )` (The Matrix/Hierarchy King)

**Question:** "Is this column currently providing the **grouping** level in the visual?"
*   **[[Context]]:** This was invented specifically for Matrix visuals and Hierarchies.

**The "Ratio" Problem (The #1 Use Case):**
Imagine a Matrix: **Year** $\to$ **Quarter** $\to$ **Month**.
You want a measure that calculates: *Sales / Parent Total*.

*   If you are on the **Month** row, you want `Sales / Quarter Sales`.
*   If you are on the **Quarter** row, you want `Sales / Year Sales`.

**The Solution:**
```dax
Dynamic Ratio = 
SWITCH( TRUE(),
    ISINSCOPE( Date[Month] ),   DIVIDE( [Sales], [Sales QTD] ), -- If we are at Month level
    ISINSCOPE( Date[Quarter] ), DIVIDE( [Sales], [Sales YTD] ), -- If we are at Quarter level
    BLANK()
)
```

---

### 3. `HASONEVALUE` vs. `ISINSCOPE` (The Total Row Check)

In visuals, the **Total Row** does not have a filter on the grouping column.

**The Scenario:**
You have a table of **Products** and a measure `[MyMeasure]`.
You want to hide the measure on the Grand Total line because the math doesn't make sense there.

**Option A: `HASONEVALUE` (The Old Way)**
```dax
IF( HASONEVALUE(Product[Name]), [MyMeasure], BLANK() )
```
*   **Behavior:** Checks if there is only one product visible.
*   **Risk:** If you filter the page to show *only one single product*, the **Grand Total** row will technically also have only one value visible. The Total will appear, which might confuse the user.

**Option B: `ISINSCOPE` (The Best Practice)**
```dax
IF( ISINSCOPE(Product[Name]), [MyMeasure], BLANK() )
```
*   **Behavior:** Checks if the `Product[Name]` column is actively grouping the row.
*   **Result:** In the Grand Total row, no column is grouping the data (it's an aggregation of everything). `ISINSCOPE` returns FALSE, hiding the value reliably, even if only one product is selected.

---

### Summary Cheat Sheet

| Function | What it asks | Best Use Case |
| :--- | :--- | :--- |
| **`SELECTEDVALUE`** | "Give me the value if it's unique." | **Harvesting Slicers**, Dynamic Titles, Conditional Formatting rules. |
| **`ISINSCOPE`** | "Is this column creating the current row grouping?" | **Matrix Hierarchies**, Calculating % of Parent, Hiding Total Rows. |
| **`ISFILTERED`** | "Did the user touch this specific slicer?" | showing/hiding visuals based on selection ("Please select a country to see details"). |
| **`ISCROSSFILTERED`**| "Is this table filtered by *anything*?" | Performance tuning, checking if a table is reduced. |
| **`HASONEVALUE`** | "Is there only 1 item in the list?" | Calculating specific logic for a single item (mostly replaced by `SELECTEDVALUE` and `ISINSCOPE` now). |

### A "Parameter Harvesting" Example (Best Practice)

If they ask: **"How do you make a measure change based on a user selection?"** (e.g., Switch between "Revenue" and "Profit").

**Your Answer:**
1.  "I create a disconnected table (a Parameter table) with rows: 'Revenue' and 'Profit'."
2.  "I use `SELECTEDVALUE` to grab the user's choice."
3.  "I use a SWITCH statement to change the measure logic."

```dax
Selected Metric = 
VAR UserChoice = SELECTEDVALUE( Parameter[MetricName], "Revenue" )
RETURN
    SWITCH( UserChoice,
        "Revenue", [Total Revenue],
        "Profit",  [Total Profit],
        [Total Revenue] -- Default
    )
```