

### The Payload

You only need to memorize **8 Functions** and **3 Patterns**.
If you know these cold, you pass. If you get asked anything else, you pivot to logic.

**The 8 Functions:**
1.  **`CALCULATE`** (The King)
2.  **`ALL`** (The Remover)
3.  **`FILTER`** (The Iterator)
4.  **`DIVIDE`** (The Safe Math)
5.  **`SELECTEDVALUE`** (The UX)
6.  **`SWITCH`** (The Logic)
7.  **`RELATED`** (The Lookup)
8.  **`USERPRINCIPALNAME`** (The Security)

**The 3 Patterns:**
1.  **% of Total** (Ratio)
2.  **YoY Growth** (Time Intel)
3.  **Rolling Average** (Advanced Time)

---

### The Core Engine


The Holy Trinity**
*   **Concept:** `CALCULATE` changes [[Context|context]]. `ALL` removes [[Context|context]]. `DIVIDE` prevents errors.
*   **The Drill:** Write this pattern on paper 10 times until you stop hesitating.

```dax
-- Pattern: % of Grand Total
Ratio = 
VAR CurrentValue = [Total Sales]
VAR AllValue = CALCULATE( [Total Sales], ALL( 'Sales' ) )
RETURN
    DIVIDE( CurrentValue, AllValue )
```

**Logic & Lookup**
*   **Concept:** `RELATED` pulls data from the "One" side to the "Many" side. `SWITCH` is cleaner than IF.
*   **The Drill:** Write a [[Calculated Column|calculated column]] logic pattern.

```dax
-- Pattern: Segmentation / Logic
Status = 
SWITCH( TRUE(),
    RELATED( 'Product'[Price] ) > 100, "High Value",
    RELATED( 'Product'[Price] ) > 50,  "Medium Value",
    "Low Value"
)
```

---

###  [[Time Intelligence]]


**Hour 1: The Basics**
*   **Functions:** `TOTALYTD`, `SAMEPERIODLASTYEAR`.
*   **The Drill:** Write the YoY Growth pattern.

```dax
-- Pattern: YoY Growth %
YoY % = 
VAR TY = [Total Sales]
VAR LY = CALCULATE( [Total Sales], SAMEPERIODLASTYEAR( 'Date'[Date] ) )
RETURN
    DIVIDE( TY - LY, LY )
```

**Hour 2: The Hard One (Rolling Avg)**
*   **Functions:** `DATESINPERIOD`.
*   **Why:** This proves you aren't a newbie.
*   **The Drill:** Memorize the arguments: `Dates`, `Start Date` (Max), `Number` (-3), `Interval` (Month).

```dax
-- Pattern: Rolling 3 Months
Rolling 3M = 
CALCULATE(
    [Total Sales],
    DATESINPERIOD( 'Date'[Date], MAX('Date'[Date]), -3, MONTH )
)
```

**Hour 3: Dynamic Titles**
*   **Function:** `SELECTEDVALUE`.
*   **The Drill:**

```dax
Title = "Report for: " & SELECTEDVALUE( 'Date'[Year], "All Years" )
```

---

### Security & RLS


**Hour 1: Security**
*   **Function:** `USERPRINCIPALNAME()`.
*   **The Drill:** Write the RLS rule.
    *   `[Email] = USERPRINCIPALNAME()`

**Hour 2: The "Blanking" Rehearsal**
*   Stop writing code.
*   Look at a blank wall.
*   **Simulate:** "I need to calculate the Rank of a product."
*   **Practice the Pivot:** *"I would use `RANKX`. I need to pass it the Table, but I must wrap the table in `ALL` to remove the filter [[Context|context]], otherwise every product ranks as #1. Then I pass the Measure."*
*   **Simulate:** "I need a Moving Annual Total."
*   **Practice the Pivot:** *"I use `CALCULATE` with `DATESINPERIOD`. I grab the date column, start at the MAX date visible, and look back minus 12 MONTHs."*

**Hour 3: Rest / Light Review**
*   Do not code. Look at your cheat sheets. Visualize the [[Star Schema]].
