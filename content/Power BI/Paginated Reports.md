# Paginated Reports (Report Builder)

While Power BI Desktop builds interactive, analytical dashboards, Paginated Reports (built via Power BI Report Builder or migrating SSRS `*.rdl` files) are designed for pixel-perfect, printable, and exportable operational reporting.

## 1. The Core Architecture

Paginated Reports do not have an internal data model like [[Vertipaq|VertiPaq]]. They are strictly query-forward.
They rely on a "Dataset" that executes a [[DAX]] or [[SQL]] query against a "Data Source" (often a published Power BI semantic model via XMLA or DirectQuery) and streams the flat result set to the renderer.

## 2. Parameters (The $N^2$ Problem)

Parameters in Paginated Reports behave differently than slicers in Power BI Desktop.
When connecting to a Power BI Semantic Model, the query built by Report Builder includes [[DAX]] `RSCustomDaxFilter` functions.

```dax
// Example of DAX built by Report Builder for a Parameter
EVALUATE SUMMARIZECOLUMNS(
    'Date'[Year],
    RSCustomDaxFilter(@DateYear,EqualToCondition,[Date].[Year],String)
)
```

**Warning:** If you have cascading parameters (e.g., Country filters Region, Region filters City), the queries to populate the parameter dropdowns will execute sequentially. If those dropdowns pull from a massive fact table instead of specialized dimension [[Tables|tables]], the report will take minutes just to load the initial UI.

*   **Pro Tip:** Always base parameter datasets on the smallest possible Dimension [[Tables|tables]], never Fact tables.

## 3. Pixel-Perfect vs. Interactive Use Cases

Typical Use Cases:
*   Invoices / Receipts (Highly formatted print artifacts).
*   Data Extracts (Exporting a 50,000-row table to CSV/Excel without Power BI Desktop's visual row limits).
*   Sub-reports (Embedding a detailed table inside a master report).

## 4. Subscriptions and Automation

A primary reason enterprises adopt Paginated Reports is native subscription bursting. You can use Power Automate to loop through a list of clients, pass their `ClientID` dynamically as a URL parameter to the `*.rdl`, render the PDF, and email it to them automatically.

*   URL Parameter Example: `https://app.powerbi.com/groups/{guid}/rdlreports/{guid}?rp:ClientID=12345`

### Summary Checklist
- [ ] Are parameter datasets explicitly pulling from Dimensions instead of Facts?
- [ ] Have cascading parameters been optimized or simplified?
- [ ] Is the report structured to fit cleanly onto an A4/Letter layout without breaking columns across pages?
