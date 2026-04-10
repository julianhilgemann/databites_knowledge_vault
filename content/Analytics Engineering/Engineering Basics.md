
# Data Engineering Theory: The Acronyms & Concepts

## 1. ACID (The "Bank Vault" Standard)

**[[Context]]:** Relational Databases ([[SQL]] Server, Oracle, PostgreSQL).
Used for **Transactions** (OLTP). If you transfer money, you want ACID.

| Letter | Concept | Definition | Mnemonic / Helper |
| :--- | :--- | :--- | :--- |
| **A** | **Atomicity** | **"All or Nothing."** The transaction happens completely, or it doesn't happen at all. No half-finished updates. | **A**tom = **A**ll or nothing.<br>*(Like an atomic bomb, you can't half-explode it).* |
| **C** | **Consistency** | **"Follow the Rules."** The data must meet all validation rules (Constraints, [[Data Types]]) before and after the transaction. | **C**orrectness.<br>*(The referee ensures the game follows the rules).* |
| **I** | **Isolation** | **"Don't Peek."** One transaction cannot see another transaction until it is finished. | **I**nvisible.<br>*(What happens in Vegas stays in Vegas until you leave).* |
| **D** | **Durability** | **"Written in Stone."** Once the system says "Success," the data is saved forever, even if the power goes out immediately after. | **D**isk.<br>*(If it's not on the Disk, it didn't happen).* |

*   **Power BI [[Context]]:** Power BI relies on ACID sources to ensure that when it imports data, it isn't importing "half-written" records. However, during a refresh, Power BI does *not* offer ACID compliance for the report viewers (users might see a report update while they are looking at it).

---

## 2. BASE (The "Social Media" Reality)

**[[Context]]:** NoSQL, Big Data, Distributed Systems (CosmosDB, Cassandra).
Used for massive scale where speed is more important than perfection.

| Letter | Concept | Definition | Mnemonic / Helper |
| :--- | :--- | :--- | :--- |
| **BA** | **Basically Available** | The system guarantees availability (it won't crash), but might not give the most recent data. | **B**asic **A**ccess.<br>*(The shop is open, but the shelves might be messy).* |
| **S** | **Soft State** | The state of the system may change over time, even without input (due to syncing). | **S**hifting.<br>*(Like quicksand/soft sand, it moves).* |
| **E** | **Eventual Consistency** | If you stop inputting data, eventually all nodes will sync up. | **E**ventually.<br>*(Like a rumor spreading—eventually everyone hears the same story).* |

*   **Power BI Context:** When connecting to Big Data sources (like a Data Lake), you might hit "Eventual Consistency." You refresh the report and see Total Sales: \$100. You refresh again 1 second later: \$105. The data was still settling.

---

## 3. CAP Theorem (The "Choose Two" Triangle)

**Context:** Distributed Systems. You physically cannot have all three.

*   **C - Consistency:** Everyone sees the same data at the same time.
*   **A - Availability:** The system always responds (no "Server Down" errors).
*   **P - Partition Tolerance:** The system works even if the network cable between servers is cut.

**The Mnemonics:**
*   **CA ([[SQL]] Server):** "I want the truth (C) and uptime (A), but I can't handle a network cut (P)." -> *Standard Relational DB.*
*   **CP (Banking/Finance):** "I want the truth (C) and I can handle a network cut (P), but the system might go down (A) to protect the data." -> *Strict ATMs.*
*   **AP (Social Media):** "I want uptime (A) and I can handle a network cut (P), but the data might be old (C)." -> *Twitter/Facebook feeds.*

---

## 4. OLTP vs. OLAP (The Workload)

The single most important distinction for modeling.

| Acronym | Name | Role | Mnemonic | Power BI Relation |
| :--- | :--- | :--- | :--- | :--- |
| **OLTP** | Online **Transaction** Processing | **Writing.** fast, small updates. (e.g., The Checkout Register). Highly Normalized (3NF). | **T** = **T**iny updates.<br>**T** = **T**ransactions. | **The Source.** Power BI sucks at querying OLTP directly (too many joins). |
| **OLAP** | Online **Analytical** Processing | **Reading.** Heavy aggregations. (e.g., The Monthly Report). Denormalized ([[Star Schema]]). | **A** = **A**nalysis.<br>**A** = **A**ggregations. | **The Destination.** Power BI *is* an OLAP engine. |

---

## 5. CRUD (The Verbs of Data)

**Context:** API development, App development.

| Letter | Action | [[SQL]] Equivalent | Power BI Context |
| :--- | :--- | :--- | :--- |
| **C** | **Create** | `INSERT` | Power Query (Adding columns). |
| **R** | **Read** | `SELECT` | **99% of Power BI.** |
| **U** | **Update** | `UPDATE` | Power Query (Replace Values). |
| **D** | **Delete** | `DELETE` | Power Query (Filter rows out). |

*   **Mnemonic:** "Data dirt is **CRUD**." (You have to clean it).

---

## 6. ETL vs. ELT (The Pipeline Order)

**ETL (Extract, Transform, Load)**
*   *Old School (On-Prem).*
*   Extract data, clean it on a dedicated server, *then* put it in the warehouse.
*   **Mnemonic:** "Clean the veggies **before** you put them in the fridge."

**ELT (Extract, Load, Transform)**
*   *Modern Cloud (Snowflake, BigQuery, Databricks).*
*   Dump raw data into the warehouse immediately, *then* use the warehouse's power to clean it.
*   **Mnemonic:** "Dump the groceries in the fridge, chop them **later** when cooking."

*   **Power BI Context:** Power BI Dataflows are technically **ETL**. You extract, Power Query transforms it, and loads it into the model.

---

## 7. SMP vs. MPP (The Hardware Architecture)

**SMP (Symmetric Multi-Processing)**
*   **Concept:** One big server. If it gets slow, buy a bigger server (Vertical Scaling).
*   **Mnemonic:** **S**haring **M**y **P**rocessor. (SQL Server).

**MPP (Massively Parallel Processing)**
*   **Concept:** Many small servers working together. If it gets slow, add more cheap servers (Horizontal Scaling).
*   **Mnemonic:** **M**any **P**uny **P**rocessors. (Synapse, Snowflake, Spark).

*   **Power BI Context:** The Power BI Service (Premium) runs on **MPP** architecture in the backend to handle massive datasets.

---

## Summary Checklist for the Power BI Engineer

1.  **Source:** Is it **OLTP** (SQL)? If so, it follows **ACID**.
2.  **Pipeline:** Are we doing **ETL** (Power Query) or **ELT** (SQL Views)?
3.  **Model:** Convert the relational data into **OLAP** ([[Star Schema]]).
4.  **Expectations:** If the source is **BASE** (NoSQL), warn the users about Eventual Consistency.