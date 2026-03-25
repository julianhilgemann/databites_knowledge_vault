# Spaced Repetition (Anki)

This document contains high-yield, theoretical Question/Answer pairs designed to be imported into spaced-repetition software like **Anki**.

By committing these foundational mechanics to memory, you will be able to instantaneously debug complex [[DAX]] and data modeling problems in your daily work. 

## The Core Deck (DAX & Architecture)

You can copy-paste these, or convert the table into a CSV for bulk Anki importing.

| Front (Question) | Back (Answer) |
| :--- | :--- |
| **What is the difference between Filter [[Context]] and Evaluation Context?** | Evaluation Context is the overarching term. It consists of two entirely separate halves: **Filter Context** (what data is visible) and **Row Context** (the current row iterating). |
| **What does [[Context Transition]] do, fundamentally?** | It transforms a Row Context into an equivalent Filter Context. This happens automatically when you invoke `CALCULATE()` or implicitly when you call a Measure inside an iterator (like `SUMX`). |
| **What are the three encoding types used by the [[Vertipaq|VertiPaq]] engine?** | 1. Value Encoding (mathematical reduction). <br> 2. Dictionary / Hash Encoding (mapping distinct strings to integers). <br> 3. Run-Length Encoding / RLE (compressing repeated sequential values). |
| **What determines if VertiPaq can use Run-Length Encoding (RLE) effectively?** | The physical sort order of the data when it is loaded. VertiPaq sorts by the lowest [[Cardinality|cardinality]] columns first to group repeating values and maximize RLE compression. |
| **What is the "Golden Rule" of DAX Calculate?** | `CALCULATE` completely modifies or overrides the existing Filter Context with its own filter arguments *before* evaluating the core expression. |
| **Why should you use `ADDCOLUMNS` instead of `SUMMARIZE` for math?** | `SUMMARIZE` should only be used to group dimensions. Using it to execute calculations creates un-optimized DAX query plans. Always wrap `SUMMARIZE` inside `ADDCOLUMNS` to perform the math. |
| **What happens if you use `Table.Buffer` or `List.Buffer` in Power Query?** | It immediately stops [[Query Folding]]. All data up to that point is downloaded into local gateway memory before proceeding to the next step. |
| **What is the fundamental difference between a [[Calculated Column]] and a Measure?** | A Calculated Column is evaluated row-by-row during data refresh (using Row Context) and permanently consumes RAM/Storage. A Measure is evaluated on-the-fly when a visual renders (using Filter Context) and heavily consumes CPU. |
| **Why is the "[[Star Schema]]" superior to a highly Normalized (3NF) relational model in Power BI?** | VertiPaq thrives on wide, denormalized dimensions. Star Schemas require far fewer joins, drastically reducing CPU cycles, and they perfectly align with exactly how DAX [[Filter Propagation|filter propagation]] flows (1-to-many, Dimension-to-Fact). |
| **What are the two mandatory parameters required to configure [[Incremental Refresh]]?** | `RangeStart` and `RangeEnd`; both must be strictly typed as `DateTime`. |
| **What happens under the hood when a Composite Model visual queries both local and remote [[Tables|tables]]?** | The engine generates DAX, sends it over the network to the remote Hub, waits for the summary dataset payload, and then joins it locally in the formula engine. Joining on high-cardinality keys causes massive network latency. |
| **What does the `TREATAS` DAX function do?** | It creates a virtual relationship by taking the active filter context from one set of columns and magically applying it identically to an entirely disconnected set of columns. |
