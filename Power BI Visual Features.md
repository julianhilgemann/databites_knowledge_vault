
### 1. Visual Calculations (The "Excel" Layer)

**Status:** The new game-changer (GA in 2024).

*   **The Concept:** Writing [[DAX]] calculations that execute **on the visual grid** (Client-Side), rather than in the data model (Server-Side). It works like Excel formulas (A1 + B1).
*   **The Syntax:** It introduces functions like `PREVIOUS()`, `NEXT()`, `RUNNINGSUM()`, and `MOVINGAVERAGE()`.
*   **The "Old Way":** Writing complex [[DAX]] window functions (`OFFSET`, `WINDOW`, `CALCULATE`) just to compare a row to the row above it.
*   **The Senior Use Case:**
    *   "I use Visual Calcs for **Matrix-specific math** (e.g., 'Row 1 vs Row 2'). This keeps my Semantic Model clean. I don't want to pollute the enterprise model with niche measures that only apply to one specific table visual."
*   **The Risk:** If you hide columns in the visual, the calculation might break. It is dependent on the visual layout.

---

### 2. Drill Through (The "Microscope")

**The Concept:** Moving from a Summary Page to a Detail Page, carrying the [[Context|context]] (filters) with you.

#### A. Standard Drill Through
*   **Action:** Right-click a bar $\to$ "Drill through" $\to$ "Details Page."
*   **Setup:** Drag the dimension (e.g., `Product ID`) into the "Drill-through" well on the destination page.

#### B. Button Drill Through (The UX Upgrade)
*   **Action:** User selects a bar. A button lights up "See Details." User clicks the button.
*   **Why:** **Affordance.** Right-clicking is not intuitive for non-technical users. Buttons are obvious.
*   **Setup:** Create a Button $\to$ Action: Drill through $\to$ Destination: [Details Page].

#### C. Cross-Report Drill Through (The Enterprise Move)
*   **Scenario:** You have a "Summary Dashboard" (Dataset A) and a "Granular Transaction Report" (Dataset B).
*   **Action:** Clicking a KPI in Report A opens Report B filtered to that [[Context|context]].
*   **Requirement:** [[Tables]] must share the same schema names or be defined in the settings.

---

### 3. Bookmarks (The "State Manager")

**The Concept:** Saving the **State** of the report (Filters, Visibility, Sort Order).

**The Warning:** "All Visuals" vs. "Selected Visuals."

*   **Scenario:** You make a bookmark to swap a Bar Chart for a Table.
*   **The Bug:** You select "All Visuals" and "Data" (default). A user filters to "Year=2024," then clicks your bookmark. The bookmark **resets** the filter back to what it was when you created it (e.g., "All Years"). The user hates this.
*   **The Fix:**
    1.  Select the specific visuals involved in the swap.
    2.  Update Bookmark setting to **"Selected Visuals"** only.
    3.  Uncheck **"Data"** (so it doesn't touch filters).
*   **Pro Tip:** *"I master the Bookmark pane to create 'App-like' behavior, like Toggles and Pop-ups, but I am extremely careful with the 'Data' property to ensure I don't overwrite user slicer selections."*

---

### 4. Report Page Tooltips (The "Mini-Report")

**The Concept:** When hovering over a visual, show a different, smaller report page instead of the standard black box.

*   **Best Practice:**
    *   Keep it small (320x240 px).
    *   Don't use complex custom visuals inside a tooltip (rendering lag).
    *   Use it to show **Trends** (Sparkline) or **Breakdowns** (Top 5 items) for that specific data point.
*   **Auto-Scale:** Ensure you set "Page View" to "Actual Size" when designing the tooltip page so you know exactly how it looks.

---

### 5. Interaction Settings (Edit Interactions)

**The Concept:** Controlling how Visual A affects Visual B on the same page.

*   **Modes:**
    1.  **Filter:** (The Funnel icon). Removes unrelated data. (Chart resizes).
    2.  **Highlight:** (The Chart icon). Dims unrelated data. (Chart stays same size, unrelated part turns grey).
    3.  **None:** (The Circle Slash). Visual A ignores Visual B.
*   **Senior Preference:**
    *   "I almost always switch from **Highlight** to **Filter**. Highlighting is confusing—users struggle to compare the 'dimmed' bar vs the 'solid' bar. Filtering provides a cleaner view."
    *   "I use **None** on my KPI Cards at the top so that clicking a monthly trend line doesn't filter the 'Yearly Total' card to a single month."

---

### 6. Personalize Visuals (The "Self-Service" Valve)

**The Concept:** Allowing the end-user to change the chart type or measures *without* calling you.

*   **Setup:** Enable "Personalize Visuals" in the Report Settings.
*   **The UX:** User sees a small icon on the chart header. They can swap "Bar Chart" to "Table" or change "Sales" to "Profit."
*   **Governance:** You can use **Perspectives** (in [[TMDL]]) to limit *what* fields they can see when they personalize. (e.g., Don't let them see the raw cost columns).
*   **Pro Tip:** *"I enable Personalize Visuals to reduce 'Ad-Hoc' change requests. If a manager wants to see a chart as a table, they can do it themselves, and I can focus on new features."*

---

### Summary Checklist

| Feature | Junior Usage | Senior Usage |
| :--- | :--- | :--- |
| **Visual Calcs** | Doesn't use them. | Uses them for Matrix math to keep the model clean. |
| **Drill Through** | Right-click default. | **Button** based (better UX) or **Cross-Report** (Enterprise). |
| **Bookmarks** | Breaks filters (Left default settings). | Uses **"Selected Visuals" + Uncheck "Data"** to create safe toggles. |
| **Tooltips** | Standard black box. | **Report Page Tooltips** providing [[Context|context]] (Trend over time). |
| **Interactions** | Default (Highlighting). | **Filter** mode for clarity; **None** mode for fixed KPIs. |
| **Hierarchy** | Uses Drill-down arrows. | Uses **"Stepped Layout"** in Matrix to show hierarchies clearly. |

### Mnemonic: **"B.I.D."** (Button, Interaction, Data)
*   **B**uttons for Drill-through (UX).
*   **I**nteractions set to Filter (Clarity).
*   **D**ata unchecked in Bookmarks (Safety).