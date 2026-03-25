
# 🧠 The [[DAX]] "Execution Loop" (Meta-Heuristic)

When I write or debug [[DAX]], I run this 4-step [[Simulation|simulation]] in my head:

### 1. The Environment: "What is the Incoming Filter [[Context]]?"

**"What is the box I am standing in?"**
*   Look at the Slicers (Year, Region).
*   Look at the Visual Coordinates (Row Headers, Axis).
*   *Self-Check:* "Am I filtering `Sales[Color]` or `Product[Color]`? Precise lineage matters."

### 2. The Scope: "Do I Accept, Ignore, or Modify the Box?"

**"Do I stay inside the box, or do I break the walls?"**
*   **Accept:** Standard aggregation (`SUM`).
*   **Ignore:** `ALL`, `REMOVEFILTERS` (For denominators).
*   **Modify:** `KEEPFILTERS` (Intersect), `USERELATIONSHIP` (Change path), or Time Intel (Shift the window).

### 3. The Granularity: "What Table am I Iterating?"

**"At what level does the math happen?"**
*   This is the most critical step for advanced logic.
*   Am I doing this math at the **Grand Total** level?
*   Am I doing it row-by-row on the **Product** table?
*   Am I doing it on a **Virtual Table**?
*   *Key Tool:* `VALUES`, `SUMMARIZE`, `ADDCOLUMNS`.

### 4. The Logic: "The Operation & Transition"

**"What is the math, and did I trigger [[Context Transition]]?"**
*   Apply the math: `SUM`, `RANKX`, `DIVIDE`.
*   **The Senior Check:** "Did I wrap this in `CALCULATE`? If so, my Row [[Context]] just transformed into Filter [[Context]]."

---

# 🧩 The Examples (Refined)

### Scenario 1: % of Grand Total
1.  **Environment:** Filtered to "Shoes".
2.  **Scope:** I need to **Ignore** the "Shoes" filter for the denominator. (`ALL`).
3.  **Granularity:** Entire Sales table.
4.  **Logic:** Divide Current / All.

### Scenario 2: Average Sales per Customer
1.  **Environment:** Filtered to "2024".
2.  **Scope:** **Accept** the 2024 filter.
3.  **Granularity:** I need to iterate the **Customer** list, not the Sales transaction list. (Virtual Table `VALUES(Customer)`).
4.  **Logic:** Calculate Sales for each Customer, then Average them. (`AVERAGEX`).

---

# 🔥 The "Pro Tip"

When they ask **"How do you troubleshoot complex [[DAX]]?"** or **"How do you write measures?"**, use this script. It frames you as an Architect who thinks before typing.

> **"I don't just guess syntax. I run a 4-step [[Simulation|simulation]] in my head:**
> 
> 1.  **First, I define the 'Incoming Filter Context'** from the visual.
> 2.  **Second, I decide the Scope:** Do I need to respect those filters, or use `CALCULATE` to remove/modify them (like for a `% of Total`)?
> 3.  **Third, I define the Granularity:** Do I need a Virtual Table to aggregate at a specific grain (like 'Per Customer')?
> 4.  **Finally, I apply the Math.**
>
> This mental model allows me to catch issues like **[[Context Transition]]** bugs or **Granularity Mismatches** before I even write the code."
