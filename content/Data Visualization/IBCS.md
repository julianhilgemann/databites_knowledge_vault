
# The SUCCESS Formula

**Definition:** A consistent visual language for business reporting. Just as musicians can read sheet music instantly because the notation is standard, executives should be able to read reports instantly without relearning the design every time.

**The Goal:** Maximize information density and minimize cognitive load.

---

## The SUCCESS Formula

### 1. **S**ay (Convey a Message)

Charts should not just display data; they should tell a story.
*   **The Rule:** Every chart or page must have a message.
*   **Power BI Implementation:**
    *   Use **Dynamic Titles** (via [[DAX]]) that state the insight (e.g., *"Revenue growing 5% vs PY"*), not just the content (*"Revenue by Year"*).
    *   **Action:** [[Dynamic Formatting]]

### 2. **U**nify (Standardize Notation)

This is the most recognizable part of IBCS. Use the same visual language across all reports.
*   **The Semantic Notation:**
    *   **Actuals:** Solid Black / Dark Grey.
    *   **Budget / Plan:** Hollow (Outlined) bars.
    *   **Forecast:** Hatched (Striped) bars.
    *   **Previous Year:** Light Grey.
    *   **Variance (Good):** Green.
    *   **Variance (Bad):** Red.
*   **Power BI Implementation:**
    *   Create a JSON Theme file that enforces these colors.
    *   Use **[[Calculation Groups]]** or Field Parameters to ensure comparisons always use the correct colors (e.g., Cost increasing = Red, Revenue increasing = Green).

### 3. **C**ondense (Increase Density)

Fit more information into less space without clutter.
*   **The Rule:** High information density allows for [[Context|context]].
*   **Power BI Implementation:**
    *   Use **Small Multiples** (Trellis Charts) instead of 5 separate visuals or a slicer.
    *   Use **Sparklines** in table columns.
    *   Use the **New Card Visual** to group multiple KPIs tightly.

### 4. **C**heck (Ensure Visual Integrity)

Avoid misleading the user.
*   **The Rule:** Visual scaling must be consistent.
*   **The "Lie Factor":**
    *   **Axes:** Bar charts **must** start at 0.
    *   **Scaling:** If Chart A shows \$1M and Chart B shows \$100k, they cannot be the same size unless the scaling is synchronized.
*   **Power BI Implementation:**
    *   Use "Auto" axis carefully.
    *   If comparing side-by-side charts, hard-code the Axis Max to be the same using a [[DAX]] measure.

### 5. **E**xpress (Choose Proper Visualization)

Select the correct chart for the data relationship.
*   **The Rule:**
    *   **Time:** Horizontal Axis (Columns/Lines).
    *   **Structure/Category:** Vertical Axis (Bars).
    *   **Variance:** Waterfall or "Pin" charts.
*   **Power BI Implementation:**
    *   Avoid Pie Charts (IBCS strictly forbids them). Use Tree Maps or Stacked Bars.
    *   Refer to [[Chart Selection]].

### 6. **S**implify (Avoid Clutter)

Remove everything that is not data.
*   **The Rule:** Maximize the **Data-Ink Ratio**.
*   **Power BI Implementation:**
    *   Remove: Background images, heavy borders, gridlines (unless necessary), decorative colors, 3D effects.
    *   Simplify labels: Use "2024" instead of "Year 2024". Use "M" for Millions.

### 7. **S**tructure (Organize Logic)

Data should flow logically.
*   **The Rule:** Follow the **Minto Pyramid Principle**.
*   **Power BI Implementation:**
    *   **Top:** Executive Summary / Conclusion (BANs).
    *   **Middle:** Trends and Drivers (Bridge/Waterfall).
    *   **Bottom:** Details and Data ([[Tables]]).
    *   Ensure **Drill-through** paths follow this logic.

---

## 💡 The "Specialist" Implementation

Power BI's native visuals struggle to be 100% IBCS compliant (e.g., it's hard to make "Outlined" bars for Budget without hacks).

**How to sound like an Expert to Stakeholders:**

1.  **The "Workaround" Answer:**
    > "I implement IBCS principles using native visuals by enforcing a strict Color Theme (Black for Actuals, Grey for PY) and using Error Bars to represent Budgets/Targets efficiently."

2.  **The "Pro" Answer (External Tools):**
    > "For strict IBCS reporting (like P&L statements), native visuals fall short. In those cases, I prefer using **Zebra BI** (Certified Custom Visual) or building custom specifications in **Deneb (Vega-Lite)**. This allows me to create proper Variance Pin charts and Waterfalls that standard Power BI cannot do."

---

## 🔗 Related Notes
*   [[Data Visualization Design]]
*   [[Chart Selection]]
*   [[Dashboard Layouts]]