
# Phase 1: The Philosophy (The "Why")

### The "5 Whys" Technique
Never accept the first request at face value.
*   **Stakeholder:** "I need a Pie Chart showing Sales by Country."
*   **You (Junior):** "Okay." (Builds it).
*   **You (Senior):** "Why do you need to see Sales by Country?"
    *   *Stakeholder:* "To see which regions are underperforming."
    *   *You:* "Why does that matter right now?"
    *   *Stakeholder:* "Because we have a marketing budget surplus we need to allocate next week."
    *   *You:* "Ah. So you don't need a Pie Chart. You need a **ranked Bar Chart** highlighting regions with the **lowest market penetration** but **highest potential**."

### The Outcome-Driven Approach

Shift the conversation from **Outputs** (Charts/Exports) to **Outcomes** (Actions).
*   **The Question:** "If this number turns Red tomorrow morning, what physical action will you take?"
    *   If the answer is "Nothing," **do not build that KPI.** It is noise.

---

# Phase 2: The Elicitation Methods (The "How")

### 1. The Interview (One-on-One)
*   **Best for:** C-Level Executives.
*   **Technique:** **Semi-Structured.** Have 5 core questions, but follow the conversation.
*   **Key Question:** "What is the one number you check on your phone before you get out of bed?" (This defines the Main KPI).

### 2. The Workshop (Group)
*   **Best for:** Operational Teams (Sales Managers, Accountants).
*   **Technique:** **Event Storming / Sticky Notes.**
    *   Draw the business process on a whiteboard (Order $\to$ Invoice $\to$ Ship $\to$ Pay).
    *   Ask them to stick "Questions" on the steps where they currently lack visibility.
*   **The Trap:** The "Loudest Voice." Don't let one manager dominate. Use "Silent Brainstorming" (write on sticky notes first, then discuss) to equalize the room.

### 3. Document Analysis (Archaeology)
*   **Best for:** Replacing legacy systems.
*   **Technique:** Ask for their current "Excel Hell" spreadsheets.
*   **What to look for:**
    *   *Hardcoded adjustments:* (e.g., `+ 500` manually added to a total). This is hidden business logic you must migrate to [[dbt]]/[[DAX]].
    *   *Email Distribution Lists:* Who actually reads this?

---

# Phase 3: The Deliverables (The "What")

A Senior Specialist does not just write "Build Dashboard" in Jira. You produce structured artifacts.

### 1. The Bus Matrix ([[Kimball]])
A grid mapping **Business Processes** (Rows) to **Dimensions** (Columns).
*   **Why:** It proves you are thinking about the Enterprise Model, not just one report.
*   **Example:**
    *   *Sales Fact:* Uses Customer, Product, Date.
    *   *Inventory Fact:* Uses Product, Warehouse, Date. (Does NOT use Customer).

### 2. The KPI Definition Card
Don't just name a metric. Define it rigorously.
*   **Name:** Gross Margin %.
*   **Owner:** CFO.
*   **Formula:** `(Revenue - COGS) / Revenue`.
*   **Logic:** "Does Revenue include shipping? Does COGS include rebates?"
*   **Grain:** Calculated at the Line Item level, aggregated to Month.

### 3. The Mockup (Wireframing)
**Never build in Power BI first.** It’s too expensive to change.
*   **Low Fidelity:** Excalidraw / Whiteboard. (Boxes and Arrows).
*   **High Fidelity:** Figma. (Pixel perfect).
*   **The Contract:** "Sign off on this Figma picture. I will build exactly this. Any changes after this point requires a new ticket."

---

# Phase 4: The "User Story" Structure

Write your tickets (Jira/DevOps) using the Agile Syntax.

**Standard:**
> **AS A** Regional Sales Manager,
> **I WANT** to see Year-over-Year growth by Product Category,
> **SO THAT** I can identify which product lines are shrinking and offer discounts.

**Acceptance Criteria (AC):**
*   Must verify against the "Finance_Monthly_Final" Excel file.
*   Must drill through to Transaction Level.
*   RLS must restrict view to the User's Region only.

---

# Phase 5: Handling Difficult Stakeholders (Best Practice)

**Scenario:** Stakeholder asks for something technically bad (e.g., "I want a 50-column table export").

**The "Yes, And..." Pivot:**
*   **Don't say:** "No, Power BI isn't for exporting [[Tables|tables]]."
*   **Do say:** "I understand you need the granular data for reconciliation. **However**, Power BI works best as an aggregated analytical tool. How about we build a high-level exception report that **highlights only the rows that are wrong**, and then we provide a 'Drill-to-Detail' button that gives you the export for just those rows?"

**Scenario:** Scope Creep.
*   **The Fix:** The **"Version 2" Parking Lot.**
    *   "That’s a great idea. To ensure we hit the release date for the MVP (Minimum Viable Product), let’s add that to the V2 backlog. If we finish V1 early, we can pull it forward."

---

# Summary Checklist

If they ask: **"How do you gather requirements?"**

1.  **"I identify the Audience first."** (Strategic vs. Operational).
2.  **"I focus on Decisions, not Charts."** (The '5 Whys').
3.  **"I prototype in Figma."** (Fail fast, fail cheap).
4.  **"I define the Bus Matrix."** (Ensuring Conformed Dimensions).
5.  **"I write rigorous Acceptance Criteria."** (Definition of Done).

### Mnemonic: **"D.A.D.D."**
*   **D**iscover (Interviews/Workshops).
*   **A**nalyze (Excel archaeology/Bus Matrix).
*   **D**esign (Figma/Wireframe).
*   **D**ocument (Jira Tickets/KPI Definitions).