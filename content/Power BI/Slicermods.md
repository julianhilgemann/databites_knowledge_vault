This is the toolbox you use inside `CALCULATE`.

Think of `CALCULATE` as a room. Before you enter, you have filters on your clothes (Year=2024, City=Berlin).

These functions determine **what clothes you take off or put on** before you step into the room to do the math.

Here is the hierarchy of [[Context]] Modifiers, from "Nuclear" to "Surgical."

---

### 1. The "Nuclear" Option: `ALL` & `REMOVEFILTERS`

**Mechanism:** "Ignore the Slicers completely."

#### A. `ALL(Table)`

*   **Behavior:** Removes **every** filter from that specific table.
*   **Mental Model:** "I don't care what the user selected. Give me the Grand Total of the whole table."
*   **Code:**
    ```dax
    Grand Total = CALCULATE( [Total Sales], ALL(Sales) )
    ```
*   **Warning:** `ALL(Sales)` ignores filters on *Sales*, but if you filter a separate *Date* table, that filter still works (unless Sales is the Fact table and Date propagates to it).

#### B. `ALL(Column)`

*   **Behavior:** Removes filters from **only that specific column**. Other columns in the same table remain filtered.
*   **Mental Model:** "Ignore the Color slicer, but keep the Year slicer."
*   **Code:**
    ```dax
    All Colors Sales = CALCULATE( [Total Sales], ALL(Product[Color]) )
    ```
*   **Use Case:** Denominator for "% of Color Total".

#### C. `REMOVEFILTERS`

*   **Behavior:** Exactly the same as `ALL` when used inside `CALCULATE`.
*   **Difference:** `ALL` returns a table (can be used in `SUMX`). `REMOVEFILTERS` does not return a table (can **only** be used in `CALCULATE`).
*   **Best Practice:** Use `REMOVEFILTERS` for readability when your goal is just to clear filters.

---

### 2. The "Visual" Option: `ALLSELECTED`

**Mechanism:** "Ignore the breakdown *inside* the visual, but keep the filters from the *outside*."

*   **Behavior:** It restores the [[Context|context]] to what it was at the "top" of the visual, before the row/axis split the data.
*   **Mental Model:** "Give me the total for everything visible in this chart."
*   **Use Case:** Calculating **"% of Grand Total"** inside a matrix or chart.

**The Scenario:**
*   Slicer on Page: `Year = 2024`
*   Visual Row: `City = Berlin`
*   **Code:**
    ```dax
    Visual Total = CALCULATE( [Total Sales], ALLSELECTED(Sales) )
    ```
*   **Result:**
    *   It **keeps** `Year = 2024` (External filter).
    *   It **removes** `City = Berlin` (Internal/Visual filter).
    *   *Result:* Total Sales for all Cities in 2024.

---

### 3. The "Freeze" Option: `ALLEXCEPT`

**Mechanism:** "Remove everything **EXCEPT** these specific columns."

*   **Behavior:** It is the inverse of `ALL`. It clears the whole table *unless* the filter is on the column you listed.
*   **Mental Model:** "Lock this column. Make everything else dynamic."
*   **Code:**
    ```dax
    Fixed to Country = CALCULATE( [Total Sales], ALLEXCEPT(Customer, Customer[Country]) )
    ```
*   **Scenario:** If you drill down from Country $\to$ State $\to$ City:
    *   This measure will always show the **Country Total**, ignoring the State/City selection.

---

### 4. The "Constructive" Option: `FILTER`

**Mechanism:** "Add a new, complex filter list."

*   **Behavior:** Iterates through a table, evaluates a condition, and returns a list of rows that match.
*   **Mental Model:** "Go find the specific rows where [Price] > [Cost] and use those."

#### A. `FILTER(Table)` vs. `FILTER(ALL(Column))`
This is a critical performance distinction.

*   **Bad (Slow):**
    ```dax
    FILTER( Sales, Sales[Quantity] > 10 )
    ```
    *   *Why:* Loads the **entire** wide Fact table into memory to check one column.
*   **Good (Fast):**
    ```dax
    FILTER( ALL(Sales[Quantity]), Sales[Quantity] > 10 )
    ```
    *   *Why:* Loads only the unique values of the Quantity column (Dictionary).

---

### 5. The "Intersection" Option: `KEEPFILTERS`

**Mechanism:** "Don't overwrite. Intersect."

*   **[[Context]]:** By default, `CALCULATE` is a bully. If you say `Color="Red"`, it overwrites any existing color filter.
*   **Behavior:** `KEEPFILTERS` tells `CALCULATE` to respect the existing filter and find the **overlap** (AND logic).

**The Real-world Scenario:**
*   Slicer: `Color = "Blue"`
*   **Measure A (Standard):** `CALCULATE( [Sales], Product[Color]="Red" )`
    *   *Result:* **Shows Red Sales.** (The bully overwrites Blue with Red).
*   **Measure B (KeepFilters):** `CALCULATE( [Sales], KEEPFILTERS(Product[Color]="Red") )`
    *   *Result:* **BLANK.** (It looks for rows that are BOTH Blue AND Red. Impossible).

*   **Use Case:** Showing "Top 10 Customers" but respecting a user's slicer selection.

---

### 6. The "Relationship" Option: `CROSSFILTER`

**Mechanism:** "Hack the physical relationship lines."

*   **Behavior:** Changes the direction or activity of a relationship for just one calculation.
*   **Code:**
    ```dax
    Sales by Date = 
    CALCULATE( 
        [Total Sales],
        CROSSFILTER( Sales[OrderDate], Date[Date], None ) -- Turns relationship OFF
    )
    ```
*   **Modes:**
    *   `Both`: Turns on Bi-directional filtering (e.g., Count Dimensions based on Fact).
    *   `None`: Disconnects the table physically.
    *   `OneWay`: Forces filter to flow only one way.

---

### Summary Table for Memorization

| Function | Type | The "Plain English" Command | Use Case |
| :--- | :--- | :--- | :--- |
| **ALL** | Remover | "Clear the board. Ignore user selection." | Denominators for Ratios. |
| **ALLEXCEPT** | Remover | "Clear everything *except* this specific column." | Fixed Totals (e.g., Country Total). |
| **ALLSELECTED** | Preserver | "Clear the visual rows, keep the page slicers." | Visual Totals (% of what I see). |
| **FILTER** | Adder | "Go find rows that match this complex logic." | Price > Cost * 1.5. |
| **KEEPFILTERS**| Intersector| "Don't overwrite. Add this to the list." | "Show Red" (but result is blank if user selected Blue). |
| **REMOVEFILTERS**| Remover | "Just clear filters. Don't give me a table." | Readable alias for ALL(). |
| **CROSSFILTER**| Modifier | "Change the arrow direction on the diagram." | Counting rows in Dimension based on Fact. |

### Technical Best Practice (Best Practice)
> "When removing filters, I prefer **`REMOVEFILTERS`** over `ALL` because it expresses intent clearly.
> When adding complex filters, I filter on **columns** (`ALL(Column)`) rather than [[Tables|tables]] to minimize memory usage.
> I use **`KEEPFILTERS`** inside `CALCULATE` when I want to ensure my measure respects the user's slicers rather than overriding them."