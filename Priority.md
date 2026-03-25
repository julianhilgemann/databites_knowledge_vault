

### 1️⃣ Data Modeling (The "[[Kimball]]" Layer)

*The hiring manager wants to know you understand Structure.*

*   **[[Star Schema]]:** Center is Fact (Events), Surroundings are Dimensions ([[Context]]).
*   **The Grain:** ability to say, "This table is one row per [Order Line] per [Snapshot Date]."
*   **Conformed Dimensions:** Why `DimCustomer` must be the same table for `FactSales` and `FactReturns`.
*   **SCD Type 2:** History tracking. New row for changes. `ValidFrom` / `ValidTo`.
*   **Fact Types:** Transaction (Sales) vs. Snapshot (Inventory).
*   **[[Bridge [[Tables]]]]:** The clean solution for Many-to-Many.
*   **🚀 Specialist Nuance:** **"Bi-Directional vs. Single Direction."**
    *   *The Trap:* "Why not use Bi-Directional everywhere?"
    *   *The Answer:* "It creates ambiguity, kills performance, and allows filters to flow up-stream where they shouldn't. I always strive for Single Direction (1:*) flow."

### 2️⃣ [[DAX]] / Semantic Layer (The "Engine" Layer)

*The hiring manager wants to know you understand [[Context]].*

*   **Evaluation [[Context]]:**
    *   **Row Context:** The "Finger" (Line-by-line). Exists in Calculated Columns and Iterators (`SUMX`).
    *   **Filter Context:** The "Box" (Slicers/Visuals). Exists in Measures.
*   **[[Context Transition]]:** How `CALCULATE` turns Row Context into Filter Context.
*   **CALCULATE Modifiers:** `ALL` (ignore filters), `REMOVEFILTERS` (same), `USERELATIONSHIP` (use inactive join).
*   **Time Intel Patterns:** `SAMEPERIODLASTYEAR`, `DATEADD`, `TOTALYTD`.
*   **🚀 Specialist Nuance:** **"Safe Division."**
    *   *The Trap:* Writing `Sales / Units`.
    *   *The Answer:* "Always use `DIVIDE(Sales, Units, 0)` to handle divide-by-zero errors gracefully without crashing the visual."
*   **🚀 Specialist Nuance:** **"Auto Date/Time."**
    *   *The Trap:* "Do you use the automatic date hierarchy?"
    *   *The Answer:* "Absolutely not. It creates hidden [[Tables|tables]] for every date column and bloats the model. I always build a dedicated `DimDate` table."

### 3️⃣ Visualization (The "User" Layer)

*The hiring manager wants to know you understand Communication.*

*   **Layout:** High-level KPIs at top (BANs), Drill-through for details.
*   **Interactions:** Edit Interactions (stop charts from filtering each other if not needed).
*   **Drill-through:** Don't jam everything on one page. Create a path.
*   **🚀 Specialist Nuance:** **"Performance Analyzer."**
    *   *The Answer:* "If a visual is slow, I don't guess. I use the Performance Analyzer to see if it's the [[DAX]] query, the Visual rendering, or the 'Other' (model waiting)."

### 4️⃣ [[SQL]] / [[dbt]] (The "Architecture" Layer)

*The hiring manager wants to know you understand the Pipeline.*

*   **[[dbt]] Philosophy:** "Transform in the warehouse, not in Power BI."
*   **Layers:** Staging (Cleaning) -> Intermediate (Logic) -> Marts ([[Star Schema]]).
*   **[[SQL]] Skills:** Basic Joins, `GROUP BY`.
*   **🚀 Specialist Nuance:** **"[[Query Folding]]."**
    *   *The Concept:* Even if you use [[dbt]], you might use Power Query for prototyping. You must know that "[[Query Folding]]" pushes the work back to [[SQL]]. Breaking it (e.g., by adding an Index Column on a massive table) kills refresh times.

---

### 📝 Your "Cheat Code" Storyline

When they ask about your workflow, memorize this sentence. It covers 80% of the core requirements:

> "My philosophy is to push the heavy transformation logic upstream into **SQL/dbt** to build a clean **[[Kimball]] [[Star Schema]]**.
>
> In **Power BI**, I focus on building a clean Semantic Model—using **1-to-Many single-direction [[Relationships|relationships]]**, hiding surrogate keys, and using **[[DAX]] Measures** (not calculated columns) for business logic.
>
> Finally, for the **Dashboard**, I focus on performance and usability—avoiding clutter and using **Drill-throughs** to keep the interface clean."
