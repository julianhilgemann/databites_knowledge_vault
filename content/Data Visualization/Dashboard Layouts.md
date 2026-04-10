Here is a primer on **Layout Systems, Grids, and Narrative Structures** tailored for High-End Dashboard Design (Power BI, Tableau, or Custom Web Apps).

---

# 1. The Grid System (The Invisible Skeleton)

Before placing a single chart, you define the grid. This ensures consistency, alignment, and scalability.

### The Standard: The 12-Column Grid

Web design relies on the 12-column grid because 12 is the most mathematically flexible number (divisible by 2, 3, 4, and 6).

**How to apply in Power BI:**
*   **The Setup:** Imagine your canvas (e.g., 1280px wide) divided into 12 vertical stripes with gaps (gutters) between them.
*   **The Combinations:**
    *   **Two Large Charts:** Each spans **6** columns.
    *   **Three KPI Charts:** Each spans **4** columns.
    *   **Four Small Metrics:** Each spans **3** columns.
    *   **Sidebar + Main:** Sidebar takes **2** columns; Content takes **10**.

### The 8px / 4px Spatial System

Don't use random pixel values. Use multiples of **8** (or 4 for dense data).
*   **Margins:** 16px or 24px (Outer edge).
*   **Gutters:** 16px (Space between visuals).
*   **Padding:** 8px or 16px (Space inside the visual container).

> **Pro Tip:** In Power BI, turn on "Snap to Grid" and set your grid size to 8px or 10px. Never drag-and-drop visually; type the X/Y/Width/Height numbers in the formatting pane for pixel-perfect alignment.

---

# 2. Scanning Patterns (How Eyes Move)

Users do not read dashboards; they **scan** them. The pattern depends on the density of the information.

### A. The F-Pattern (Information Heavy)

**Best for:** Dense Dashboards, Financial Reports, Grids of Data.
**The Behavior:** The user starts top-left, reads the headline, scans down the left side (row headers), and occasionally scans across to the right when something catches their eye.

**Layout Strategy:**
1.  **Top Bar (The Horizontal Stroke):** Headline, High-level BANs (Big Angry Numbers), and Global Filters.
2.  **Left Column (The Vertical Stroke):** The "Anchor" dimensions (e.g., List of Regions, Products, or Hierarchy).
3.  **Content (The Ribs):** Visuals or Matrix rows extending to the right.

### B. The Z-Pattern (Minimalist / Landing Page)

**Best for:** Executive Summaries, Single-Screen Overviews, "Landing Pages" navigating to other reports.
**The Behavior:** The eye moves from Top-Left to Top-Right, cuts diagonally to Bottom-Left, and finishes at Bottom-Right.

**Layout Strategy:**
1.  **Top-Left:** Logo / Title ([[Context]]).
2.  **Top-Right:** The Primary Interaction (Date Slicer or "Export" button).
3.  **Center/Diagonal:** The "Hero" Chart (The main trend or insight).
4.  **Bottom-Right:** The Call to Action (CTA) or "Drill-through" button.

### C. The "Layer Cake" (The Modern Dashboard)

A variation of the F-Pattern specifically for BI.
*   **Layer 1 (Top):** High-level KPIs (What just happened?).
*   **Layer 2 (Middle):** Trends & Segments (Why did it happen?).
*   **Layer 3 (Bottom):** Granular Data (Who is responsible?).

---

# 3. The Minto Pyramid Principle (The Logic)

Derived from **Barbara Minto (McKinsey)**, this is a communication framework, not a visual one, but it dictates *what* goes into your layout.

**The Rule:** **"Bottom Line Up Front" (BLUF).** Start with the conclusion, then provide the supporting arguments, then the data.

### The Dashboard Translation: SCQA
Structure your layout to tell this story:

1.  **S - Situation (The Title):** "Sales Performance YTD."
2.  **C - Complication (The KPIs):** "Revenue is down 15% vs Budget." (This grabs attention).
3.  **Q - Question (The Charts):** "Why is it down?" (Breakdown by Region/Product showing the underperformer).
4.  **A - Answer (The Table/Drill):** "It is specifically the 'Outdoor' category in the 'West' region."

**Layout Implication:**
*   Don't hide the "Total Sales" number at the bottom. Put it at the top (The Conclusion).
*   Don't start with the 10,000-row table. Put that at the bottom (The Raw Evidence).

---

# 4. Layout Archetypes (The Templates)

When you open a blank canvas, reach for one of these standard archetypes.

### 1. The "L-Frame" (The App Layout)

*   **Structure:** Vertical Sidebar on the Left (Navigation/Slicers) + Horizontal Header on Top (Title/User Profile).
*   **Pros:** Familiar to anyone who uses web apps (SaaS style). Very efficient use of space for navigation.
*   **Cons:** Sidebar reduces horizontal width available for time-series charts.

### 2. The "Bento Box" (Modular Layout)

*   **Structure:** The canvas is divided into distinct rectangular containers (like a Japanese Bento box). Each box holds one metric and its [[Context|context]] (e.g., A KPI + A sparkline + A % change).
*   **Pros:** Highly organized. Great for "Scorecards" where metrics aren't necessarily related (e.g., HR Stats vs. Finance Stats).
*   **Cons:** Can feel disconnected if not grouped correctly.

### 3. The "Newspaper" (Columnar Layout)

*   **Structure:** Content is organized into vertical columns.
    *   *Col 1:* Revenue Metrics.
    *   *Col 2:* Cost Metrics.
    *   *Col 3:* Profit Metrics.
*   **Pros:** Great for comparing parallel metrics side-by-side.
*   **Cons:** Requires the user to scroll vertically if the columns get long.

---

# 5. Gestalt & The "Fold"

### The Fold (Above vs. Below)
Originally a newspaper term, in BI it refers to **Screen Real Estate**.
*   **Above the Fold:** What is visible without scrolling. **Critical KPIs must live here.**
*   **Below the Fold:** Details that require scrolling.

### White Space (Negative Space)
In a Grid system, the empty space is as important as the content.
*   **Active White Space:** Intentional space between your Grid Containers to guide the eye.
*   **Passive White Space:** The space inside the chart (e.g., between the axis and the line).
*   **Senior Rule:** If your dashboard feels cluttered, do not shrink the font. **Remove a chart.**

---

# Summary Checklist for Layouts

| Concept | Junior Designer | Senior Designer |
| :--- | :--- | :--- |
| **Grid** | Eyeballed alignment. | **12-Column Grid** with strict 8px/16px padding. |
| **Flow** | Random placement of charts. | **F-Pattern** for data, **Z-Pattern** for summaries. |
| **Logic** | "Here is all the data I found." | **Minto Pyramid:** Conclusion first, details last. |
| **Slicers** | Scattered near relevant charts. | Consolidated in a **Sidebar** or **Filter Pane**. |
| **Responsiveness** | Breaks on small screens. | Uses **Containers** so elements resize or stack logically. |

### Final Mnemonic for Layouts: **"Grid, Guide, Group"**

1.  **Grid:** Establish the 12-column skeleton.
2.  **Guide:** Use F or Z patterns to guide the eye.
3.  **Group:** Use Minto logic and Containers to group related arguments.