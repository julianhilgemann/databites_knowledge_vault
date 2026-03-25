Here is an extensive **Skill Assessment & Practice Suite**.

These questions are categorized by domain and graded by difficulty. Use this to simulate the pressure of a practical assessment. I have included the **"Specialist Answer Key"** based on the curriculum we built.

---

### Phase 1: Architecture & Data Modeling
*The "Whiteboard" Stage.*

#### Q1: "We have a Sales Fact table and a Budget Fact table. Sales is daily, Budget is monthly. How do you model this in Power BI?"
*   **Difficulty:** Mid-Senior
*   **What they are testing:** Granularity mismatch & Modeling patterns.
*   **The Specialist Answer:**
    1.  **Do not** join Fact-to-Fact.
    2.  Create shared **Conformed Dimensions** (Date, Product).
    3.  Link Sales to Date on `DateID`.
    4.  Link Budget to Date on `MonthID` (or `FirstDayOfMonth`).
    5.  **Alternative:** Use **[[DAX]] `TREATAS`** (Virtual Relationship) if physical [[Relationships|relationships]] are messy.

#### Q2: "I have a requirement to filter Sales by 'SalesPerson'. However, one Order can be split among 3 Salespeople. How do you build this?"
*   **Difficulty:** Senior
*   **What they are testing:** Many-to-Many [[Relationships|relationships]].
*   **The Specialist Answer:**
    1.  Reject the "Native Many-to-Many" relationship setting (Ambiguity risk).
    2.  Build a **Bridge Table** (`Bridge_SalesPerson`) containing `OrderID` and `SalesPersonID`.
    3.  Set relationship: `Dim` $\to$ `Bridge` (1:*) and `Bridge` $\to$ `Fact` (1:*).
    4.  **Critical:** Explain that the relationship from Bridge to Fact requires **Bi-Directional** filtering (or `CROSSFILTER` in [[DAX]]) to propagate.
    5.  Mention **Allocation/Weighting** logic if the revenue shouldn't be double-counted.

#### Q3: "Why shouldn't I just load one massive flat table (OBT) into Power BI? It works in Excel."
*   **Difficulty:** Basic (but requires a technical "Why").
*   **What they are testing:** Knowledge of the [[Vertipaq|VertiPaq]] Engine.
*   **The Specialist Answer:**
    1.  **Compression:** [[Vertipaq|VertiPaq]] uses Dictionary Encoding. Repeating string values ("Customer Name") 10 million times bloats the model size in RAM.
    2.  **Performance:** [[Star Schema]] allows the engine to filter small dimension [[Tables|tables]] first, then propagate IDs to the Fact (efficient). OBT forces the engine to scan massive columns for simple filters.
    3.  **Usability:** A [[Star Schema]] is easier for users to navigate (Fields pane organization) compared to a flat table with 200 columns.

---

### Phase 2: [[DAX]] & The Engine
*The "Live Coding" Stage.*

#### Q4: "Explain what happens when I put the measure `[Total Sales]` inside a `FILTER` function."
*   **Difficulty:** Senior
*   **What they are testing:** [[[[Context]] Transition]].
*   **The Specialist Answer:**
    1.  `FILTER` is an iterator. It creates a **Row [[Context]]**.
    2.  Because `[Total Sales]` is a measure, it has a hidden `CALCULATE` wrapped around it.
    3.  This triggers **[[Context Transition]]**.
    4.  The current row in the iteration is transformed into a **Filter [[Context]]**.
    5.  The measure calculates Sales *specifically for that row*.

#### Q5: "Our 'Year-over-Year' calculation is slow. It takes 10 seconds to load. How do you debug and fix it?"
*   **Difficulty:** Senior
*   **What they are testing:** Performance Tuning & Tools.
*   **The Specialist Answer:**
    1.  **Tools:** Open **Performance Analyzer** to confirm it's the DAX (not the visual rendering). Then open **DAX Studio**.
    2.  **Metrics:** Check **Server Timings** (FE vs SE). High FE (Formula Engine) means complex logic/iterators. High SE (Storage Engine) means scanning too much data.
    3.  **Common Fixes:**
        *   Are we using `FILTER(Table)` instead of `KEEPFILTERS` or `FILTER(ALL(Column))`?
        *   Are there unnecessary **Context Transitions**?
        *   Can we move logic upstream to [[SQL]]?
        *   Are we using **Bi-Directional** [[Relationships|relationships]]?

#### Q6: "How do you handle 'Totals' in a Matrix visual that don't add up correctly?"
*   **Difficulty:** Mid-Level
*   **What they are testing:** Understanding of Evaluation Context.
*   **The Specialist Answer:**
    1.  Explain **why:** The Total row isn't a sum of the rows above; it's the calculation running in a context with *no filters* on that dimension.
    2.  **The Fix:** Use `SUMX( VALUES(Dim[Column]), [Measure] )` to force the engine to iterate through the dimension members and sum the results, mimicking the visual behavior.

---

### Phase 3: Engineering & Modern Stack (Fabric/[[git|Git]])
*The "Differentiation" Stage.*

#### Q7: "We have multiple developers working on the same PBI file. We keep overwriting each other's work. How do you solve this?"
*   **Difficulty:** Senior
*   **What they are testing:** PBIP, [[TMDL]], and [[git|Git]].
*   **The Specialist Answer:**
    1.  Stop saving as `.pbix`. Save as **Power BI Project (.pbip)**.
    2.  Enable **[[TMDL]]** to store the model definition as text (YAML-like) instead of JSON.
    3.  Use **[[git|Git]]** (Azure DevOps/GitHub). Create a repo.
    4.  Implement a **Branching Strategy**: Developers work on `feature/branches` and create Pull Requests (PRs) to merge into `main`.
    5.  [[TMDL]] allows us to resolve merge conflicts line-by-line.

#### Q8: "What is the difference between DirectQuery and Direct Lake in Fabric?"
*   **Difficulty:** Senior (Fabric specific).
*   **What they are testing:** Knowledge of the latest tech.
*   **The Specialist Answer:**
    *   **DirectQuery:** Translates DAX into [[SQL]] queries sent to the source. **Slow** (Translation layer + Network latency). Good for real-time.
    *   **Direct Lake:** A Fabric feature. The Power BI engine reads the **Parquet files** (Delta Lake) directly from OneLake into memory. **Fast** (Import-like speed) but without the need to "Refresh" the dataset.

#### Q9: "Explain [[Query Folding]]. Why is it non-negotiable?"
*   **Difficulty:** Core Competency.
*   **What they are testing:** Power Query efficiency.
*   **The Specialist Answer:**
    1.  **Definition:** Power Query translating M steps into a single [[SQL]] statement.
    2.  **Why:** If it breaks, the mashup engine pulls **all raw data** into local RAM to process it. This fails on large datasets (Timeout/OOM).
    3.  **Criticality:** **[[Incremental Refresh]]** *requires* folding to dynamically inject `RangeStart` and `RangeEnd` parameters into the SQL `WHERE` clause.

---

### Phase 4: Visualization & Governance
*The "Product" Stage.*

#### Q10: "A stakeholder asks for a report with 50 columns so they can export to Excel. What do you do?"
*   **Difficulty:** Behavioral / Soft Skills.
*   **What they are testing:** Requirement Elicitation & Pushback.
*   **The Specialist Answer:**
    1.  **The Pivot:** "I understand you need detailed data. However, Power BI is an *aggregated analysis* tool, not a CSV extract engine."
    2.  **The Solution:** "Let's build an **Exception Report**. We show KPIs to highlight *which* rows need attention. Then, I'll provide a 'Drill-through' button that takes you to a paginated report (or detailed grid) specifically for those rows."
    3.  **Governance:** Check if "Export to Excel" should even be allowed based on data sensitivity.

#### Q11: "How do you manage lifecycle (Dev/Test/Prod) in Power BI?"
*   **Difficulty:** Governance.
*   **The Specialist Answer:**
    1.  Use **Deployment Pipelines** (Fabric/Premium).
    2.  **Workspaces:** Separate workspaces for `[Dev]`, `[Test]`, and `[Prod]`.
    3.  **Golden Dataset:** Separate the **Data Workspace** (Semantic Models) from the **Reporting Workspace** (Thin Reports). This ensures analysts can build reports without breaking the core model.
    4.  **Parameters:** Use parameters to switch connection strings (Dev DB $\to$ Prod DB) automatically during deployment.

---

### Phase 5: The "AI Augmented" Curveball
*Testing your specific narrative.*

#### Q12: "You mentioned you use AI to code. How do I know your code is secure and accurate?"
*   **Difficulty:** Trust check.
*   **The Specialist Answer:**
    1.  **Security:** "I never paste real data (PII) into AI. I only paste the *schema* (column names/types) or anonymized mock data."
    2.  **Accuracy:** "AI is my 'Junior Developer.' It writes the boilerplate, but I act as the 'Senior Reviewer.' I validate the DAX against context rules (e.g., checking for `ALLSELECTED` bugs) and verify M code folds. I don't implement anything I don't understand."

---

### How to use this simulation:

1.  **Randomize:** Pick 3 questions at random.
2.  **Timer:** Give yourself 2 minutes per answer.
3.  **Record:** Record your voice answering them.
4.  **Review:** Listen back. Did you sound like you were guessing, or did you sound like a consultant? (Did you use the words "Grain", "Context", "Latency", "Governance"?)