
### 1. [[DAX]] Data Types (Power BI Semantic Model)

Power BI loads data into the **[[Vertipaq|VertiPaq]] Engine**. It is highly optimized for compression. The data types here define how the engine stores column structures and how measures calculate results.

| [[DAX]] Data Type | Description | Handling of BLANK | Best Practice & Performance Notes |
| :--- | :--- | :--- | :--- |
| **Whole Number** | 64-bit Integer. <br>*(Values: -9 quintillion to +9 quintillion)* | `BLANK` + 5 = **5**<br>`BLANK` * 5 = **BLANK** | **Best for Keys.**<br>Use for PK/FK columns. Highest compression and fastest join performance. |
| **Fixed Decimal**<br>*(Currency)* | Stores as integer, divides by 10,000 (4 decimal places). | Same as Whole Number. | **Best for Finance.**<br>Avoids "floating point errors" (e.g., receiving 100.0000001). Preferred over "Decimal Number" for money. |
| **Decimal Number** | Floating Point (Double Precision). Can vary in precision. | Same as Whole Number. | **Use with Caution.**<br>Can result in tiny math errors (epsilon). High [[Cardinality|cardinality]] (many unique values) reduces compression significantly. |
| **Text** | Unicode String. | Empty String `""` is **NOT** `BLANK`.<br>`LEN(BLANK)` = **0** | **[[Cardinality]] Killer.**<br>Long text strings or columns with high uniqueness (like UUIDs) bloat model size. Split or remove if not needed. |
| **Date** | Mapped to an Integer internally (Days since 1899-12-30). | Date functions return `BLANK` if input is invalid. | **Split Date & Time.**<br>Don't use `DateTime` in one column. Split into a `Date` column (for [[Relationships|relationships]]) and a `Time` column. |
| **Boolean** | True / False (Stored as 1 / 0). | Treated as False in some logical checks, but usually distinct. | **High Performance.**<br>Very efficient storage. Prefer over text "Yes"/"No". |
| **Variant** | A generic type (can hold text OR number). | N/A | **Measure Output Only.**<br>Columns cannot be Variant. Measures can be (e.g., return a Number, but return text "N/A" on error). Bad for performance charts. |

---

### 2. [[SQL]] Data Types (The Source)

[[SQL]] databases (like [[SQL]] Server, Postgres, Oracle) focus on storage efficiency and strict constraints.

| SQL Data Type | Description | Handling of NULL | Power BI Import Implication |
| :--- | :--- | :--- | :--- |
| **INT / BIGINT** | Standard Integers.<br>(Int = 4 bytes, BigInt = 8 bytes). | `NULL` + 5 = **NULL**<br>(Null propagates). | **Ideal.**<br>Maps directly to [[DAX]] Whole Number. `BIGINT` is safer for massive ID ranges. |
| **VARCHAR(n)** | Variable length non-Unicode text. | `NULL` is distinct from Empty String `''`. | **Conversion.**<br>Power BI converts this to Unicode Text. Efficient in SQL, but PBI will expand it. |
| **NVARCHAR(n)** | Variable length **Unicode** text. | Same as Varchar. | **Native Match.**<br>Matches Power BI's text engine directly. Preferred if your data contains special characters/accents. |
| **DECIMAL(p,s)**<br>*(e.g. 19,4)* | Fixed precision math. Exact storage. | Propagates NULL. | **Maps to Fixed Decimal.**<br>If scale is $\le$ 4, PBI detects as Currency/Fixed Decimal. If scale > 4, PBI forces to Floating Point. |
| **FLOAT** | Approximate Number (Scientific notation). | Propagates NULL. | **Rounding Issues.**<br>Try to cast to DECIMAL in SQL views before importing to ensure totals match exactly. |
| **BIT** | 0 or 1. | Propagates NULL. | **Maps to Boolean.**<br>PBI reads this as True/False automatically. |
| **GUID / UUID** | 16-byte unique identifier. | Propagates NULL. | **Performance Hit.**<br>Import as Text. Very slow for PBI [[Relationships|relationships]] due to size and high uniqueness ([[Cardinality|cardinality]]). Use Integers for keys instead. |

---

### 3. The "Atomic Unit" Comparison: NULL vs. BLANK

This is where the logic breaks between your Source (SQL) and your Model (DAX).

| Feature | **SQL (Source)** | **DAX (Power BI)** |
| :--- | :--- | :--- |
| **Concept** | **Unknown Value.**<br>It is a placeholder for missing data. | **Empty Value.**<br>Can be a database null, or a result of a calculation returning nothing. |
| **Math** | `5 + NULL = NULL`<br>(If I don't know the value, I don't know the sum). | `5 + BLANK = 5`<br>(Blank is treated as zero in addition). |
| **Logic** | `NULL = NULL` is **False** (or Unknown).<br>Must use `IS NULL`. | `BLANK = BLANK` is **True**.<br>`BLANK = 0` is **True** (mostly). |
| **Strings** | `''` (Empty String) $\neq$ `NULL`. | `""` (Empty String) $\neq$ `BLANK` (usually), but sometimes DAX treats empty strings as Blanks in specific functions. |
| **Aggregation** | `COUNT(Column)` ignores NULLs.<br>`COUNT(*)` counts rows including Nulls. | `COUNT` counts numbers (ignores Blanks).<br>`COUNTROWS` counts the table rows (includes Blanks). |

---

### 4. Best Practices for the Pipeline

#### A. Handling Keys (PK/FK)
*   **SQL:** Use `INT` or `BIGINT`. Ensure `NOT NULL` on Primary Keys.
*   **Power BI:** Import as `Whole Number`.
*   **Why?** Text keys (VARCHAR/GUID) are slow to hash and compress. Integers are instant.

#### B. Handling Money
*   **SQL:** Use `DECIMAL(19,4)`.
*   **Power BI:** Change type to `Fixed Decimal Number`.
*   **Why?** Prevents penny-rounding errors in financial reports.

#### C. Handling Text
*   **SQL:** Clean your data. `Trim()` whitespaces.
*   **Power BI:** In Power Query, set "Remove Empty" or "Replace Errors".
*   **Why?** A space `" "` is different from `""` or `BLANK`. This ruins slicers and categorical groupings.

#### D. The "Blank = 0" Trap
*   **Scenario:** You have a visual showing "Average Price".
*   **SQL:** `AVG(Price)` ignores NULLs. (Sum of 10, 10, Null) / 2 = 10.
*   **DAX:** `AVERAGE(Price)` ignores Blanks.
*   **The Trap:** If you create a measure `SUM(Price) + 0`, you turn Blanks into Zeros.
    *   Now the average becomes: (Sum of 10, 10, 0) / 3 = 6.66.
    *   **Rule:** Be very intentional when converting Blanks to 0 (using the `COALESCE` function or `+0`).