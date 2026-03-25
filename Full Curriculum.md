
## 1. Data Modeling & [[Kimball]] Architecture
### A. Theoretical Foundations

- [ ] **OLTP vs. OLAP**: Understand [[Normalization|normalization]] (3NF) vs. denormalization.
- [ ] **The [[Star Schema]]**: Central fact [[Tables|tables]] vs. surrounding dimension [[Tables|tables]].
- [ ] **The Snowflake Schema**: When to use it, why to generally avoid it in Power BI.
- [ ] **The "Grain"**: Defining what a single row represents (e.g., "One row per line item per order").
- [ ] **Conformed Dimensions**: Shared dimensions (Customer, Product) across multiple business processes.
- [ ] **The Bus Matrix**: Mapping business processes (Rows) to dimensions (Columns).

### B. Fact [[Tables]]

- [ ] **Transaction Facts**: Events at a specific point in time (e.g., Sales, Clicks).
- [ ] **Periodic Snapshot Facts**: Status at regular intervals (e.g., Daily Inventory, Monthly Balance).
- [ ] **Accumulating Snapshot Facts**: Process workflows with multiple timestamps (e.g., Order Placed -> Shipped -> Delivered).
- [ ] **Measure Types**:
    - [ ] Additive (Sales Amount).
    - [ ] Semi-additive (Inventory Balance - additive across regions, not time).
    - [ ] Non-additive (Ratios, Unit Prices).
- [ ] **Degenerate Dimensions**: Dimension attributes stored in the fact table (e.g., Order ID).

### C. Dimension Tables

- [ ] **Surrogate Keys vs. Natural Keys**: Why PBI prefers integer surrogate keys for performance.
- [ ] **Hierarchies**: Creating drill-down paths (Country -> State -> City).
- [ ] **Attributes**: Descriptive columns for slicing and dicing.
- [ ] **Slowly Changing Dimensions (SCD)**:
    - [ ] **Type 0**: Fixed.
    - [ ] **Type 1**: Overwrite (No history).
    - [ ] **Type 2**: Add new row (Row versioning with Start/End dates).

### D. Advanced Modeling Patterns

- [ ] **Date/Calendar Table**: Prerequisites (continuous dates, marked as Date Table).
- [ ] **Role-Playing Dimensions**: Handling multiple [[Relationships|relationships]] (Order Date vs. Ship Date) via active/inactive [[Relationships|relationships]].
- [ ] **Many-to-Many Handling**:
    - [ ] [[Bridge Tables]] (The standard [[Kimball]] approach).
    - [ ] Bi-directional filtering (The risky Power BI approach).
- [ ] **Bi-Directional [[Relationships]]**: Ambiguity risks and performance impact.

---

## 2. Power Query (M) & Data Transformation
### A. Core Concepts
- [ ] **ETL Methodology**: Extract (Source), Transform (Power Query), Load (Model).
- [ ] **[[Data Types]]**: Importance of explicit typing for engine optimization.
- [ ] **Parameters**: Managing environments (Dev/Test/Prod) and dynamic paths.

### B. Performance & "The Pro Level"
- [ ] **[[Query Folding]]**:
    - [ ] Definition: Pushing transformations back to the [[SQL]] source.
    - [ ] Breaking Folding: Index columns, changing types after filtering.
    - [ ] Verification: "View Native Query" option.
- [ ] **[[Incremental Refresh]]**: RangeStart and RangeEnd parameters.

---

## 3. [[DAX]] & Semantic Layer
### A. Evaluation [[Context]] (Crucial)

- [ ] **Row [[Context]]**: Concept of "Current Row" (Calculated columns, Iterators).
- [ ] **Filter [[Context]]**: Active filters from visuals, slicers, and RLS.
- [ ] **[[Context Transition]]**: How `CALCULATE` turns Row Context into Filter Context.

### B. Core Syntax & Calculations

- [ ] **Measures vs. Calculated Columns**: When to use which (Storage vs. CPU).
- [ ] **Aggregators**: `SUM`, `AVERAGE`, `COUNT`, `DISTINCTCOUNT`.
- [ ] **Iterators ("X" functions)**: `SUMX`, `AVERAGEX`, `RANKX`.
    - [ ] Logic: Iterate table -> Calculate expression -> Aggregate result.

### C. The `CALCULATE` Engine

- [ ] **Syntax**: `CALCULATE(Expression, Filter1, Filter2...)`.
- [ ] **Filter Modifiers**:
    - [ ] `ALL` / `REMOVEFILTERS` (Ignore context).
    - [ ] `ALLEXCEPT` (Keep specific context).
    - [ ] `ALLSELECTED` (Respect visual context).
    - [ ] `KEEPFILTERS` (Intersect rather than override).
    - [ ] `USERELATIONSHIP` (Activate inactive relationships).
    - [ ] `CROSSFILTER` (Modify relationship direction).

### D. [[Time Intelligence]]

- [ ] **Standard Functions**: `TOTALYTD`, `SAMEPERIODLASTYEAR`, `DATEADD`.
- [ ] **Custom Patterns**: Logic for 4-4-5 calendars or non-standard fiscal years.
- [ ] **Semi-Additive Measures**: `LASTNONBLANK` (for opening/closing balances).

### E. [[Advanced [[DAX Patterns]]]]

- [ ] **Virtual Tables**: `SUMMARIZE`, `ADDCOLUMNS`, `CALCULATETABLE`.
- [ ] **Ranking & Windows**: `RANKX`, `TOPN`.
- [ ] **Variables (`VAR`/`RETURN`)**: For performance (cache result) and readability.
- [ ] **Dynamic Logic**: `SWITCH(TRUE(), ...)` patterns.
- [ ] **[[Calculation Groups]]**: Reducing measure proliferation (e.g., standard YTD calculation applied to any measure).

---

## 4. [[SQL]] (Data Validation & Prep)
### A. Basic Retrieval

- [ ] **Joins**: INNER, LEFT, RIGHT, FULL OUTER (and how they affect row counts).
- [ ] **Aggregation**: GROUP BY, HAVING vs. WHERE.

### B. Analytical [[SQL]]

- [ ] **Window Functions**: `RANK()`, `ROW_NUMBER()`, `LEAD()`, `LAG()`.
- [ ] **CTEs (Common Table Expressions)**: Using `WITH` for readable, modular code.
- [ ] **Views**: Importance of using SQL Views over direct table access for Power BI.

---

## 5. Visualization & Dashboarding
### A. Theory

- [ ] **Preattentive Attributes**: Color, Size, Position, Orientation.
- [ ] **Z-Pattern / F-Pattern**: How users scan a dashboard.
- [ ] **Data-Ink Ratio**: Eliminating clutter (gridlines, borders).
- [ ] **[[IBCS]] Standards**: (Nice to have) Standardized notation for business reports.

### B. Power BI Specifics

- [ ] **Interactivity**: Edit Interactions (controlling how visuals filter each other).
- [ ] **Drill-Through vs. Tooltips**: enhancing UX without clutter.
- [ ] **Bookmarks**: Swapping visuals or saving filter states.
- [ ] **Accessibility**: Alt-text and tab order.

---

## 6. Performance, Governance & Best Practices
### A. Optimization

- [ ] **[[Vertipaq|VertiPaq]] Engine**: Columnar storage principles.
- [ ] **[[Cardinality]]**: Why high-[[Cardinality|cardinality]] columns (UUIDs, timestamps) bloat model size.
- [ ] **Performance Analyzer**: Identifying bottlenecks ([[DAX]] vs. Visual vs. DirectQuery).
- [ ] **[[DAX]] Studio**: Basics of Server Timings.

### B. Security

- [ ] **[[Row Level Security]] (RLS)**:
    - [ ] Static (Role-based).
    - [ ] Dynamic (`USERNAME()`, `USERPRINCIPALNAME()` combined with security tables).
    - [ ] Object Level Security (OLS) awareness.

### C. Enterprise Features

- [ ] **[[Composite Models]]**: Mixing DirectQuery and Import modes.
- [ ] **Aggregations**: Pre-summarizing data for DirectQuery performance.
- [ ] **Deployment Pipelines**: Dev -> Test -> Prod workflow.

---

## 7. Data Visualization Mastery
### A. [[Chart Selection]] Strategy

- [ ] **Comparison & Ranking**:
    - [ ] **Bar vs. Column**: When to use horizontal (long labels) vs. vertical (time series/fewer categories).
    - [ ] **Ribbon Charts**: Visualizing rank changes over time.
    - [ ] **Slope Charts**: comparing start vs. end states.
- [ ] **Part-to-Whole**:
    - [ ] **Treemaps**: Hierarchical data density.
    - [ ] **Donut/Pie**: The rule of "Max 2-3 slices" or strictly avoiding them.
    - [ ] **100% Stacked Bar**: Relative composition comparisons.
- [ ] **Trend & Correlation**:
    - [ ] **Line Charts**: Continuous axes (Time) vs. Categorical axes.
    - [ ] **Scatter Plots**: Identifying outliers and clusters (2 measures + 1 dimension).
    - [ ] **Combo Charts**: Dual-axis (e.g., Sales amount bars + Gross Margin % line).
- [ ] **Variance & Performance**:
    - [ ] **Waterfall Charts**: Understanding walk-from-start-to-end (P&L logic).
    - [ ] **Bullet Charts**: Progress against a target (Actual vs. Budget vs. Forecast).
    - [ ] **Gauge Charts**: When to use (single KPI) vs. when to avoid (space waste).
- [ ] **Advanced Features**:
    - [ ] **Small Multiples**: Splitting one chart into a grid by a dimension (Category/Region).
    - [ ] **Analyze Feature**: "Explain the increase/decrease" (AI visual awareness).

### B. Formatting & formatting Logic

- [ ] **Conditional Formatting**:
    - [ ] Background color / Font color based on rules (RAG status).
    - [ ] Data Bars within Matrix/Table visuals.
    - [ ] Icons (KPI indicators).
- [ ] **Dynamic Titles**: Using DAX measures to write chart titles based on slicer selection (`"Sales for " & SELECTEDVALUE(Region)`).
- [ ] **Field Parameters**: allowing users to dynamically switch X/Y axis or Legend dimensions.

---

## 8. Dashboard Design & UX
### A. Layout & Architecture (The DAR Method)

- [ ] **Dashboard Layer**: High-level KPIs, at-a-glance status, limited interaction.
- [ ] **Analysis Layer**: Slicers, interactive charts, identifying root causes.
- [ ] **Reporting Layer**: Detailed tables, grain-level data, exportable lists.

### B. Navigation & Interactivity

- [ ] **Drill-Through**:
    - [ ] Setting up drill-through pages.
    - [ ] Cross-report drill-through (jumping between different PBI reports).
- [ ] **Bookmarks & Selection Pane**:
    - [ ] Creating "Toggle" switches (e.g., switching between 'Sales $' and 'Units' views).
    - [ ] Showing/Hiding help overlays or filter panes.
- [ ] **Page Navigation**: Custom buttons vs. standard page navigator.

### C. Storytelling & Accessibility

- [ ] **Z-Pattern vs. F-Pattern**: Placing the most important KPI (BAN - Big Angry Number) in top-left.
- [ ] **Whitespace**: Using padding to separate logical business sections.
- [ ] **Color Blindness**: Using accessible palettes (avoiding Red/Green only).
- [ ] **Mobile Layout**: Creating specific mobile views vs. responsive desktop views.

### D. Governance in Visualization

- [ ] **Custom Visuals**: Risks of using non-certified visuals (security/performance/breaking changes).
- [ ] **Theme Files (JSON)**: Standardizing corporate colors and font sizes across reports.

---

## 9. SQL & [[dbt]] (The Semantic Prep Layer)
### A. SQL for Data Modeling

- [ ] **Data Cleaning Functions**:
    - [ ] `COALESCE()`: Handling NULLs before they hit Power BI.
    - [ ] `CAST()` / `CONVERT()`: Fixing [[Data Types|data types]] explicitely.
    - [ ] String manip: `TRIM()`, `UPPER()`, `LEFT()`, `CONCAT()`.
- [ ] **Logic Handling**:
    - [ ] `CASE WHEN`: Creating buckets/segments (e.g., "High", "Medium", "Low") at the source.
- [ ] **Date Generation**:
    - [ ] Generating a sequence of dates for the DimDate source in SQL.
- [ ] **Set Operations**:
    - [ ] `UNION ALL` vs `UNION`: Stacking historical files (Performance implications).

### B. [[dbt]] (Data Build Tool) Concepts

- [ ] **The "T" in ELT**: Understanding [[dbt]] does transformation, not extraction/load.
- [ ] **Project Structure**:
    - [ ] **Sources**: Defining raw data connections.
    - [ ] **Staging (stg)**: 1:1 with source, renaming columns to snake_case, basic casting.
    - [ ] **Intermediate (int)**: Complex logic, joins, business rules.
    - [ ] **Marts (fct/dim)**: The final [[Star Schema]] tables ready for Power BI import.
- [ ] **Materialization Strategies**:
    - [ ] `view`: Virtual, calculated on run.
    - [ ] `table`: Physically stored, replaced on run.
    - [ ] `incremental`: Only processing new/changed rows (crucial for big data).
    - [ ] `ephemeral`: Reusable code snippets (CTE injection).

### C. dbt & [[Kimball]] Implementation

- [ ] **Snapshots**:
    - [ ] Implementing **SCD Type 2** automatically using `dbt snapshot`.
    - [ ] `check` strategy vs. `timestamp` strategy.
- [ ] **Surrogate Keys**:
    - [ ] Using `dbt_utils.generate_surrogate_key` (hashing business keys) to create unique IDs for Dimension/Fact joins.
- [ ] **Testing (Data Quality)**:
    - [ ] `unique`, `not_null` constraints.
    - [ ] `accepted_values` (e.g., Status must be 'Open', 'Closed').
    - [ ] `relationships` (Referential integrity check between Fact and Dim).

### D. Integration: dbt to Power BI

- [ ] **Documentation**: How `description:` in dbt YAML files propagates to Power BI field descriptions (if supported by connector/tooling).
- [ ] **The "Golden Dataset"**:
    - [ ] Treating the dbt "Marts" layer as the single source of truth.
    - [ ] Avoiding further transformation in Power Query (Folding only) if dbt is used.