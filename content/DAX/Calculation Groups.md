
### 1. The Core Concept: "Measure Bloat Killer"

**The Problem (The Old Way):**

You have 3 base metrics: `Sales`, `Cost`, `Margin`.
The business wants to see them as: `YTD`, `QTD`, `MTD`, `YoY`, `YoY %`.
*   **Result:** $3 \times 5 = 15$ separate measures to write and maintain.
*   **Maintenance Nightmare:** If the logic for `YTD` changes, you have to fix it in 3 places.

**The Solution (Calculation Groups):**

You write the [[Time Intelligence ]]logic **ONCE**.
You apply it to **ANY** measure.
*   **Result:** 3 Base measures + 1 Calculation Group.

---

### 2. How it Works (The Mechanics)

A Calculation Group creates a physical table in your model with two columns:
1.  **Name:** The label the user sees (e.g., "YTD", "Current").
2.  **Ordinal:** The sort order (0, 1, 2...).

Behind the scenes, it uses a special placeholder function: **`SELECTEDMEASURE()`**.

#### The Syntax Pattern
When a user selects "YTD" from the Calculation Group slicer (or puts it in a Matrix column), the engine effectively wraps your measure like this:

```dax
CALCULATE(
    SELECTEDMEASURE(),  -- This becomes [Sales], [Cost], or [Margin] dynamically
    DATESYTD( 'Date'[Date] )
)
```

---

### 3. The Top 3 Use Cases (Best Practice)

#### Case A: Standard [[Time Intelligence]]
Instead of writing `Sales YTD`, `Sales LY`, `Sales YoY`, you create a "Time Intel" group.

*   **Item: Current** $\to$ `SELECTEDMEASURE()`
*   **Item: YTD** $\to$ `CALCULATE(SELECTEDMEASURE(), DATESYTD('Date'[Date]))`
*   **Item: YOY %** $\to$
    ```dax
    VAR CurrentVal = SELECTEDMEASURE()
    VAR PrevVal = CALCULATE(SELECTEDMEASURE(), SAMEPERIODLASTYEAR('Date'[Date]))
    RETURN DIVIDE(CurrentVal - PrevVal, PrevVal)
    ```

#### Case B: Currency Conversion ([[Dynamic Formatting]])
This is the **Superpower** of Calculation Groups. They can change the **Format String** dynamically.

*   **Item: USD** $\to$ `SELECTEDMEASURE()` (Format: `$#,0.00`)
*   **Item: EUR** $\to$ `SELECTEDMEASURE() * [ExchangeRate]` (Format: `#,0.00 €`)

*   **Pro Tip:** *"I use Calculation Groups for currency because it allows me to not only convert the number but also change the axis symbol from $ to € automatically, which standard measures cannot do."*

#### Case C: "Toggle" Metrics (Variable Axis)
You want a chart where the user can switch between "Revenue" and "Units Sold" without bookmarks.

*   Create a Calc Group "Metric Selector".
*   **Item: Revenue** $\to$ `[Total Revenue]`
*   **Item: Units** $\to$ `[Total Units]`
*   **Setup:** Put a dummy measure `1` in the visual, and filter the visual using the Calc Group. The Calc Group overwrites the `1` with the actual measure logic.

---

### 4. Advanced: Ordering & Precedence

What happens if you have **Two** Calculation Groups applied to the same visual?
*   Group 1: **[[Time Intelligence]]** (YTD).
*   Group 2: **Currency** (Convert to EUR).

Does it convert to EUR, then do YTD? Or do YTD, then convert to EUR?

**The Precedence Property:**
Every Calculation Group has a `Precedence` integer (0, 10, 20...).
*   **Higher Precedence = Evaluation Inner Layer.**
*   **Lower Precedence = Evaluation Outer Layer.**

You must ensure the order makes sense mathematically (usually Time Intel first, then Currency).

---

### 5. The "Gotchas" (Specialist Knowledge)

#### A. Recursion
If you reference a measure inside a Calculation Item (e.g., `CALCULATE([Sales], ...)`), the Calculation Group might try to apply itself *to itself* infinitely.
*   **Fix:** `CALCULATE( [Sales], REMOVEFILTERS( 'CalcGroupTable' ) )`. This prevents the recursion loop.

#### B. Incompatible Measures
You don't want "YTD" applied to a measure like `[Margin %]` (Adding percentages over time is bad math).
*   **Fix:** Use `ISSELECTEDMEASURE` to exclude specific metrics.
    ```dax
    IF(
        ISSELECTEDMEASURE( [Margin %] ),
        SELECTEDMEASURE(), -- Return raw value, ignore YTD logic
        CALCULATE( SELECTEDMEASURE(), DATESYTD(...) )
    )
    ```

#### C. Tooltips
Calculation Groups apply to **everything** in the visual, including Tooltips. Sometimes this breaks the tooltip [[Context|context]].
*   **Fix:** You might need to exclude the tooltip measures using the logic above.

---

### 6. Tools for Authoring

1.  **Tabular Editor 2/3:**
    *   Historically the only way to do it.
    *   Still the fastest way. You can script the creation of 10 items in seconds.
2.  **Power BI Desktop (Model Explorer):**
    *   As of late 2023/2024, you can create them natively in the "Model" view.
    *   It is slower than Tabular Editor but easier for beginners.
3.  **[[TMDL]]:**
    *   Calculation Groups are defined cleanly in text files. Very easy to copy-paste logic between projects.

---

### Summary Checklist

1.  **"I use them for DRY."**
    (Don't Repeat Yourself. I reduce measure count by 80%).
2.  **"I leverage Dynamic Format Strings."**
    (Changing $ to % or € depending on the selection).
3.  **"I manage Precedence."**
    (I understand how multiple groups stack on top of each other).
4.  **"I use Tabular Editor."**
    (I don't click buttons manually; I script the creation of standard time intel groups).

### Mnemonic: **"Wrap and Map"**
*   **Wrap:** It **Wraps** the existing measure in new logic (`CALCULATE`).
*   **Map:** It **Maps** user selection (YTD) to [[DAX]] functions (`DATESYTD`).