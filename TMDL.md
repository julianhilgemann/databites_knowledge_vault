Here is a deep dive into **TMDL (Tabular Model Definition Language)**.

### 1. What is TMDL?
**TMDL** is a text-based grammar designed specifically to define Power BI (Tabular) data models.

*   **The Old Way (TMSL/JSON):** A machine-readable format. Heavy use of brackets `{}`, commas, and quotes. [[DAX]] expressions were messy because you had to "escape" every quote (e.g., `\"Column\"`).
*   **The New Way (TMDL):** A human-readable format. It looks very similar to **YAML** or **Python**. It uses **indentation** to define structure.

---

### 2. The Syntax Anatomy
TMDL is optimized for readability. Here is how it handles the core components of a model.

#### A. The Table Definition

Notice the lack of curly braces and commas. Structure is defined by 4-space indentation.

```yaml
table Customer
    lineageTag: a1b2-c3d4-e5f6

    column CustomerKey
        dataType: int64
        isKey
        sourceColumn: CustomerKey
        summarizeBy: none

    column 'Full Name'
        dataType: string
        sourceColumn: FullName
```

#### B. The Measure Definition (The Best Part)

This is the killer feature. In JSON, writing [[DAX]] was painful. In TMDL, **[[DAX]] looks like DAX**.

```yaml
    measure 'Total Sales' = SUM(Sales[Amount])
        formatString: "$#,0.00"
        displayFolder: "Financials"
```

#### C. Block Text (Descriptions & Long DAX)

If you have a long description or a multi-line DAX measure, TMDL uses triple quotes `'''` (like Python docstrings).

```yaml
    measure 'Year over Year Growth' = 
        '''
        VAR CurrentSales = [Total Sales]
        VAR PrevSales = CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Date'[Date]))
        RETURN
            DIVIDE(CurrentSales - PrevSales, PrevSales)
        '''
        description: '''
            Calculates the growth % compared to the same period 
            in the previous year. Returns BLANK if no previous data.
        '''
```

---

### 3. TMDL vs. JSON (The "Why it matters" comparison)

Imagine you want to change the format string of a measure.

**In JSON (The nightmare):**
```json
{
  "name": "Total Sales",
  "expression": "SUM('Sales'[Amount])",
  "formatString": "\\$#,0.00",  <-- Note the escaped backslash
  "annotations": [
    {
      "name": "PBI_FormatHint",
      "value": "{\"isGeneralNumber\":true}"
    }
  ]
}
```

**In TMDL (The clean way):**
```yaml
measure 'Total Sales' = SUM('Sales'[Amount])
    formatString: "$#,0.00"
```

**Key Talking Point:**

> "JSON was designed for machines to read. TMDL is designed for humans to read and edit. It removes the syntactic noise so I can focus on the logic."

---

### 4. The File Structure (Folder Breakdown)

TMDL is not just a language; it is a **serialization strategy**. It breaks the model "Monolith" into small pieces.

When you save as `.pbip` with TMDL enabled, you get a `definition` folder:

1.  **`model.tmdl`**: The entry point. Contains global settings (Culture, Data Access Options).
2.  **`tables/`**: A folder containing **one .tmdl file per table**.
    *   `Sales.tmdl`
    *   `Customer.tmdl`
    *   `Date.tmdl`
3.  **`relationships.tmdl`**: (Optional) Sometimes [[Relationships|relationships]] are defined inside the [[Tables|tables]], sometimes separately depending on complexity.

**Why this wins in [[git|Git]]:**

*   **Conflict Resolution:** If Developer A edits `Sales.tmdl` and Developer B edits `Customer.tmdl`, [[git|Git]] merges them **automatically**. There is zero conflict.
*   **Code Review:** When you open a Pull Request in DevOps, the reviewer sees exactly what changed:
    *   `+ measure 'New KPI' = ...`
    *   They don't see 5,000 lines of JSON noise.

---

### 5. Advanced TMDL Features

#### A. Standard Formatting
TMDL has a strict standard.
*   **Indent:** 4 spaces (Tab is forbidden).
*   **Delimiters:**
    *   Alphanumeric names (`Sales`): No quotes needed.
    *   Special characters/Spaces (`Total Sales`): Single quotes `'Total Sales'` required.

#### B. Perspectives

You can define Perspectives (views of the model) cleanly at the bottom of the file.

```yaml
perspective 'Finance View'
    tables
        Sales
        'Date'
    measures
        'Total Sales'
        'Margin %'
```

#### C. Role-Level Security (RLS)

Roles are defined in TMDL with the same clean syntax.

```yaml
role 'Europe Users'
    modelPermission: read

    tablePermission 'Dim_Geography' = [Region] = "Europe"
```

---

### 6. Tools for Editing TMDL

#### 1. VS Code (The Primary Tool)
*   Install the **"TMDL / TSL"** extension.
*   It provides Syntax Highlighting, folding, and error checking.
*   This is where you do "Find and Replace" operations (e.g., Renaming `[Turnover]` to `[Revenue]` across 50 measures).

#### 2. Power BI Desktop (New - "TMDL View")

*   As mentioned in the "Updates" section, Desktop now has a native text editor view.
*   This allows you to write TMDL without installing VS Code, bridging the gap for less technical developers.

#### 3. Tabular Editor 3

*   TE3 fully supports TMDL serialization. You can toggle between the UI view and the Code view instantly.

---

### 7. Summary: Expert "Power Moves"

If asked about **TMDL**, hit these points to sound like an expert:

1.  **"It enables true Code-First Development."**
    (I can write a script to generate a TMDL file to build a standard Date table for every new project automatically).
2.  **"It solves the [[git|Git]] Merge Nightmare."**
    (Because it splits [[Tables|tables]] into separate files, my team can work in parallel without overwriting each other).
3.  **"It makes Diffing easier."**
    (I can clearly see that a measure formula changed from `SUM` to `AVERAGE` in the Pull Request, without decoding JSON escape characters).
4.  **"I use it for Bulk Refactoring."**
    (I use VS Code Regex replace to update naming conventions across the entire model in seconds).

### Mnemonic
**"TMDL is YAML for DAX."**
(It brings the simplicity of YAML configuration to the complexity of Power BI Data Models).