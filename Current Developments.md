As of **December 2025**, the Power BI and Fabric ecosystem has undergone massive shifts toward "Code-First" development and professional engineering standards.

### 1. User Defined Functions (UDFs)

Microsoft has released **two** distinct types of UDFs. It is critical not to confuse them, as they serve different layers of the stack.

*   **[[DAX]] User-Defined Functions (Preview - Sep 2025):**
    *   **What they are:** You can finally define your own reusable functions in [[DAX]], similar to how you would in Excel or programming languages.
    *   **Syntax:** uses the `DEFINE FUNCTION` keyword.
    *   **Use Case:** Instead of rewriting a complex "Tax Calculation" or "[[Time Intelligence]]" logic inside 50 different measures, you write it once as a Function. You then call `CalculateTax([Sales])` inside your measures.
    *   **Location:** These are authored in the **[[DAX]] Query View** or **[[TMDL]] View**.

*   **Fabric User Data Functions (GA - Sep 2025):**
    *   **What they are:** These are **Python/PySpark** functions designed for the Fabric engineering layer.
    *   **Use Case:** Data Engineers write a cleaning function (e.g., `standardize_address()`) in Python. This function is saved as a native Fabric item and can be called from **Notebooks**, **Pipelines**, and even invoked via API.
    *   **Why it matters:** It stops the "copy-pasting code between notebooks" anti-pattern.

### 2. [[TMDL]] Editor in Power BI Desktop
**Status:** Public Preview (Launched Jan 2025)

*   **The Change:** Power BI Desktop now has a dedicated **"[[TMDL]] View"** tab on the left sidebar (alongside Report, Table, and Model views).
*   **What is TMDL?** *Tabular Model Definition Language*. It is a human-readable, YAML-like syntax for defining your data model.
*   **Capabilities:**
    *   **Code-First Modeling:** You can create measures, calculated columns, and [[Relationships|relationships]] by typing code instead of clicking menus.
    *   **Bulk Edits:** You can copy-paste a block of 50 measures, do a "Find & Replace" (e.g., change "Sum" to "Average"), and apply it instantly.
    *   **[[git|Git]] Integration:** Because TMDL is text-based, it resolves merge conflicts much cleaner than the old JSON format.
*   **VS Code Extension:** There is now a generally available VS Code extension for TMDL that includes semantic highlighting and IntelliSense if you prefer editing outside Desktop.

### 3. DAX Editor in Desktop (DAX Query View)

**Status:** Major Updates (Throughout 2025)

The "new DAX editor" typically refers to the **DAX Query View**, which has evolved from a simple query runner into a full-blown development environment.

*   **Live Model Editing:** You can now write a measure in the Query View and click **"Update Model"** to save it back to the PBIX. You no longer need to write measures in the tiny formula bar.
*   **Copilot Integration:** You can ask Copilot to "Write a query showing sales by region," and it generates the complex DAX query for you.
*   **Direct Lake Support:** As of late 2025, you can "Live Edit" Direct Lake semantic models (from Fabric) directly in Desktop using this view, without needing to switch to the web.

### 4. New Report File System (PBIR)

**Status:** Preview (Becoming Default Jan 2026)

This is the biggest change to the file format in Power BI's history.

*   **PBIP (Project) vs. PBIR (Report):**
    *   **PBIP:** The "Project" file structure that saves your work as a folder instead of a zipped `.pbix` file.
    *   **PBIR:** The **Power BI Enhanced Report Format**. This is the specific format for the *Report* visual layer inside that folder.
*   **The Shift:**
    *   **Old Way:** All report visuals were trapped in one massive, unreadable `report.json` file.
    *   **New Way (PBIR):** Every visual, page, and bookmark gets its **own separate file** in a structured folder system.
*   **Why it matters:**
    *   **Source Control:** If two developers edit different pages, [[git|Git]] can merge them automatically.
    *   **Programmatic Generation:** You can write a script (Python/PowerShell) to generate reports automatically by creating these small JSON files.

### Summary of the "New Workflow"

If you are working in Power BI today (Dec 2025), your "Pro" workflow looks like this:

1.  Save as **PBIP** (using **PBIR** format) for [[git|Git]] versioning.
2.  Use **TMDL View** to write bulk measures and manage the model.
3.  Use **DAX Query View** to test complex logic and create **UDFs**.
4.  Use **Fabric User Functions** for upstream data engineering in Notebooks.