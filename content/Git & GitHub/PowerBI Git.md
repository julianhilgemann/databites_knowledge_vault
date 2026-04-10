Here is a comprehensive **[[git|Git]] Primer** specifically tailored for the modern Power BI developer using **[[TMDL]]** and **PBIP**.

---

### 1. The Core Shift: Binary vs. Text

To understand why this is a big deal, you must understand the file formats.

*   **Old Way (`.pbix`):** A zipped binary file.
    *   **[[git|Git]] Behavior:** [[git|Git]] treats it like a generic "blob." If you change one measure, Git sees the *entire file* as changed. You cannot merge work from two people; the last save wins.
*   **New Way (`.pbip`):** A Project Folder.
    *   **Git Behavior:** It splits your report into human-readable text files.
        *   **Dataset:** Saved as **[[TMDL]]** (Tabular Model Definition Language).
        *   **Report:** Saved as **PBIR** (Power BI Enhanced Report Format).
    *   **Result:** If you change one measure, Git records a change to *only* that specific text file (e.g., `Sales.tmdl`). Two developers can work on the same model simultaneously.

---

### 2. The Setup (Enabling Developer Mode)

As of late 2024/2025, these features are becoming standard, but ensure they are enabled:

1.  **Power BI Desktop:** File $\to$ Options $\to$ Preview Features.
    *   Check **Power BI Project (.pbip) save option**.
    *   Check **Store semantic model using [[TMDL]] format**.
    *   Check **Store reports using enhanced metadata format (PBIR)**.
2.  **VS Code:** Install **Visual Studio Code**. This is your new "Command Center."
3.  **Extensions:** Install the **[[TMDL]]** extension for VS Code (for syntax highlighting).

---

### 3. The Anatomy of a Project

When you "Save As" $\to$ `.pbip`, you don't get a file. You get a **Folder Structure**:

```text
MyProject/
│
├── MyProject.pbip          # The launcher (double-click to open PBI Desktop)
├── .gitignore              # Tells Git what to ignore (Auto-generated)
│
├── MyProject.Report/       # The Visuals Layer
│   ├── definition.pbir     # The report definition
│   └── definition/         # Folder containing individual JSON files for every page/visual
│
└── MyProject.SemanticModel/ # The Data Layer (TMDL)
    ├── definition.pbism    # Model connection settings
    └── definition/         # THE MAGIC FOLDER
        ├── tables/         # One .tmdl file per table (e.g., Customer.tmdl)
        ├── cultures/       # Translations
        └── model.tmdl      # Global settings
```

---

### 4. The `.gitignore` (Crucial)

Power BI Desktop automatically creates this file for you. **Never delete it.** It stops you from committing massive data caches or local passwords to Git.

**What it ignores by default:**

*   `*.abf`: The **Data Cache**. We do *not* want 500MB of imported data in GitHub. We only want the *logic* (schema).
*   `localSettings.json`: Your personal machine settings (e.g., "Don't show this popup again").
*   `.pbi/`: Internal temporary folders.

**Implication:** When your colleague clones your repo, they get the **Model structure**, but **Empty [[Tables]]**. They must hit "Refresh" to pull data from [[SQL]]/Source.

---

### 5. The Workflow (Step-by-Step)

#### Step 1: Initialize
Open your `.pbip` folder in VS Code. Open the Terminal (`Ctrl + ~`) and run:
```bash
git init
```

#### Step 2: Track Changes

You make a change in Power BI Desktop (e.g., add `[Total Sales]` measure) and click Save.
*   Switch to VS Code.
*   You will see **1 file changed**: `tables/Sales.tmdl`.
*   Click the file to see the **Diff**:
    *   *Red line:* `// Old code`
    *   *Green line:* `Measure 'Total Sales' = SUM(Sales[Amount])`

#### Step 3: Commit

Stage and commit your changes in VS Code (Source Control tab).
*   **Message:** "Added Total Sales measure to Sales table"
*   **Action:** Commit.

#### Step 4: Branching (Best Practice)

Never work on the `main` branch.
1.  **Create Branch:** `git checkout -b feature/new-kpis`
2.  **Do Work:** Edit in Power BI Desktop $\to$ Save.
3.  **Commit:** Commit to your feature branch.
4.  **Merge/PR:** Push to Azure DevOps/GitHub and create a **Pull Request** to merge `feature/new-kpis` into `main`.

---

### 6. TMDL Specifics (Why it wins)
**TMDL** is designed to look like YAML. It is indentation-based and clean.

**Example of `Product.tmdl`:**
```yaml
table Product
    lineageTag: 3e4f-2a1c...

    column ProductKey
        dataType: int64
        sourceColumn: ProductKey
        summarizeBy: none

    measure 'Product Count' = COUNTROWS('Product')
        formatString: "#,0"
```

**Conflict Resolution:**

If Developer A changes `ProductKey` datatype and Developer B adds a Description to `ProductKey`:
*   **Binary (.pbix):** HARD CONFLICT. Impossible to resolve.
*   **TMDL:** Git usually auto-merges this because they are on different lines. If there is a conflict, you open the text file and simply choose which lines to keep.

---

### 7. Summary Checklist

If asked about **Git integration in Power BI**, hit these points:

1.  **"I use PBIP and TMDL."** (Shows you are current).
2.  **"I exclude data."** (Mention `.gitignore` and `cache.abf`).
3.  **"I review Diffs in VS Code."** (Shows you validate code before committing).
4.  **"I use Branching strategies."** (Feature branches for new requests).
5.  **"I leverage Fabric Git Integration."** (Connect the workspace directly to the Azure DevOps branch so the Service syncs automatically).