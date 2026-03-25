
## Part 1: The Theory (The [[Kimball]] Methodology)

The Star Schema is a mature modeling technique (popularized by Ralph [[Kimball]]) designed for reading data (Analytics), not writing data (Transaction Processing).

### 1. The Structure

It is called a "Star" because it looks like one in a diagram:
*   **Center:** A single, massive table containing metrics.
*   **Points:** Smaller [[Tables|tables]] radiating outwards, containing descriptions.

### 2. The Components

#### A. Fact Table (The "Verbs")

Contains the **events** that occurred.
*   **Content:** Foreign Keys (integers) and Numerical Measures.
*   **Shape:** Long (Millions of rows) and Narrow (Few columns).
*   **Volatility:** Grows constantly.
*   **Examples:** `Sales`, `Clicks`, `Temperature_Readings`, `Inventory_Movements`.

#### B. Dimension Table (The "Nouns")

Contains the **[[Context|context]]** (Who, What, Where, When).
*   **Content:** Descriptive text, attributes, and one Primary Key.
*   **Shape:** Short (Few rows) and Wide (Many columns).
*   **Volatility:** Changes slowly.
*   **Examples:** `Customer` (Name, Address), `Product` (Color, Size), `Date` (Year, Month).

---

## Part 2: Power BI Implementation (The [[Vertipaq|VertiPaq]] Reality)

Power BI is optimized *specifically* for Star Schema. If you fight the schema, you fight the engine.

### 1. The "[[Filter Propagation]]" Concept

This is the physics of Power BI.
*   **The Law:** Filters flow from the **One** side (Dimension) to the **Many** side (Fact).
*   **The User Action:** A user clicks "2024" in a slicer (Dimension).
*   **The Engine Action:**
    1.  The engine filters the *Date Dimension* down to row ID 2024.
    2.  That ID travels down the relationship wire to the *Sales Fact*.
    3.  The *Sales Fact* is filtered to only show rows matching that ID.
    4.  The remaining rows are summed up.

### 2. Star Schema vs. Snowflake Schema

*   **Snowflake:** A Dimension filters another Dimension, which filters the Fact (e.g., `Product Category` -> `Product` -> `Sales`).
    *   *[[SQL]] Theory:* Good. It is "Normalized" (no repeated data).
    *   **Power BI Reality:** **Bad.**
        *   It forces the engine to traverse multiple [[Relationships|relationships]] (hops) to get to the data.
        *   It complicates [[DAX]] logic.
        *   **Best Practice:** Denormalize! Collapse the snowflake. Put `Category` and `SubCategory` directly into the `Product` table. [[Vertipaq|VertiPaq]] compression makes the storage cost of this redundancy negligible.

### 3. Star Schema vs. OBT (One Big Table)

*   **OBT:** A single flat sheet (Excel style) with all Customer names, Product names, and Sales in one grid.
*   *Pandas/Python Theory:* Often preferred for dataframes.
*   **Power BI Reality:** **Inefficient.**
    *   The engine can't compress a table effectively if text repeats millions of times.
    *   Simple questions ("Count of Customers") become slow `DISTINCTCOUNT` operations rather than instant row counts of a Dimension table.

---

## Part 3: Modeling Rules & Best Practices

### 1. [[Relationships]]: Always 1-to-Many
*   **Config:** `*:1` (Many-to-One).
*   **Direction:** **Single** (Filters flow from Dim to Fact).
*   **Avoid:** **Bi-directional (Both)** filtering.
    *   *Why?* It creates ambiguity. If filters flow upstream, Table A filters Table B, which filters Table C, which might filter Table A again. This kills performance and yields unpredictable numbers.

### 2. Fact Table Hygiene
*   **Hide the Keys:** The User should never see `Customer_ID` or `Product_Key`. Hide them in "Report View".
*   **Numbers Only:** Ideally, a Fact table contains *only* Keys and Numbers.
    *   *Exception:* If a text column has extremely high [[Cardinality|cardinality]] and is unique to the transaction (e.g., "Transaction Comment"), it can stay in the Fact, but it will bloat the file.

### 3. Dimension Hygiene
*   **Surrogate Keys:** As discussed previously, join on Integers.
*   **Grouping:** If you want to group by "Age", create an "Age Group" column in your Customer Dimension *before* loading. Don't calculate it on the fly in the Fact table.

### 4. The "Date" Dimension
Power BI requires a dedicated Date Dimension for [[Time Intelligence]] functions (`SAMEPERIODLASTYEAR`, `TOTALYTD`).
*   **Do not** rely on the Date column inside the Fact table.
*   **Do not** use Power BI's "Auto Date/Time" feature (it creates hidden, bloated [[Tables|tables]]).
*   **Do:** Connect a standard Calendar table to your Fact table on `DateKey`.

---

## Part 4: Handling Multiple Fact [[Tables]] (The "Galaxy" Schema)

Real-world projects often have multiple processes (e.g., *Sales* and *Budget*). This is called a **Galaxy Schema** (multiple Stars sharing Dimensions).

### The Challenge
*   *Sales* is at the **Day** level.
*   *Budget* is at the **Month** level.

### The Solution (Data Modeling)
1.  **Shared Dimensions (Conformed Dimensions):** Use the *same* `Date` table and `Product` table for both Facts.
2.  **Granularity Matching:**
    *   Connect `Date[DateID]` to `Sales[DateID]`.
    *   Connect `Date[MonthID]` to `Budget[MonthID]` (or use [[DAX]] to handle the grain mismatch).
3.  **Never join Fact to Fact:** Never try to connect *Sales* directly to *Budget*. Always go through the shared Dimensions.

---

## Summary: The "Perfect" Power BI Model

| Feature | Best Practice | Why? |
| :--- | :--- | :--- |
| **Schema Shape** | **Star** | Fastest performance; easiest [[DAX]]. |
| **Relationship** | **1:Many** | Native to the engine. |
| **Filter Direction** | **Single** (Dim $\to$ Fact) | Prevents ambiguity and circular logic loops. |
| **Key Type** | **Integer** (Surrogate) | Minimum RAM usage; fastest joins. |
| **Columns in Fact** | **FKs + Metrics** | Keep the table "Narrow" for compression. |
| **Columns in Dim** | **Attributes** | Keep the table "Wide" for filtering [[Context|context]]. |
| **Snowflaking** | **Collapse it** | Reduces join hops; simplifies the model. |
| **Null Handling** | **Replace with -1** | Prevents the "(Blank)" member; ensures referential integrity. |

### Final Engineering Thought
**"Push logic upstream."**
If you are writing complex DAX to [[Bridge Tables|bridge tables]] or clean up data, you have a modeling problem.
*   Join tables in [[SQL]]/Power Query.
*   Clean columns in [[SQL]]/Power Query.
*   Let the Model simply *exist* as a clean Star Schema so DAX can focus on math, not structural fixes.