# Composite Models (DirectQuery for Power BI datasets)

Composite Models allow an architect to build a global, governed Enterprise Semantic Model, and then allow business analysts to chain off that model, adding their own departmental data (Excel files, SharePoint lists) without duplicating the massive enterprise data.

## 1. The Hub and Spoke Architecture

*   **The Hub:** A monolithic, highly governed import-mode semantic model containing core dimensions (Date, Employee, Client) and core facts (Sales, Budgets). Maintained by IT.
*   **The Spoke:** A thin report/model created by an analyst. They connect to the Hub using DirectQuery, then Import their local Excel file containing departmental targets. They create [[Relationships|relationships]] between the local Import table and the remote DirectQuery table.

## 2. Performance Implications

This architecture is revolutionary for data governance (a single version of the truth), but it has massive performance hazards.

When a visual requires data from both the remote Hub and the local Spoke, the [[Vertipaq|VertiPaq]] engine must:
1. Generate [[DAX]] to query the Hub.
2. Receive the summary table over the network.
3. Mash it up internally against the local Excel data in the formula engine.

If the [[Cardinality|cardinality]] of the join column is very high (e.g., joining on Transaction ID), the network payload will be enormous, and the formula engine will crash or perform terribly. Join only on low-[[Cardinality|cardinality]] dimension keys (e.g., Region, Product Category).

## 3. Storage Modes

To optimize a Composite Model, you must understand the storage modes:
*   **Import:** Data sits in local memory. Fastest.
*   **DirectQuery:** Queries are sent live to the remote source ([[SQL]] or remote Power BI Dataset). Slower.
*   **Dual Mode:** A dimension table is both Imported (for fast local joins) and DirectQuery'd (for remote joins).

*   **Pro Tip:** "Dual" mode is the silver bullet for resolving Composite Model performance bottlenecks. If you build a Date table locally, set it to Dual mode so that local measures resolve locally, and remote fact measures resolve remotely without moving data unnecessarily.

## 4. Security Lineage
If the remote "Hub" model has [[Row Level Security]] (RLS) applied, the identity of the end-user viewing the "Spoke" report is inherently passed through via Single Sign-On (SSO). RLS rules cannot be bypassed by chaining models.

### Summary Checklist
- [ ] Are cross-island [[Relationships|relationships]] strictly using low-cardinality dimensions?
- [ ] Are common dimensions set to "Dual" storage mode?
- [ ] Is the business aware that chaining models adds a layer of rendering latency?
