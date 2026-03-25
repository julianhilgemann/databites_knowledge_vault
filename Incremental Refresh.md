# Incremental Refresh & Partitions

For enterprise-scale datasets (typically Fact [[Tables|tables]] > 10 million rows or > 1GB), performing a full snapshot refresh every night is unacceptable. It drains source database resources, slows down the Power BI Premium capacity, and causes timeout failures.

## 1. The Core Architecture
Incremental Refresh relies on [[SQL]] Server Analysis Services (SSAS) partitions under the hood. Power BI dynamically manages these partitions based on a rolling window policy.

### The Required Parameters
To set this up, you must strictly define two `DateTime` parameters in Power Query:
*   `RangeStart`
*   `RangeEnd`

You must filter the large Fact table natively in Power Query using exactly `[DateColumn] >= RangeStart and [DateColumn] < RangeEnd`.

*   **Warning:** This step **must** achieve [[Query Folding]] back to the source database. If the date filter does not fold, the gateway will download the entire historical table into memory to filter it, entirely defeating the purpose of Incremental Refresh.

## 2. Policy Configuration Configuration

Once the parameters are set, you define the policy in Power BI Desktop on the table:
*   **Store rows in the last:** (e.g., 5 Years) -> This is the *Historical Window*.
*   **Refresh rows in the last:** (e.g., 3 Days) -> This is the *Rolling Window*.

*   **Pro Tip:** The rolling window should always account for the source system's "late-arriving data" latency. If an order from yesterday might get updated today, set the refresh window to at least 3-5 days to ensure the partition picks up the changes.

## 3. Detect Data Changes (Premium Feature)

If you have a Premium capacity, you can check the box for "Detect data changes". This requires a specialized column (usually a `Last_Updated_DateTime` or a high watermark `RowVersion` integer) on the source database. Power BI will only refresh partitions in the rolling window if the maximum value of this column has changed since the last refresh.

## 4. Under the Hood (Tabular Editor & XMLA)

When you deploy a dataset with Incremental Refresh configured via the XMLA endpoint, you can open Tabular Editor and map to the workspace.
You will see that Power BI has created a series of partitions on the Fact table (e.g., `2019`, `2020`, `2021`, `2024-Q1`, `2024-03-24`, etc.).
Using TMSL via [[SQL]] Server Management Studio (SSMS), you can manually process a single historical partition if source data changes exceptionally.

### Summary Checklist
- [ ] Are `RangeStart` and `RangeEnd` defined exactly as `DateTime` [[Data Types|data types]]?
- [ ] Does the date filter step achieve `Query Folding` in Power Query?
- [ ] Have you accounted for late-arriving dimensions/facts in the rolling window?
