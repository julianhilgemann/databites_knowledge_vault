Here is the deep dive into **Row-Level Security (RLS)**.


### 1. The Concept

**RLS (Row-Level Security)** restricts **data** access for a given user.
*   It does **not** restrict access to the Report or the Workspace.
*   *Scenario:* Alice and Bob both have access to the "Global Sales Report." When Alice opens it, she sees only US data. When Bob opens it, he sees only EU data.

### 2. The Two Types of RLS

#### A. Static RLS (The Basic Way)

You hardcode filters into Roles inside Power BI Desktop.
*   **Setup:** Create a Role called "West Manager". Add [[DAX]] filter on `Dim_Geography`: `[Region] = "West"`.
*   **Pros:** Simple, quick for very small, non-changing requirements.
*   **Cons:** Nightmare to maintain. If you have 50 regions, you need 50 Roles. You have to redeploy the PBIX file to add a new region.

#### B. Dynamic RLS (The Senior Way)

You create **one single Role** that dynamically filters data based on who is logged in.
*   **Setup:**
    1.  You need a table in your model (e.g., `Security_Table`) mapping Emails to Dimensions (Region/Department).
    2.  You use the [[DAX]] function `USERPRINCIPALNAME()` to compare the logged-in user against that table.
*   **Pros:** scalable. To add a user, you just insert a row in the database/Excel source. No PBIX redeployment needed.

---

### 3. The "Golden Pattern" for Dynamic RLS

This is the standard architectural pattern you should describe.

1.  **The Security Table:**
    Create a table `RLS_Users` with columns: `UserEmail`, `RegionID`.
    *(Note: One user might have multiple rows if they manage multiple regions).*

2.  **The Relationship:**
    Join `RLS_Users[RegionID]` to `Dim_Geography[RegionID]`.

3.  **The Direction (Crucial):**
    *   [[Relationships]] usually filter One-to-Many.
    *   Here, the Security Table filters the Dimension.
    *   You must check the box: **"Apply security filter in both directions"** (if the security table is hidden/disconnected) OR ensure the filter flows downhill to the Fact.

4.  **The [[DAX]] Rule:**
    On the `RLS_Users` table, add this filter:
    ```dax
    [UserEmail] = USERPRINCIPALNAME()
    ```

5.  **The Result:**
    *   I log in (`me@company.com`).
    *   The `RLS_Users` table filters to show only *my* rows.
    *   That filter propagates to `Dim_Geography`.
    *   `Dim_Geography` propagates to `Fact_Sales`.
    *   I only see my sales.

---

### 4. Key DAX Functions for RLS

| Function | Description | Best Practice |
| :--- | :--- | :--- |
| **`USERPRINCIPALNAME()`** | Returns the User's Email (UPN) (e.g., `alice@company.com`). | **ALWAYS USE THIS.** It matches the Power BI Service login ID (Entra ID/Azure AD). |
| **`USERNAME()`** | Returns `DOMAIN\User`. | Avoid in cloud scenarios. It behaves differently in Desktop vs. Service. |
| **`LOOKUPVALUE()`** | Finds a value in an unconnected table. | Use this if you cannot use physical [[Relationships|relationships]] for security (e.g., complex many-to-many security logic). |
| **`PATHCONTAINS()`** | Checks parent-child hierarchies. | Essential for **HR Hierarchies** (Manager sees their team + their team's team). |

---

### 5. The "Manager Hierarchy" Problem

**Scenario:** A Manager needs to see data for all employees below them in the org chart.
**Solution:** Flatten the hierarchy using `PATH`.

1.  **Org Table:** Contains `EmployeeID` and `ManagerID`.
2.  **[[Calculated Column]]:** `Path = PATH(EmployeeID, ManagerID)`
    *   *Result:* `101|105|200` (The chain of command).
3.  **RLS DAX:**
    ```dax
    PATHCONTAINS( OrgTable[Path], 
                  LOOKUPVALUE(OrgTable[EmployeeID], OrgTable[Email], USERPRINCIPALNAME()) 
    )
    ```
    *   *Translation:* "Is the current user's ID found anywhere in the hierarchy path of this row?"

---

### 6. Object-Level Security (OLS)

**The Cousin of RLS.**
*   **RLS:** Hides *Rows*. (Alice sees 10 rows, Bob sees 20).
*   **OLS:** Hides *Columns* or *[[Tables]]*. (Alice sees the "Salary" column. Bob does not even know the "Salary" column exists—it is physically removed from the model for him).
*   **How to implement:** You cannot do this in Power BI Desktop UI (as of late 2023). You must use **Tabular Editor** (External Tool).
*   **Risk:** Visuals that use the hidden column will **break** (show an 'X' error) for the restricted user. You must handle this carefully.

---

### 7. Pitfalls & Best Practices (Checklist)

#### A. Security Groups vs. Individuals

*   **Bad:** Assigning "alice@company.com" to the role in Power BI Service.
*   **Good:** Assigning the "Sales_Managers" **Security Group** (AD Group) to the role.
*   **Why:** When Alice leaves and Bob joins, IT updates the AD Group. You don't have to touch Power BI.

#### B. The "Publish to Web" Trap

*   **Rule:** RLS **DOES NOT WORK** with "Publish to Web" (Public embed).
*   **Why:** Publish to Web creates an anonymous generic user. The model doesn't know who is viewing it, so it reveals everything.

#### C. Performance

*   **Impact:** The RLS DAX filter is appended to **every single query** the user sends.
*   **Risk:** If your RLS logic is complex (e.g., tons of `LOOKUPVALUE` or `FILTER`), every visual will be slow.
*   **Fix:** Keep RLS logic in the ETL/Data Warehouse. Create a clean, simple `Security_Bridge` table so the DAX is just `[Email] = UPN`.

#### D. Testing

*   **Desktop:** "View As" feature. (Modeling Tab $\to$ View As $\to$ Select Role & User).
*   **Service:** Workspace $\to$ Dataset $\to$ Security $\to$ Ellipsis (...) $\to$ "Test as role".

---

### 8. Summary Table: Where does the logic live?

| Logic Location | Method | Verdict |
| :--- | :--- | :--- |
| **Power BI Desktop** | Static Filters (Role = `[Country]="USA"`) | **Junior.** Hard to maintain. |
| **Database ([[SQL]])** | `Security` Table joined to Fact | **Senior.** Dynamic, centralized, scalable. |
| **DAX Measures** | `IF( USERPRINCIPALNAME() = ..., [Sales], BLANK() )` | **Bad.** Doesn't secure the data, only the number. User can still see dimensions. |
| **Data Source (DirectQuery)** | Single Sign-On (SSO) passing user to [[SQL]] Server | **Advanced.** The DB handles security. Power BI just passes the user identity. Good for compliance. |

### Mnemonic for RLS
**"UPN to the Bridge"**
*   Use **U**SER**P**RINCIPAL**N**AME().
*   Connect it to a **Bridge** (Security) table.
*   Filter the Dimension, let it flow to the Fact.