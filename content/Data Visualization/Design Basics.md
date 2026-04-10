Here is a curated list of **General UI/UX Principles**, **Psychological Laws**, and **Design Concepts**. This goes beyond just "making things pretty" and focuses on how human brains process interfaces.

---

### 1. The "Laws of UX" (Psychology)
These are heuristic rules observed in human-computer interaction.

| Law | The Concept | Practical Application (Dashboards/Apps) |
| :--- | :--- | :--- |
| **Jakob’s Law** | **"Users spend most of their time on *other* sites."** They expect your site to work like the ones they already know. | Don't reinvent the wheel. Put the Search bar at the top right. Put the Logo at the top left. Use standard icons (Gear for settings, House for home). |
| **Hick’s Law** | **"More choices = More time to decide."** Decision time increases logarithmically with the number of options. | Don't give a user 20 Slicers. Don't show a navigation menu with 15 items. Group them into categories or use drill-downs to reduce cognitive load. |
| **Fitts’s Law** | **"Target size and distance matter."** The time to acquire a target is a function of the distance to and size of the target. | Make "Important" buttons (Submit, Filter) **larger** and place them **closer** to where the mouse cursor usually rests. Don't make users travel across the screen to click "Apply." |
| **Miller’s Law** | **"The Magic Number 7 (±2)."** The average person can only keep ~7 items in their working memory. | Never have a list, a menu, or a set of tabs with more than 7 items. Chunk information into groups. |
| **Doherty Threshold**| **"Speed is trust."** Productivity soars when the computer and user interact at a pace (<400ms) that ensures neither has to wait. | If a query takes >1 second, show a **Skeleton Loader** or a Spinner. If the UI freezes, the user thinks it's broken. |

---

### 2. Gestalt Principles (Visual Perception)

"Gestalt" means "Unified Whole." It explains how the brain groups individual elements into a single object.

#### A. Proximity (The #1 Rule)

**Rule:** Objects that are close to each other are perceived as a group.
*   **Application:** If a Label is far from its Input Field, the user gets confused. If a "Date Filter" is placed next to a specific chart, users assume it *only* filters that chart.
*   **Fix:** Use Whitespace. Put related items (Header + Chart + Metric) tightly together, and separate that group from the next group with wider space.

#### B. Similarity

**Rule:** Objects that look alike are perceived as having the same function.
*   **Application:** All "Action" buttons should be Blue. All "Cancel/Delete" buttons should be Grey/Red.
*   **Mistake:** Making a "Download" button look exactly like a "Navigate" button. The user will click the wrong one.

#### C. Common Region (Enclosure)

**Rule:** Elements sharing an area with a clearly defined boundary are perceived as a group.
*   **Application:** This is the logic behind **Cards** in UI. Placing a faint gray border or shadow around a KPI and a Chart tells the brain "This is one story."

#### D. Continuity

**Rule:** The eye follows the smoothest path. Elements arranged in a line or curve are perceived to be more related than elements not on the line.
*   **Application:** Align your text and visuals to a **Grid**. If your left margins jump around (zigzag), the eye gets tired scanning. Perfect alignment reduces eye strain.

#### E. Closure

**Rule:** The brain fills in missing information to perceive a complete shape.
*   **Application:** You don't need a heavy black box around every visual. Just the alignment of the text and the edge of the chart creates an "invisible line" that the brain perceives as a container. Less ink, same structure.

---

### 3. Visual Design Fundamentals (The UI Layer)

#### A. Hierarchy (Visual Weight)

The user should know immediately what is most important. You control this with:
1.  **Size:** Big = Important.
2.  **Color:** Bright/Saturated = Look here.
3.  **Position:** Top Left = Primary.
4.  **Weight:** Bold font = Header.

**The Squint Test:**
Squint your eyes at the screen until it blurs. What stands out?
*   *Good:* The Main KPI or the "Call to Action" button.
*   *Bad:* The company logo or a decorative background element.

#### B. Whitespace (Negative Space)

Whitespace is not "empty space"; it is an active design element.
*   **Macro Whitespace:** Space between major sections (Dashboard Layout).
*   **Micro Whitespace:** Space between lines of text (Line Height) or between a button text and the button edge (Padding).
*   **Rule:** If it feels cluttered, **add padding**, don't make things smaller.

#### C. Affordance

An object's physical characteristics should suggest how it is used.
*   **Button:** Looks raised or has a different color $\to$ "I can click this."
*   **Input Field:** Looks empty or has a line $\to$ "I can type here."
*   **Mistake:** Flat design gone wrong (a button that is just text with no border/background). Users won't know it's clickable.

#### D. Feedback

For every action, there must be a reaction.
*   **Hover:** Button changes color when mouse is over it.
*   **Active:** Button presses down when clicked.
*   **State:** Toggle switch slides from Left to Right.
*   **Why:** Without feedback, users click 5 times because they think the system didn't register the click.

---

### 4. Accessibility (A11y) - The "POUR" Principles

Standard guidelines (WCAG) to ensure everyone can use your tool.

*   **P - Perceivable:** Content must be visible to senses (Contrast ratio 4.5:1, Alt text for images).
*   **O - Operable:** UI components must be navigable (Keyboard navigation, no keyboard traps).
*   **U - Understandable:** Text is readable, predictable behavior (Error messages explain *how* to fix the error).
*   **R - Robust:** Compatible with tools (Screen readers).

---

### 5. Summary Cheat Sheet

If asked about your **Design Process**, use this flow:

1.  **Empathize:** "I identify the user's goal (Jobs to be Done)."
2.  **Hierarchy:** "I use **Gestalt Principles** (Proximity/Common Region) to group related data."
3.  **Interaction:** "I apply **Fitts's Law** to ensure buttons are accessible and **Hick's Law** to minimize decision fatigue."
4.  **Consistency:** "I adhere to **Jakob's Law** by using standard patterns so users don't have to relearn how to use the interface."
5.  **Refine:** "I use the **Squint Test** to check visual hierarchy and ensure **Whitespace** is used effectively."

### Mnemonic: **CRAP** (The 4 Pillars of Graphic Design)

*   **C - Contrast:** Make different things look different.
*   **R - Repetition:** Repeat styles (fonts, colors) for consistency.
*   **A - Alignment:** Nothing is placed arbitrarily; everything connects visually to something else.
*   **P - Proximity:** Group related items together.