
# Data Normalization: The Theory vs. The Reality

### 1. What is it?

Normalization is the process of organizing a database to:
1.  **Reduce Redundancy:** Don't store "Steve" and "Berlin" 1,000 times if Steve buys 1,000 items. Store it once.
2.  **Improve Integrity:** If Steve moves to Paris, you update his address in **one** place, not 1,000 rows.

### 2. The Normal Forms (The Steps)

The academic standard is based on E.F. Codd's rules. You usually only care about the first three (1NF, 2NF, 3NF).

#### The Mnemonic (The Oath)

To remember the rules, memorize this famous database quote:
> **"Every non-key attribute must provide a fact about:**
> 1.  **The Key** (1NF)
> 2.  **The Whole Key** (2NF)
> 3.  **And Nothing But The Key** (3NF)
> **...so help me Codd."**

---

### Step-by-Step Example

Imagine a messy spreadsheet: **Orders Table**
| OrderID | Items | Customer | CustomerCity | Supplier | SupplierPhone |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 101 | Apple, Banana | Alice | Berlin | FruitCo | 555-0199 |

#### 1NF: First Normal Form ("The Key")

**Rule:** Atomic Values. No lists in a single cell. No repeating groups.
*   **Action:** Split "Apple, Banana" into two rows.
*   **Result:** Now we have a unique composite key (`OrderID` + `Item`).

#### 2NF: Second Normal Form ("The Whole Key")

**Rule:** No Partial Dependency. All data must rely on the *entire* Primary Key.
*   **Problem:** `SupplierPhone` depends on `Supplier`, not on the specific `Item` or `Order`.
*   **Action:** Move Supplier data to a separate table.
*   **Result:** `Supplier` table and `Order` table.

#### 3NF: Third Normal Form ("Nothing But The Key")

**Rule:** No Transitive Dependency. Columns shouldn't depend on *other* non-key columns.
*   **Problem:** `CustomerCity` depends on `Customer`. It doesn't really depend on the `OrderID`. If Alice places a new order, her city is already known.
*   **Action:** Move Customer data to a `Customer` table.
*   **Result:** We now have 3 [[Tables|tables]]: `Orders`, `Customers`, `Suppliers`.

---

### 3. Normalization in Power BI (The Twist)

In a professional setting, you must distinguish between **Application Design (OLTP)** and **Power BI Design (OLAP)**.

| Feature | [[SQL]] / App Development (OLTP) | Power BI / Analytics (OLAP) |
| :--- | :--- | :--- |
| **Goal** | Fast Writes (Insert/Update). | Fast Reads (Sum/Count). |
| **Ideal Form** | **3NF (Third Normal Form).** | **[[Star Schema]] (Dimensional).** |
| **Joins** | Loves them. Joins are fine. | Hates them. Joins are expensive. |
| **Redundancy** | **Forbidden.** (Data integrity risk). | **Encouraged.** ([[Vertipaq|VertiPaq]] compresses it). |

#### The "Snowflake" Problem

If you import a strictly 3NF database into Power BI, you get a **Snowflake Schema**:
*   `Sales` $\to$ `Product` $\to$ `SubCategory` $\to$ `Category`.
*   **Power BI Verdict:** **Bad.**
    *   To filter Sales by "Category", the filter has to jump through 3 [[Relationships|relationships]]. This is slow.

#### The Power BI Solution: Denormalization

You take the 3NF data and "Denormalize" the Dimensions.
*   **Action:** Merge `Category`, `SubCategory`, and `Product` into **One Wide Product Table**.
*   **Result:** Yes, the word "Clothing" (Category) is repeated 10,000 times in the Product table.
*   **Why it's okay:** Power BI's engine ([[Vertipaq|VertiPaq]]) stores that column as a dictionary. It only stores the word "Clothing" once and uses pointers. It is highly optimized for this.

---

### 4. Summary: What to say in a professional setting

**The Scenario:**
Stakeholder: *"Our source database is highly normalized. How do you handle that in Power BI?"*

**The Best Practice Answer:**
1.  **"I respect normalization in the Fact Table."**
    *   The Fact table should stay narrow (Keys + Metrics). I don't want customer names in the Fact table (that's too much redundancy/bloat).
2.  **"I denormalize the Dimensions."**
    *   I will use Power Query (or [[SQL]] Views) to join `City`, `State`, and `Country` into a single `Dim_Geography` table.
    *   I will join `Product`, `SubCat`, and `Cat` into a single `Dim_Product` table.
3.  **"The Goal is a [[Star Schema]]."**
    *   I want fewer [[Tables|tables]] with fewer hops ([[Relationships|relationships]]), even if that means the Dimension [[Tables|tables]] are "wide" and contain repeated text values.

### 5. Quick Glossary

*   **Update Anomaly:** The risk in denormalized data where you update a record in one place but forget another (e.g., updating Steve's address in row 1 but missing row 500). *Power BI doesn't care because it's Read-Only.*
*   **Insertion Anomaly:** You can't add a new Product Category until you have a Product that belongs to it. (3NF solves this).
*   **Referential Integrity:** Ensuring that if a Sale links to Product ID 5, Product ID 5 actually exists. (Power BI mimics this with the "Blank" row behavior).

### Final Mnemonic for Power BI

**"Normalize the Facts, Denormalize the Dimensions."**
(Keep the transactions efficient, but make the descriptive tables flat and easy to slice).