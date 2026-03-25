
### 1. The Core Philosophy

dbt does **not** extract or load data. It assumes the raw data is already in your Data Warehouse (Snowflake, BigQuery, Databricks, Postgres).
*   **The Motto:** "Everything is a `SELECT` statement."
*   **The Function:** It compiles your [[SQL]] code (mixed with Jinja templating) into raw [[SQL]] and runs it against the warehouse to create [[Tables]] and Views.
*   **The Goal:** Bring Software Engineering best practices (Version Control, Testing, CI/CD, Documentation) to [[SQL]].

---

### 2. Materializations (How data is saved)

This is the **#1 Core dbt Topic**. You define these in the `dbt_project.yml` or the specific model config.

| Materialization | SQL Equivalent | Behavior | Best Use Case |
| :--- | :--- | :--- | :--- |
| **View** | `CREATE VIEW AS...` | Virtual. No data stored. Runs every time you query it. | **Staging Layer (`stg_`).** Lightweight transformations. |
| **Table** | `CREATE TABLE AS...` | Physical storage. Drops the table and rebuilds it entirely every run. | **Dimension [[Tables]] (`dim_`).** When data volume is manageable and read speed matters. |
| **Incremental** | `MERGE` / `INSERT` | **The Complex One.** Only adds/updates rows that have changed since the last run. | **Fact [[Tables]] (`fct_`).** Massive event logs where rebuilding the whole table takes too long. |
| **Ephemeral** | `WITH alias AS...` | **The Ghost.** Does not exist in the DB. It is injected as a CTE into downstream models. | **Reusable Logic.** Code snippets used in multiple models to avoid cluttering the warehouse. |

**Mnemonic: VITE**
(View, Incremental, Table, Ephemeral).

---

### 3. The DAG (Dependency Graph) & Lineage

dbt automatically figures out the order in which to run your scripts. It builds a **DAG** (Directed Acyclic Graph).

*   **The Function:** `{{ ref('model_name') }}`
*   **The Rule:** Never write `FROM database.schema.table`. Always write `FROM {{ ref('stg_users') }}`.
*   **Why?**
    1.  **Environments:** dbt automatically switches between Dev (`dbt_jdoe_schema`) and Prod (`analytics_schema`).
    2.  **Dependencies:** dbt knows it must build `stg_users` *before* it tries to build `dim_users`.

*   **The Source:** `{{ source('raw_system', 'users') }}`
    *   Used only in the very first layer (Staging) to reference the raw data loaded by your EL tool (Fivetran/Airbyte).

---

### 4. Project Structure (The "Layers")
Structuring a dbt project correctly is a sign of a Senior Engineer.

#### Layer 1: Staging (`stg_`)

*   **Input:** `{{ source() }}` (Raw Data).
*   **Materialization:** View.
*   **Tasks:** Rename columns to English, cast [[Data Types|data types]], basic cleanup (trim). **1-to-1 mapping** with source tables.

#### Layer 2: Intermediate (`int_`)

*   **Input:** `{{ ref('stg_...') }}`.
*   **Materialization:** Ephemeral or Table.
*   **Tasks:** Heavy logic. Joins, complex calculations, aggregations. This is where the messy SQL lives.

#### Layer 3: Marts (`dim_` / `fct_`)

*   **Input:** `{{ ref('int_...') }}`.
*   **Materialization:** Table or Incremental.
*   **Tasks:** Final polish. [[Star Schema]] readiness.
    *   **Dimensions:** Wide tables, descriptive attributes.
    *   **Facts:** Narrow tables, keys and metrics.
*   **Consumer:** Power BI reads from *here*.

---

### 5. Testing (Data Quality)
dbt allows you to test data *before* it breaks your Power BI report.

#### A. Generic Tests (Built-in)

Defined in `schema.yml`.
1.  **Unique:** No duplicates in the ID column.
2.  **Not Null:** No missing values.
3.  **Accepted Values:** Status must be one of `['Open', 'Closed']`.
4.  **[[Relationships]]:** Referential Integrity (Foreign Key check). Does `customer_id` in Sales exist in the Customer table?

#### B. Singular Tests (Custom)

You write a specific SQL query. If the query returns **rows**, the test **fails**.
*   *Example:* `SELECT * FROM {{ ref('orders') }} WHERE order_date > ship_date` (Orders cannot ship before they are placed).

---

### 6. Snapshots (SCD Type 2)
dbt handles history tracking automatically.

*   **Scenario:** A user changes their email address in the source system. The old email is gone (SCD Type 1).
*   **dbt Snapshot:** You tell dbt to watch the `users` table.
*   **Result:** dbt maintains a table with `dbt_valid_from` and `dbt_valid_to` columns.
    *   Row 1: User A, Old Email, Valid To: 2024-01-01.
    *   Row 2: User A, New Email, Valid To: NULL (Current).

---

### 7. Macros & Jinja (The "Programming" Part)
Jinja is the templating language `{ ... }` inside SQL.

*   **Variables:** `{% set tax_rate = 0.19 %}`
*   **Control Flow:** `{% if target.name == 'dev' %} LIMIT 100 {% endif %}` (Don't load all data in dev).
*   **Macros:** Functions written in SQL.
    *   *Example:* A macro to convert cents to dollars repeatedly. `{{ cents_to_dollars('amount_column') }}`.
    *   *DRY Principle:* **D**on't **R**epeat **Y**ourself.

---

### 8. dbt Best Practices for Power BI

1.  **Hide the Logic:** Power BI should simply do `SELECT * FROM fct_sales`. If you are doing heavy joins or `CASE WHEN` logic in Power Query or [[DAX]], move it upstream to dbt.
2.  **Surrogate Keys:** Generate your Integer Surrogate Keys in dbt using the `dbt_utils.generate_surrogate_key()` macro (usually hashes) or using `ROW_NUMBER()`.
3.  **Documentation:** Use `dbt docs generate`. It reads the descriptions from your `.yml` files and creates a website showing the lineage and column definitions.
4.  **One Big Table (OBT) vs [[Star Schema]]:**
    *   dbt can build both.
    *   *Best Practice:* Build the [[Star Schema]] (`dim` and `fct`) for Power BI.
    *   If Data Scientists need a flat dataframe, create a separate OBT model that joins the Star Schema together for them.

---

### Summary Checklist

If they ask: **"How do you ensure data quality in your pipeline?"**
**Answer:** "I use dbt **Generic Tests** (Unique, Not_Null) on my Staging models to catch errors early. I use **Relationship Tests** on my Marts to ensure Referential Integrity for the Star Schema. I run these tests in my CI/CD pipeline before merging code."

If they ask: **"Explain Incremental Models."**
**Answer:** "Incremental models allow us to process only the new data (delta) rather than rebuilding the full history every day. I configure the `is_incremental()` logic in dbt to filter based on a timestamp column, appending new rows and merging updates to existing keys."