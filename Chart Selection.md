
### 1. The Core Philosophy: "The Job of the Data"
Before choosing a visual, ask: **"What is the relationship I am trying to show?"**

There are only 5 jobs data usually does:

1.  **Comparison** (Ranking items)
2.  **Trend** (Change over time)
3.  **Composition** (Part-to-whole)
4.  **Correlation** (Relationship between two variables)
5.  **Distribution** (Spread/Frequency)

---

### 2. The Selection Matrix

#### A. Comparison (Ranking & Magnitude)

*Goal: Who is #1? Who is last?*

*   **The Standard:** **Bar Chart** (Horizontal).
    *   *Why Horizontal?* It accommodates long category labels (e.g., "Product Names") without tiling your head.
*   **The "Pro" Move:** **Bar Chart + Error Bars.**
    *   *[[Context]]:* Use error bars to show the previous year's value or a target. It’s cleaner than side-by-side bars.
*   **The Specialist Move:** **Small Multiples (Trellis).**
    *   *Scenario:* Comparing Sales by Product across 4 Regions.
    *   *Don't:* Use a Stacked Bar (hard to compare middle segments).
    *   *Do:* Use a Bar Chart with "Region" in the **Small Multiples** well. It creates a grid of 4 clean charts sharing an axis.

#### B. Trend (Time Series)
*Goal: Is it going up or down?*

*   **The Standard:** **Line Chart.**
*   **The "Pro" Move:** **Line Chart + Forecast.**
    *   *[[Context]]:* Enable the "Analytics Pane" features to show a trend line or a forecast confidence interval.
*   **The Specialist Move:** **Deneb / Vega-Lite.**
    *   *Scenario:* You need highly customized Step-Charts or "Slope Charts" (Start vs End only) that Power BI native visuals struggle with.
    *   *Pro Tip:* "For complex time-series where I need specific annotations or conditional markers, I use **Deneb** to write custom JSON specifications."

#### C. Composition (Part-to-Whole)
*Goal: How much does X contribute to the total?*

*   **The Trap:** **Pie Charts.**
    *   *Why:* The human brain struggles to compare angles.
*   **The Standard:** **Donut Chart.** (Better, because you can put the Total KPI in the center).
*   **The "Pro" Move:** **Tree Map.**
    *   *[[Context]]:* When you have hierarchical data (e.g., Category $\to$ SubCategory). It uses rectangular area, which is easier to estimate than slices.
*   **The Specialist Move:** **Decomposition Tree.**
    *   *Why:* It allows the *user* to choose how to break down the composition (Ad-hoc analysis). "Let me break this down by Region... now by Product... now by Sales Rep."

#### D. Correlation ([[Relationships]])
*Goal: Does Spend affect Sales?*

*   **The Standard:** **Scatter Plot.**
*   **The "Pro" Move:** **Scatter Plot with Play Axis.**
    *   *Context:* Animate the correlation over time (Hans Rosling style).
*   **The Specialist Move:** **Key Influencers (AI Visual).**
    *   *Why:* Instead of forcing the user to interpret the scatter plot, use the AI visual to say: *"When Discount increases by 10%, Likelihood to Churn decreases by 1.2x."*

#### E. Target vs. Actual (Status)
*Goal: Are we winning?*

*   **The Trap:** **Gauge Chart.** (Takes up huge space for 1 number).
*   **The Standard:** **KPI Card.**
*   **The Specialist Move:** **Bullet Chart (Custom Visual or Deneb).**
    *   *Anatomy:* A solid bar (Actual) inside a wider bar (Qualitative Range/Target) with a marker line (Goal).
    *   *Why:* It packs a massive amount of context into a tiny footprint. ([[IBCS]] standard).

---

### 3. The "Anti-Patterns" (What to avoid)

In a professional setting, aggressively denounce these:

1.  **3D Charts:** They distort data perspective. Never use them.
2.  **Dual Y-Axis (with different scales):**
    *   *Scenario:* Sales ($) on Left, Margin (%) on Right.
    *   *Risk:* Lines cross each other visually, implying a correlation where there isn't one.
    *   *Solution:* Small Multiples (stack them vertically).
3.  **Stacked Area Charts:**
    *   *Risk:* It is impossible to read the trend of the "middle" layers because their baseline keeps moving.
    *   *Solution:* Line chart or Panel Chart.

---

### 4. Advanced: The "New Card" Visual (Core 2024/2025 Skill)

Power BI released the **"New Card Visual"** (multi-card). It is a game changer.

*   **Old Way:** 5 separate cards, aligned manually, grouped with a rectangle shape.
*   **New Way:** One visual container holding 5 measures.
    *   **Features:** Built-in images, accent bars, sub-labels, and specific conditional formatting per metric.
    *   **Performance:** It renders **1 query** instead of **5 queries**.
    *   **Pro Tip:** "I strictly use the New Card visual to consolidate KPIs. It reduces the DOM elements on the page, improving rendering performance by 20-30%."

---

### Summary Table: The "Cheat Sheet"

| Goal | Junior Choice | Senior Choice | Why? |
| :--- | :--- | :--- | :--- |
| **Compare Categories** | Column Chart | **Bar Chart (Horizontal)** | Better for reading labels. |
| **Trend** | Line Chart | **Line + Small Multiples** | Avoids "Spaghetti charts" (too many overlapping lines). |
| **Hierarchy** | Pie Chart | **Decomposition Tree** | AI-driven exploration is superior to static slices. |
| **Performance** | Gauge | **Bullet Chart** | Information density ([[IBCS]] standard). |
| **KPIs** | Many Cards | **New Card (Multi)** | 1 Query vs. Many Queries (Performance). |
| **[[Tables]]** | Native Table | **Matrix** | Allows grouping/collapsing (Drill-down). |

### Mnemonic: **"C.T.C"** (Context, Type, Clarity)

1.  **Context:** Do I need a target? (Bullet).
2.  **Type:** Is it time or category? (Line vs Bar).
3.  **Clarity:** Can I remove ink? (Small Multiples).