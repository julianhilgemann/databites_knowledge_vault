# Advanced Power Query (M Language)

While the UI handles 90% of basic ETL, an Enterprise Power BI Developer must master advanced M scripting to handle complex APIs, distinct error handling, and query performance optimization.

## 1. Custom Functions (`() =>`)

When you find yourself repeating the same transformation steps across multiple queries (e.g., stripping special characters, invoking a REST API with pagination), you should abstract it into a Custom Function.

```pq
// fnCleanString
// This function removes all non-alphanumeric characters.
(InputText as text) as text =>
let
    // Convert text to list of characters
    TextList = Text.ToList(InputText),
    // Select only A-Z, a-z, and 0-9
    FilteredList = List.Select(TextList, each 
        (Character.ToNumber(_) >= 48 and Character.ToNumber(_) <= 57) or 
        (Character.ToNumber(_) >= 65 and Character.ToNumber(_) <= 90) or 
        (Character.ToNumber(_) >= 97 and Character.ToNumber(_) <= 122)
    ),
    // Recombine into text
    Result = Text.Combine(FilteredList)
in
    Result
```
*   **Pro Tip:** Always document functions with metadata so the `Invoke Function` UI in Power Query remains user-friendly for other developers.

## 2. Robust Error Handling (`try ... otherwise`)

Rows with errors prevent a semantic model from refreshing in the Service. Anticipating errors natively in M prevents silent failures.

```pq
let
    Source = Web.Contents("https://api.example.com/data"),
    // Safe extraction that defaults to null if the column is missing or corrupt
    SafeValue = try Source[DataColumn] otherwise null
in
    SafeValue
```

## 3. Controlling Evaluation (`Table.Buffer` & `List.Buffer`)

M evaluates lazily (streamed execution). When you reference a specific sub-query or list multiple times (for example, in a nested join or as a filter parameter), M might re-evaluate that query over and over again, destroying refresh performance.

*   `Table.Buffer()` loads a table completely into memory during the refresh before passing it to the next step.
*   `List.Buffer()` does the same for a single column/list.

**Warning:** Using `.Buffer` destroys [[Query Folding]] upstream. Only use it *after* the heavy database operations have folded, usually right before complex application-level filtering or looping.

## 4. Record and List Operations
Instead of expanding JSON rows into a massive Cartesian product, operate on the Lists natively.
*   `Table.AddColumn(Source, "Sum", each List.Sum([ListColumn]))`

### Summary Checklist
- [ ] Are repetitive transformations abstracted into Custom Functions?
- [ ] Have known volatile columns been wrapped in `try ... otherwise`?
- [ ] Are small lookup [[Tables|tables]] fully `Table.Buffer`ed before being joined repeatedly?
- [ ] Has [[Query Folding]] been maximized before applying native M functions?
