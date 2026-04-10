
# The Concept: "The Waterfall"

Think of Filter Propagation as **Water flowing downhill**.
*   **The Source:** The "One" side (Dimension Table).
*   **The Destination:** The "Many" side (Fact Table).
*   **Gravity:** The Relationship Direction.

By default, water flows from the Dimension to the Fact. It does **not** naturally flow back up.

---

# The Scenario: The "Hat" Problem

Let's imagine a simple model.

### 1. The [[Tables]]

**Table A: `Dim_Customer` (The "One" Side)**
| CustomerID | Name | City |
| :--- | :--- | :--- |
| 1 | **Alice** | Berlin |
| 2 | **Bob** | Paris |
| 3 | **Charlie**| London |

**Table B: `Dim_Product` (The "One" Side)**
| ProductID | Product | Color |
| :--- | :--- | :--- |
| 100 | **Red Hat** | Red |
| 101 | **Blue Shirt**| Blue |

**Table C: `Fact_Sales` (The "Many" Side)**
| OrderID | CustomerID (FK) | ProductID (FK) | Amount |
| :--- | :--- | :--- | :--- |
| 1 | 1 (Alice) | 100 (Red Hat) | $50 |
| 2 | 2 (Bob) | 101 (Blue Shirt) | $20 |

---

# Scenario 1: Standard Propagation (Downhill)

**Goal:** Calculate Total Sales for "Berlin".

1.  **User Action:** You create a Slicer on `Dim_Customer[City]` and select **"Berlin"**.
2.  **Propagation:**
    *   `Dim_Customer` filters itself to ID `1` (Alice).
    *   The filter flows **down** the relationship to `Fact_Sales`.
    *   `Fact_Sales` keeps only rows where `CustomerID = 1`.
3.  **The Result:** `SUM(Amount)` = **$50**.

**Status:** ✅ **Perfect.** This is how Power BI is designed to work.

---

# Scenario 2: The "Upstream" Problem (Uphill)

**Goal:** Count how many customers bought a "Red Hat".

1.  **User Action:** You create a Slicer on `Dim_Product[Product]` and select **"Red Hat"**.
2.  **Propagation:**
    *   `Dim_Product` filters itself to ID `100`.
    *   The filter flows **down** to `Fact_Sales`.
    *   `Fact_Sales` keeps only Order #1 (Alice).
3.  **The Question:** You put a Card visual on the canvas: `COUNTROWS(Dim_Customer)`.
4.  **The Result:** **3** (Alice, Bob, and Charlie).

**Why?**
*   The filter reached `Fact_Sales`, but it stopped there.
*   It did **not** flow "back up" (Uphill) to `Dim_Customer`.
*   Therefore, `Dim_Customer` remains unfiltered. It sees all 3 people.

**Status:** ❌ **Counter-intuitive.** (Users hate this).

---

# Scenario 3: The "Bi-Directional" Fix (Dangerous)

You go to "Manage [[Relationships]]" and change the `Fact_Sales` <-> `Dim_Customer` relationship from **Single** to **Both**.

1.  **Propagation:**
    *   Filter "Red Hat" $\to$ `Dim_Product`.
    *   `Dim_Product` $\to$ `Fact_Sales` (Found Alice's order).
    *   **NEW:** `Fact_Sales` $\to$ `Dim_Customer` (Flows back up).
    *   `Dim_Customer` is filtered to ID 1 (Alice).
2.  **The Result:** `COUNTROWS(Dim_Customer)` = **1**.

**Status:** ⚠️ **Technically works, but dangerous.**
*   **Performance:** You just doubled the work the engine has to do.
*   **Ambiguity:** If you have multiple fact [[Tables|tables]] (e.g., Sales and Budget), enabling "Both" can create "Circular Dependency" errors where the engine doesn't know which path to take.

---

# Scenario 4: The Best Practice ([[DAX]] Control)

Keep the relationship as **Single**. Use [[DAX]] to force the filter upstream *only when needed*.

### The [[DAX]] Solution: `CROSSFILTER`

```dax
Customers Who Bought Item = 
CALCULATE(
    COUNTROWS(Dim_Customer),
    CROSSFILTER(
        Fact_Sales[CustomerID],     -- The Many Side column
        Dim_Customer[CustomerID],   -- The One Side column
        Both                        -- Temporarily turn on Bi-Directional
    )
)
```

**How it works:**
1.  The formula activates.
2.  It creates a temporary, virtual "Both" direction relationship just for this specific calculation.
3.  It counts Alice.
4.  It reverts the relationship back to Single immediately.

**Status:** ✅ **Best Practice.** It is surgical and performant.

---

# What to Look For (The Inspection Checklist)

When you inherit a Power BI file, look for these visual indicators and code patterns.

### 1. The Relationship Arrows (Model View)
Look at the lines connecting your [[Tables|tables]].

*   **`1 -> *` (Single Arrow):**
    *   *Meaning:* Filters flow from One to Many.
    *   *Verdict:* **Good.** This is the standard.
*   **`1 <-> *` (Double Arrow):**
    *   *Meaning:* Bi-directional filtering.
    *   *Verdict:* **Red Flag.** Ask yourself: "Why is this on?" Is it a lazy fix for a slicer? If so, turn it off and use DAX. Is it a Bridge Table? Then it might be necessary.

### 2. Auto-Exist (The "Column" Trap)
If you filter on two columns from the *same* table, Power BI works differently than filtering on two columns from *different* tables.

*   **Scenario:**
    *   Table: `Dim_Product` has columns `Color` and `Category`.
    *   Filter 1: `Color = "Red"`.
    *   Filter 2: `Category = "Hat"`.
*   **Auto-Exist:** The engine looks at the *existing combinations* in the table. It calculates the intersection immediately.
*   **Risk:** If you have massive [[Cardinality|cardinality]] on one of those columns (like a UUID), this intersection calculation can crash the visual.

### 3. Expanded Tables (The Mental Model)
In DAX, when you have a relationship, the "Many" side technically *contains* the "One" side.

*   **If you are standing in `Fact_Sales`:** You can "see" `Dim_Customer[City]` directly.
    *   *DAX:* `RELATED(Dim_Customer[City])` works.
*   **If you are standing in `Dim_Customer`:** You cannot "see" `Fact_Sales` directly because there are many rows.
    *   *DAX:* `RELATED(Fact_Sales[Amount])` **fails**.
    *   *Fix:* You must aggregate it: `RELATEDTABLE(Fact_Sales)` or `SUMX(RELATEDTABLE(Fact_Sales), [Amount])`.

---

# Summary Table: Filter Propagation Best Practices

| Scenario | Relationship Type | Direction | Recommendation |
| :--- | :--- | :--- | :--- |
| **Standard Reporting**<br>(Sales by City) | 1:Many | **Single** | **Default.** Always use this unless you can't. |
| **Slicer filtering Slicer**<br>(Select "Red Hat", only show Customers who bought it) | 1:Many | **Both** (Maybe) | Use **Bi-directional** cautiously if user experience demands it, but beware of ambiguity. |
| **Calculation**<br>(Count Customers who bought X) | 1:Many | **Single** | Use **DAX** (`CROSSFILTER` or `CALCULATE` logic) to push filters uphill. Do not change the model. |
| **Many-to-Many**<br>(Bridge Table) | 1:Many - Many:1 | **Both** | On the relationship between the **Fact and Bridge**, generally set to Single. On **Dimension to Bridge**, set to Both (or Single depending on logic). |

### Mnemonic for Memorization
**"The Valve"**

Think of the relationship arrow as a **One-Way Valve** in a pipe.
*   **Single (`<`):** Water flows down. Pressure building up at the bottom (Fact) does not push water back up to the top.
*   **Both (`<>`):** The valve is removed. Water sloshes back and forth. If you have too many of these connected, the water (filters) gets confused about which pipe to flow down, causing "Circular Dependency."