
### 1. The Pre-Requisite: The Date Table
Time Intelligence **does not work** without a proper Date Table.

*   **Rule 1:** Must be a contiguous range of dates (no gaps).
*   **Rule 2:** Must cover the entire year (Jan 1 to Dec 31) even if sales started in March.
*   **Rule 3:** **Mark as Date Table** in the model settings. This tells the engine to turn off auto-summarization on the Year column and creates the necessary internal metadata for [[DAX]] functions.

---

### 2. Standard Patterns (Gregorian Calendar)
*Use these when the company uses a standard Jan-Dec year.*

#### A. Cumulative (YTD)
```dax
Sales YTD = TOTALYTD( [Total Sales], 'Date'[Date] )
```
*   **Fiscal Variation:** `TOTALYTD( [Sales], 'Date'[Date], "06-30" )` (Ends June 30th).

#### B. Same Period Last Year (YoY)
```dax
Sales LY = CALCULATE( [Total Sales], SAMEPERIODLASTYEAR( 'Date'[Date] ) )
```

#### C. Moving Annual Total (MAT / Rolling 12 Months)
Great for smoothing out seasonality trends.
```dax
Sales R12M = 
CALCULATE(
    [Total Sales],
    DATESINPERIOD( 'Date'[Date], MAX( 'Date'[Date] ), -12, MONTH )
)
```

---

### 3. The "Universal" Pattern (Custom Logic)
*Use this when standard [[DAX]] functions fail (e.g., 4-4-5 Retail Calendars, strange Fiscal Years, or strict control requirements).*

**The Logic:** Stop using "Time Intelligence Functions" and start using **Filter [[Context]] Manipulation**.

#### A. Custom YTD (The "Filter/All" Pattern)
This works on *any* calendar if you have a `Year` and `DayOfYear` column.

```dax
Sales YTD Custom = 
VAR MaxDate = MAX( 'Date'[Date] )
VAR CurrentYear = MAX( 'Date'[Year] ) -- Or Fiscal Year
RETURN
    CALCULATE(
        [Total Sales],
        'Date'[Year] = CurrentYear,
        'Date'[Date] <= MaxDate,
        REMOVEFILTERS( 'Date' ) -- Clear existing filters to grab the whole year so far
    )
```

#### B. Week-Based Comparison (Retail Standard)
**The Problem:** `SAMEPERIODLASTYEAR` compares Jan 1st 2024 (Monday) to Jan 1st 2023 (Sunday). Retailers hate this because weekends don't align.
**The Fix:** Compare **Week 1 vs Week 1**.

**Pre-requisite:** An `ISO_Week_Index` (Integer) in your Date Table (1, 2, 3... running forever).

```dax
Sales Last Week Year (LWY) = 
VAR CurrentWeekIndex = SELECTEDVALUE( 'Date'[ISO_Week_Index] )
RETURN
    CALCULATE(
        [Total Sales],
        ALL( 'Date' ),
        'Date'[ISO_Week_Index] = CurrentWeekIndex - 52
    )
```
*   **Pro Tip:** *"For Retail clients, I ignore standard Time Intelligence. I implement a **Week Offset** logic to ensure holidays and weekends align perfectly year-over-year."*

---

### 4. Handling "Incomplete Periods" (The Unfair Comparison)

**The Problem:**
*   It is March 15th.
*   **Sales YTD:** Jan 1 to Mar 15.
*   **Sales LY:** Jan 1 to Dec 31 (if you just use `CALCULATE([Sales], YEAR=2023)`).
*   **Result:** A massive negative variance because you are comparing 2.5 months against 12 months.

**The Fix:**
You must filter last year to **match the max date of the current year**.

```dax
Sales LY (Fair) = 
VAR MaxDateAvailable = CALCULATE( MAX( Sales[OrderDate] ), REMOVEFILTERS() ) -- Last actual sale
VAR CurrentContextMaxDate = MAX( 'Date'[Date] )
VAR RealMax = MIN( MaxDateAvailable, CurrentContextMaxDate ) 

RETURN
    CALCULATE(
        [Total Sales],
        SAMEPERIODLASTYEAR( 'Date'[Date] ),
        'Date'[Date] <= RealMax -- Caps last year at March 15th
    )
```

---

### 5. [[Calculation Groups]] (The Senior Architecture)

**The Problem:**
You have measures: `Sales`, `Cost`, `Margin`, `Qty`.
You need: `YTD`, `QTD`, `MTD`, `YoY`, `YoY %` for **all of them**.
Result: $4 \times 5 = 20$ Measures. (Measure Bloat).

**The Solution:**
Create **One** Calculation Group called `Time Intelligence`.

**Items in Calculation Group:**
1.  **Current:** `SELECTEDMEASURE()`
2.  **YTD:** `CALCULATE( SELECTEDMEASURE(), DATESYTD( 'Date'[Date] ) )`
3.  **YOY %:**
    ```dax
    VAR Current = SELECTEDMEASURE()
    VAR Prev = CALCULATE( SELECTEDMEASURE(), SAMEPERIODLASTYEAR( 'Date'[Date] ) )
    RETURN DIVIDE( Current - Prev, Prev )
    ```

**The Usage:**
*   Put `Time Intelligence[Name]` in the Columns of a Matrix.
*   Put `[Total Sales]` (and Cost, Margin) in the Values.
*   **Result:** The calculation group applies the logic dynamically to whatever measure is in the visual.

---

### 6. The "Month-End" Trap (Edge Case)

**The Scenario:**
*   Current Date: March 31st.
*   Previous Month: February (ends on 28th).
*   `DATEADD( ... -1, MONTH )`: Tries to find Feb 31st. Returns **BLANK**.

**The Fix:**
Decide if you want **Strict Date Shifting** or **Period Shifting**.

*   **`DATEADD`**: Shifts the specific dates. (Good for daily trends).
*   **`PARALLELPERIOD`**: Shifts the **entire container** (the whole month of Feb).

```dax
-- Safe Month Comparison
Sales PM (Safe) = 
CALCULATE( 
    [Total Sales], 
    PARALLELPERIOD( 'Date'[Date], -1, MONTH ) 
)
```
*   *Note:* `PARALLELPERIOD` ignores the granular days selected and gives you the total for the whole parallel month.

---

### Summary Table

| Scenario | Pattern | Why? |
| :--- | :--- | :--- |
| **Standard Reporting** | `TOTALYTD`, `SAMEPERIODLASTYEAR` | Fast, standard, works for 90% of cases. |
| **Retail / 4-4-5** | **Filter Manipulation** (`ALL`, `FILTER`) | Native functions fail on non-Gregorian calendars. |
| **Day Alignment** | **Week Index Offset** | Ensures Monday compares to Monday. |
| **Measure Bloat** | **[[Calculation Groups]]** | Reduces 100 measures down to 5 base measures + 1 Calc Group. |
| **Lagging Data** | **Fair Comparison Logic** | Prevents comparing "Full Month" LY vs "Half Month" TY. |

### Mnemonic: **"Standard, Custom, Group"**

1.  **Standard:** Use native [[DAX]] functions first.
2.  **Custom:** If Fiscal/Retail, use Filter logic (Offsets).
3.  **Group:** Use [[Calculation Groups]] to scale it.