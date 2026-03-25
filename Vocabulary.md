
### 1. Data Modeling & [[Kimball]] (The Architecture)

| Term | The "Specialist" Definition | The Pro Tip (Why it matters) |
| :--- | :--- | :--- |
| **Grain** | What a single row represents (e.g., "One line per order"). | "I always define the Grain first. If the grain mixes (Header vs Line), measures will double-count." |
| **[[Star Schema]]** | Fact table in center, Dimensions surrounding. | "I prefer Star over Snowflake because the **[[Vertipaq|VertiPaq]]** engine is optimized for single-hop [[Relationships|relationships]]." |
| **Surrogate Key** | An artificial integer key (1, 2, 3) used for joining. | "I hide natural keys (e.g., OrderID-String) and join on Integers to improve performance and save RAM." |
| **Conformed Dimension** | A dimension (e.g., Customer) shared by multiple Facts. | "This allows us to slice Sales and Returns by the exact same attributes without ambiguity." |
| **SCD Type 2** | Slowly Changing Dimension. Tracks history (New Row). | "I rely on the warehouse/[[dbt]] to handle Type 2 logic so Power BI just sees clean rows with a Surrogate Key." |
| **Degenerate Dimension** | A dimension attribute stored in the Fact (e.g., Order #). | "It doesn't need its own table, but I check its **[[Cardinality]]** to ensure it doesn't bloat the model." |
| **Bridge Table** | A table used to resolve Many-to-Many [[Relationships|relationships]]. | "I avoid the native PBI Many-to-Many setting; a Bridge Table is transparent and predictable." |
| **Referential Integrity** | Ensuring every Fact FK has a matching Dimension PK. | "If a match is missing, PBI creates a **(Blank)** row. I treat that Blank row as a high-[[Priority|priority]] data quality alert." |
| **Junk Dimension** | Combining low-[[Cardinality|cardinality]] flags (Yes/No columns) into one table. | "Instead of cluttering the Fact with 10 flag columns, I create one Junk Dimension to reduce table width." |
| **Role-Playing Dimension** | Using one physical table for multiple contexts (Order/Ship Date). | "I prefer using `USERELATIONSHIP` in [[DAX]] over importing the Date table 3 times, to keep the model clean." |

---

### 2. Power BI Engine & [[DAX]] (The Mechanics)

| Term | The "Specialist" Definition | The Pro Tip (Why it matters) |
| :--- | :--- | :--- |
| **Evaluation [[Context]]** | The environment a formula runs in (Row vs. Filter). | "Understanding this is the difference between a Junior and a Senior dev." |
| **Row [[Context]]** | "The Finger." Iterating line-by-line. | "It iterates, it doesn't filter. Unless we use `CALCULATE` ([[Context Transition]])." |
| **Filter [[Context]]** | "The Box." The active filters from slicers/visuals. | "Every measure starts here. `CALCULATE` is the only way to modify it." |
| **[[Context Transition]]** | Turning Row Context into Filter Context. | "This is why we wrap expressions in `CALCULATE` inside iterators—to make sure the filter propagates." |
| **[[Cardinality]]** | The number of *unique* values in a column. | "**High Cardinality** is the #1 driver of model size. I optimize by splitting DateTime into Date and Time." |
| **[[Query Folding]]** | Pushing transformation logic back to [[SQL]]. | "I always check 'View Native Query' to ensure I haven't broken folding, which kills refresh speed." |
| **[[Vertipaq|VertiPaq]]** | The In-Memory Columnar Database engine of PBI. | "It compresses data by column. That's why wide [[Tables|tables]] are fine, but high unique values are bad." |
| **[[Calculation Groups]]** | Dynamic measure switching logic. | "I use these to replace 20 individual [[Time Intelligence]] measures (YTD/MOM) with one Calculation Item." |
| **Direct Lake** | Loading Parquet files from OneLake directly into PBI memory. | "It gives me Import-like speed with DirectQuery-like freshness, bypassing the legacy refresh process." |
| **Iterator Functions** | Functions ending in X (`SUMX`, `FILTER`). | "I use these when math must happen **row-by-row** (Price * Qty) before aggregating, to avoid 'Average of Averages' errors." |

---

### 3. [[SQL]] / Backend / [[dbt]] (The Pipeline)

| Term | The "Specialist" Definition | The Pro Tip (Why it matters) |
| :--- | :--- | :--- |
| **Execution Order** | The logic flow: FROM $\to$ WHERE $\to$ GROUP $\to$ SELECT. | "I know I can't filter by a SELECT Alias in the WHERE clause because the Alias hasn't been created yet." |
| **CTE** | Common Table Expression (`WITH...`). | "I use CTEs over nested subqueries for readability, similar to using `VAR` in [[DAX]]." |
| **Window Function** | `RANK()`, `LAG()`, `ROW_NUMBER()` `OVER...` | "I prefer calculating complex ranks or previous-row logic in [[SQL]] window functions before PBI loads it." |
| **Materialized View** | A view that is physically stored/cached. | "If the SQL query is too slow for DirectQuery, I materialize the view to speed up the read." |
| **ACID** | Atomicity, Consistency, Isolation, Durability. | "For transactional integrity in the source, I rely on ACID. For analytics, I accept eventual consistency." |
| **[[dbt]] Materialization** | How dbt saves the model (View, Table, Incremental). | "I use **Incremental** models for massive Fact [[Tables|tables]] to only process the delta (new rows) and save compute." |
| **Predicate Pushdown** | The database optimizing a query by filtering early. | "I write clean DAX to ensure Power BI can push the `WHERE` clause down to the SQL Source (Folding)." |

---

### 4. Data Viz, Dashboard & UI/UX (The Front End)

| Term | The "Specialist" Definition | The Pro Tip (Why it matters) |
| :--- | :--- | :--- |
| **Preattentive Attributes** | Visual cues (Color, Size) processed instantly by the brain. | "I use color sparingly as a preattentive attribute to highlight 'Bad' performance, not for decoration." |
| **Z-Pattern / F-Pattern** | The eye's natural scanning route. | "I place the most critical **BANs** (Big Angry Numbers) in the top-left, following the user's scanning pattern." |
| **Gestalt Principles** | How the brain groups elements (Proximity, Enclosure). | "I use whitespace and proximity to group related metrics, rather than drawing heavy boxes around everything." |
| **Data-Ink Ratio** | The proportion of "Ink" used for data vs. decoration. | "I remove gridlines and borders to maximize the Data-Ink ratio. Clutter creates cognitive load." |
| **Grid System** | A layout structure (usually 12 columns). | "I adhere to a strict 12-column grid and 8px spacing to ensure the dashboard feels professional and aligned." |
| **Affordance** | Design cues that tell a user how to interact. | "I make sure buttons look clickable (shadows/hover effects) to provide good affordance." |
| **[[IBCS]]** | International Business Communication Standards. | "I use standard notations—Solid bars for Actuals, Outline bars for Budget—so users instantly recognize the context." |

---

### 5. Governance & Modern Engineering (The Ecosystem)

| Term | The "Specialist" Definition | The Pro Tip (Why it matters) |
| :--- | :--- | :--- |
| **[[TMDL]]** | Tabular Model Definition Language (Text-based). | "I use [[TMDL]] to enable code-first development. It makes solving **[[git|Git]] Merge Conflicts** actually possible." |
| **PBIP** | Power BI Project (Folder structure). | "I save as PBIP so I can track changes in individual files, rather than committing a binary blob (.pbix) to [[git|Git]]." |
| **Golden Dataset** | Separating Data (Model) from Reporting (Thin Report). | "I never build reports inside the dataset file. I separate them to allow Self-Service without risking the Model integrity." |
| **Deployment Pipelines** | Dev $\to$ Test $\to$ Prod lifecycle management. | "I strictly use pipelines to ensure we never edit reports directly in the Production workspace." |
| **Endorsement** | Promoted vs. Certified content. | "I limit Certification rights to the CoE team so users know exactly which data is trusted." |
| **Fabric Lakehouse** | Storage for structured and unstructured data (Spark). | "I use the Lakehouse when I need to do heavy Python engineering before the data hits Power BI." |