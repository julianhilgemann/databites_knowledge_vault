
## 1. The Core Philosophy: "Bottom-Up"

Ralph Kimball’s approach differs from the traditional "Inmon" approach (which builds a massive, perfect Enterprise Data Warehouse first).

*   **The Kimball Motto:** "Start with the business process, not the data."
*   **The Approach:** Build smaller, subject-specific areas (Data Marts) that provide immediate value, then stitch them together later.
*   **Power BI Fit:** This is a **perfect match** for Power BI, which is often implemented iteratively (e.g., "Let's build a Sales Dashboard first, then add Inventory next month").

---

## 2. The 4-Step Design Process

Kimball defined a rigid 4-step process to design any model. If you skip these in Power BI, you usually end up with a "Spaghetti Model."

### Step 1: Select the Business Process

*   **Theory:** Don't say "I am modeling the Sales Database." Say "I am modeling the **Order Fulfillment** process."
*   **Power BI:** This determines the scope of your Dataset (Semantic Model). One model should answer questions about one coherent process (or a few tightly linked ones).

### Step 2: Declare the Grain

*   **Theory:** What does **one single row** in the Fact table represent?
    *   *Level 1:* One row per Line Item on an Invoice? (Finest grain).
    *   *Level 2:* One row per Order Header?
    *   *Level 3:* One row per Day per Store? (Aggregated).
*   **Power BI:** **Atomic Grain is Best.** [[Vertipaq|VertiPaq]] loves detail. Don't pre-aggregate (SUM) in [[SQL]] unless the data volume is petabytes. Load the lowest level of detail (Line Item) so users can slice by anything.

### Step 3: Identify the Dimensions

*   **Theory:** The "Who, What, Where, When, Why."
*   **Power BI:** These become your **Slicers** and **Axis** fields. If a user wants to "Slice by City," you need a Geography Dimension.

### Step 4: Identify the Facts

*   **Theory:** The numerical measurements that result from the process.
*   **Power BI:** These become your **Measures** (Sum of Sales, Count of Orders).

---

## 3. Critical Kimball Concepts for Power BI

### A. Slowly Changing Dimensions (SCD)

Data changes over time (e.g., a Salesperson moves from the "West" region to the "East" region). How do you handle history?

| SCD Type | Methodology | Power BI Implementation |
| :--- | :--- | :--- |
| **Type 0** | **Fixed.** Original value never changes. | Ignore updates. Rare. |
| **Type 1** | **Overwrite.** The old value is gone. "West" becomes "East". | **Most Common.** Good for current state reporting. History is lost (old sales will look like they happened in the East). |
| **Type 2** | **Add Row.** Keep history. Create a new row for the user with a new Surrogate Key and a Date Range (`ValidFrom`, `ValidTo`). | **The Gold Standard.** Requires an Integer Surrogate Key in the Fact table. Allows you to see sales in the West *when they were in the West* and East *now*. |

### B. Role-Playing Dimensions

A single physical table used for multiple purposes.
*   **Scenario:** You have one `Date` table, but your Fact table has `OrderDate`, `ShipDate`, and `DueDate`.
*   **Kimball Theory:** Use Views to create `Dim_OrderDate`, `Dim_ShipDate`, etc.
*   **Power BI Practice:**
    *   *Option A (Physical):* Import the Date table 3 times. (Easier for users).
    *   *Option B (Virtual):* Import Date once. Create one active relationship and two inactive ones. Use [[DAX]] `USERELATIONSHIP()` to activate the specific path when needed.

### C. Conformed Dimensions (The "Bus Architecture")

This is how you prevent "Silos" of data.
*   **Theory:** If "Customer" appears in the *Sales* Data Mart and the *Support* Data Mart, it must be the **exact same structure** (same Keys, same Names).
*   **Power BI:** This is critical for **Cross-Report Drill Through** and **[[Composite Models]]**. If you want to compare Sales vs. Budget, both [[Tables|tables]] must share the exact same `Date` and `Product` dimension [[Tables|tables]].

### D. Junk Dimensions

*   **Theory:** You have 10 columns of low-[[Cardinality|cardinality]] flags (`Is_Verified`, `Has_Discount`, `Status_Code`, `Paid_Flag`).
*   **Problem:** Don't put 10 tiny dimensions in your [[Star Schema]] (it looks messy). Don't leave them in the Fact (it creates width).
*   **Solution:** Create one single "Junk Dimension" containing every unique *combination* of those flags, give each row a Key, and link it to the Fact.
*   **Power BI:** Reduces model clutter significantly and improves compression.

---

## 4. The Three Types of Fact [[Tables]]

Kimball identifies three distinct patterns. Mixing these in one Power BI table is a common mistake.

### 1. Transaction Fact

*   **Definition:** One row per event. (e.g., Every beep at a checkout counter).
*   **Power BI:** The most common. Best for summing, counting, and flexible analysis.

### 2. Periodic Snapshot Fact

*   **Definition:** One row per period. (e.g., Use [[SQL]] to capture the Inventory Level *every night* at midnight).
*   **Power BI:** Essential for "Stock on Hand" or "Account Balance" reporting. You cannot sum these over time (Inventory on Jan 1 + Jan 2 $\neq$ Total Inventory). You must use `LASTNONBLANK` or `AVERAGE` in [[DAX]].

### 3. Accumulating Snapshot Fact

*   **Definition:** One row per life-cycle. (e.g., A Loan Application). The row has multiple date columns: `ApplicationDate`, `ApprovalDate`, `FundingDate`. The columns are updated as the process moves forward.
*   **Power BI:** Great for calculating "Lag" or "Lead" times (e.g., `AVERAGE(FundingDate - ApplicationDate)`). Harder to load because the data source is constantly updating existing rows.

---

## 5. Summary: Kimball vs. The Modern Data Stack

How does the 1990s theory hold up in 2024?

| Kimball Rule | Modern Data Engineering / Power BI Reality |
| :--- | :--- |
| **"[[Star Schema]] is King"** | **TRUE.** [[Vertipaq|VertiPaq]] is essentially a [[Star Schema]] engine. |
| **"Surrogate Keys (Integers)"** | **TRUE.** Mandatory for performance in Power BI. |
| **"ETL at Night"** | **PARTIAL.** Power BI allows [[Incremental Refresh|incremental refresh]], but the logic of preparing clean dimensions remains the same. |
| **"One Version of the Truth"** | **TRUE.** Handled via "Shared Datasets" (One Power BI Dataset used by 20 reports). |
| **"Avoid Snowflakes"** | **TRUE.** Power BI prefers denormalized Dimensions over Snowflakes for speed and simplicity. |
| **"Null Handling"** | **TRUE.** Kimball says "No Nulls in Keys." Power BI agrees (replace with `-1 / Unknown`). |

### The "One Big Table" (OBT) Debate

Modern Data Warehouses (Snowflake/BigQuery) are so fast that some engineers skip Kimball and just make "One Big Table."
*   **In [[SQL]]:** This works fine.
*   **In Power BI:** **Do not do this.** Power BI has a 10GB-100GB limit (usually). OBT is inefficient with memory. Kimball's [[Normalization|normalization]] of text into Dimensions allows Power BI to compress data 10x-50x smaller than OBT.

### Final Takeaway

Think of **Ralph Kimball** as the original architect of Power BI's brain.
If you prepare your data in SQL using **Kimball's Star Schema**, your Power BI reports will be:
1.  **Fast** ([[Relationships]] work instantly).
2.  **Simple** ([[DAX]] formulas are short).
3.  **Accurate** (No orphan records or ambiguous joins).