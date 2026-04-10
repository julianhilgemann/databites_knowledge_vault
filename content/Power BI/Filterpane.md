
The key thing to understand is: **To [[DAX]], the Filter Pane is just another source of Filter [[Context]].** However, the *User Experience* implies a hierarchy that [[DAX]] doesn't strictly respect unless you code it carefully.

Here are the specific interactions to be aware of:

### 1. The "Bully" Problem (Overwriting the Filter Pane)

**[[Context]]:** Users treat the Filter Pane as a "Hard Rule." If they set the Page Filter to `Year = 2024`, they expect *everything* on the page to be 2024.

**The Trap:** Standard `CALCULATE` predicates overwrite the Filter Pane.

*   **Scenario:**
    *   Filter Pane (Page Level): `Color = "Blue"`
    *   Measure: `Red Sales = CALCULATE( [Total Sales], Product[Color] = "Red" )`
*   **Result:** The Measure displays **Red Sales**.
*   **Why:** The measure's internal filter (`Color="Red"`) replaces the incoming filter [[Context|context]] from the Pane (`Color="Blue"`).
*   **The Fix:** If you want the measure to return BLANK because Red is not Blue (i.e., respect the user's selection), you must use `KEEPFILTERS`.
    ```dax
    Safe Red Sales = CALCULATE( [Total Sales], KEEPFILTERS( Product[Color] = "Red" ) )
    ```

### 2. `ALL` vs. Visual-Level Filters in the Pane

**Context:** The Filter Pane allows you to add filters to a *specific visual* only.

**The Interaction:**
*   If you apply a Visual-Level filter in the Pane (e.g., "Exclude Unknowns"), and your measure uses `ALL(Table)`, **that filter is ignored**.
*   **Insight:** This often causes "Data Quality" tickets.
    *   *User:* "I filtered out the NULLs in the Filter Pane for this chart!"
    *   *Developer:* "My measure calculates `% of Grand Total` using `ALL(Sales)`, so it put the NULLs back in the denominator."
    *   *Solution:* You need to be specific about *which columns* you are removing filters from (`ALL(Column)`), or apply the data cleaning upstream in Power Query so you don't rely on the Filter Pane to hide bad data.

### 3. `ALLSELECTED` and the "Outer" Context

`ALLSELECTED` is notoriously tricky regarding the Filter Pane because it draws a line between "Inner" filters (the Visual) and "Outer" filters (The Pane/Slicers).

*   **Page/Report Level Filters in Pane:** Treated as "Outer." `ALLSELECTED` **keeps** them.
*   **Visual Level Filters in Pane:** Treated as "Inner." `ALLSELECTED` **removes** them.

**The Trap:**
If a user drags a field into the "Filters on this visual" well to act as a permanent constraint (e.g., "Status = Active"), `ALLSELECTED` will ignore that constraint and calculate the total based on *all* statuses, potentially making your "% of Total" ratio smaller than 100%.

### 4. The "Hidden" Filter Table Trap

In the Filter Pane, you can filter by a column that is **not** in the visual itself.

*   **Scenario:**
    *   Visual: Sales by `Country`.
    *   Filter Pane: `Year = 2024`.
    *   Measure: `CALCULATE( [Sales], ALL(Sales) )`.
*   **Result:** `ALL(Sales)` removes the filter on `Country` (which is in the visual) AND `Year` (which is in the Pane) because they both live in the `Sales` table.
*   **Best Practice:** This is why `ALL(Column)` is safer than `ALL(Table)`.
    *   `ALL(Sales[Country])` removes the visual grouping.
    *   It *keeps* the `Sales[Year]` filter from the Pane.

### 5. Measure Filters in the Pane ([[Context Transition]])

You can drag a **Measure** into the Filter Pane (e.g., "Show items where [Total Sales] > 100").

*   **Mechanism:** This forces the engine to iterate through every single dimension combination in the visual, run the measure, check the result, and then filter.
*   **Performance Warning:** This is extremely expensive (slow).
*   **[[DAX]] Interaction:** If your DAX measure inside the visual *also* does complex [[Context Transition]], having a Measure Filter in the pane adds a second layer of iteration. This is a common cause of "Resource Exceeded" errors.

### Summary Checklist

If asked about the **Filter Pane**:

1.  **"I assume the Filter Pane is the 'Truth' for the user."**
    (I use `KEEPFILTERS` to ensure I don't accidentally override a Page-Level filter they set).
2.  **"I am careful with `ALL(Table)`."**
    (I know it wipes out Report-Level filters in the Pane if they come from that table, which might show historical data the user tried to filter out).
3.  **"I distinguish between Slicers and Visual Filters."**
    (I know `ALLSELECTED` ignores Visual-Level filters in the pane but respects Page-Level filters).

**Mnemonic:**
**"The Pane is a Slicer."**
Don't overthink it. Technically, the Filter Pane is just a collection of Slicers that save screen space. Any DAX logic that applies to a Slicer applies to the Pane.