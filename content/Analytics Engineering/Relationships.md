
| Relationship Type                           | [[Star Schema]] [[Context]]                                                                                     | PK (One Side) Expectation                                                              | FK (Many Side) Expectation                                                         | Handling of Blanks & Orphans                                                                                                                      | PBI Model Recommendation                                                                                       |
| :------------------------------------------ | :------------------------------------------------------------------------------------------------------ | :------------------------------------------------------------------------------------- | :--------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------ | :------------------------------------------------------------------------------------------------------------- |
| **1:N (One-to-Many)**                       | **The Standard.**<br>Connects **Dimension** (1) to **Fact** (N).                                        | **Unique.**<br>Must contain unique values. No duplicates allowed.<br>Ideally Non-Null. | **Non-Unique.**<br>Repeated values allowed (e.g., multiple sales for one Product). | **"Inferred Member"**<br>If Fact contains a value (or Null) missing in Dim, PBI creates a virtual **(Blank)** row in Dim to catch them.           | **Highly Recommended.**<br>This is the foundation of a high-performance [[Star Schema]].                           |
| **1:1 (One-to-One)**                        | **Rare.**<br>Usually "Vertical Partitioning" (Split [[Tables|tables]]) or separate [[Tables|tables]] for security boundaries. | **Unique.**<br>Both sides of the join must be unique.                                  | **Unique.**<br>Both sides of the join must be unique.                              | **Similar to 1:N**<br>If Table A filters Table B, and Table B has a row missing from A, it may result in a blank grouping depending on direction. | **Avoid.**<br>Usually better to merge these into a single table in Power Query for better performance.         |
| **N:N (Many-to-Many)**<br>*(Native/Direct)* | **Ambiguous.**<br>Connecting Fact to Fact (e.g., Sales vs Budget) or dirty Dimensions.                  | **Non-Unique.**<br>Neither column is required to be unique.                            | **Non-Unique.**<br>Neither column is required to be unique.                        | **Risky.**<br>Does **not** create "Inferred Members" (Virtual Rows). Unmatched rows usually drop out or cause confusing filter [[Context|context]].           | **Use with Caution.**<br>Often creates unpredictability. Requires careful [[DAX]] measures to calculate correctly. |
| **N:N (Many-to-Many)**<br>*(Bridge Table)*  | **The "Correct" M:N.**<br>Uses a bridge table to resolve two facts or specific complex logic.           | **Bridge ID is Unique.**<br>The Bridge acts as the "1" side for both original [[Tables|tables]].  | **FKs point to Bridge.**<br>Both original tables act as the "Many" side.           | **Standard 1:N Logic**<br>Applies the "Inferred Member" logic from the Bridge to the outer tables.                                                | **Recommended for M:N.**<br>Converts a complex M:N problem into two standard 1:N relationships.                |

---

### Deep Dive: Concepts & Mechanics

#### 1. PK (Primary Key) vs. FK (Foreign Key) in Power BI

While Power BI is not a relational database like [[SQL]] Server (it is a columnar analytic engine), it mimics these constraints during data load and relationship creation.

*   **The "One" Side (PK):** Power BI **strictly enforces uniqueness** here. If you try to create a 1:Many relationship and the "One" side has a duplicate value (e.g., Product ID 100 appears twice), Power BI will throw an error and refuse to create the relationship.
    *   *Blank Issue:* If your "One" side has multiple `nulls`, Power BI views `null` as a value. Two `nulls` = Duplicate. The relationship will fail.
*   **The "Many" Side (FK):** Power BI is permissive here. It allows duplicates and nulls.

#### 2. The "Inferred Member" (Handling Orphans & Nulls)

This is specific to **1:N** relationships (and by extension, properly modeled [[Bridge Tables|Bridge tables]]).

When the engine sees a value in the **FK (Fact)** column that does not exist in the **PK (Dimension)** column, it does not drop the data. It preserves the total sales/count by creating a bucket for them.

*   **Orphan Scenario:** Fact has `ProductID: 999`. Dimension stops at `100`.
    *   *Result:* Visual shows a row named **(Blank)** containing the data for 999.
*   **Null Scenario:** Fact has `ProductID: null` (or Blank).
    *   *Result:* Visual puts this data into that same **(Blank)** row.

**Note:** You cannot click/drill through the **(Blank)** header to see "What IDs are inside this?" nicely. It is a black box aggregation of everything unmatched.

#### 3. Why Native N:N is dangerous regarding Blanks

In a native Many-to-Many relationship (where you just drag a line between two non-unique columns in Power BI):
*   There is no "One" side to hold the "Master List" of items.
*   If Table A has "Red, Blue" and Table B has "Red, Green":
    *   "Red" matches.
    *   "Green" and "Blue" are orphans.
*   **The Behavior:** Unlike 1:N, Power BI usually does **not** generate a universal (Blank) bucket to catch the orphans in N:N. Depending on the filter direction (Single vs. Bi-directional), data simply disappears (is filtered out) because there is no master lookup list to anchor the query.

### Final Recommendation for Your Model

1.  **Aim for [[Star Schema]]:** Always try to shape data into **1:N** relationships.
2.  **Clean Keys:** Ensure the "One" side (Dimension) has **zero** duplicates and **zero** blanks/nulls in the ID column.
3.  **Handle Nulls in ETL:** If your Fact table (FK) has `nulls`, replace them in Power Query with `-1` or `0`, and ensure your Dimension table has a corresponding row for `-1` labeled "Unknown" or "No Category".
    *   *Why?* It prevents the generic "(Blank)" from appearing in visuals, replacing it with a deliberate "Unknown", which is much easier for end-users to understand.