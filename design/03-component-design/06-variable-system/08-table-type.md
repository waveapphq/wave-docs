### 3.6.8 Table Type

#### REQ-3.5.5.1 Table Structure and Schema

##### REQ-3.5.5.1 Data table support

The table type provides two-dimensional structured data storage with named columns and typed cells, enabling spreadsheet-like data structures for test data, query results, and tabular outputs. Tables enforce schema consistency across all rows ensuring every row conforms to the declared column structure, unlike hashes which provide arbitrary key-value mapping.

**What counts as a table**: A value of a table-typed variable, input, output or setting is a table or a list whose every item is a row (a map of column name to cell). A list holding anything other than rows, such as `[1, 2]`, is not a table and is refused with the same type message as any other mismatch, whether it was typed, computed by an expression or returned by a Function; a list of rows is realized as a table before it is stored or handed on.

**Column Schema Definition**: Tables consist of column definitions specifying column names and data types. Each column has a unique name within the table (duplicate column names are rejected) and an explicit type from the supported set: string, integer, float, boolean. Column types enforce type safety for cell values, with None allowed for any column type to represent missing data. Column ordering is significant and preserved throughout the table's lifecycle, enabling both positional (index-based) and named access patterns.

**Column Type Semantics**: The four supported column types have strict validation rules:
- **String**: Textual data including empty strings, UTF-8 encoded
- **Integer**: Whole numbers without fractional components (Python `int`, excludes `bool`)
- **Float**: Numbers with decimal precision (accepts both `int` and `float` values, excludes `bool`)
- **Boolean**: Logical true/false values (Python `bool` only, not numeric interpretations)

Type validation occurs on every cell assignment, preventing type mismatches from entering the table structure. Values that do not match the column type raise TypeError with a descriptive message identifying the column name, expected type, and actual type received.

**Default Column Naming**: When columns are added without explicit names, the system generates default names following the pattern "Column N" where N is the zero-based column index at insertion time. This ensures every column has a name for associative access while supporting programmatic table construction without requiring manual naming.

**Empty Tables and Schema Preservation**: Tables support zero-row states while preserving their complete column schema. An empty table with defined columns maintains column names and types, enabling proper initialization of table variables and type-safe empty state handling. This distinguishes "query returned zero rows" (empty table with schema) from "no table" (null value).

#### REQ-3.5.5.2 Row and Column Operations

The table type provides mutable operations for adding, inserting, and removing both rows and columns, enabling dynamic table construction and modification during workflow execution.

**Row Operations**:
- **add_row(values)**: Appends a row to the table. If the values list is shorter than the column count, remaining columns are filled with None. If values is None, all columns receive None.
- **insert_row(values, row_idx)**: Inserts a row at the specified index. Supports negative indexing where -1 inserts before the last row, -2 before the second-to-last, etc.
- **remove_row(row_idx)**: Removes the row at the specified index. Supports negative indexing.
- **set_row(values, row_idx)**: Updates multiple column values in an existing row starting from column 0. Extra values beyond column count are ignored; missing values leave columns unchanged.
- **clear()**: Removes all rows while preserving the column schema.

**Column Operations**:
- **add_col(values, name, col_type)**: Appends a column to the table. Default type is string. If values list is shorter than row count, remaining rows receive None.
- **insert_col(values, col_idx, name, col_type)**: Inserts a column at the specified index with negative indexing support.
- **remove_col(col)**: Removes a column by name (string) or index (integer). Supports negative indexing for index-based removal.

**Negative Indexing**: All row and column operations support Python-style negative indexing. Index -1 refers to the last element, -2 to the second-to-last, and so forth. This applies to both read operations (cell access, row retrieval) and write operations (insert, remove, set).

**Type Validation on Mutation**: All mutation operations validate values against column types before modifying the table. Invalid values are rejected with TypeError before any changes occur, ensuring tables always maintain type consistency.

#### REQ-3.5.5.3 Cell Access Patterns

Tables support multiple access patterns for reading and writing cell values, accommodating both positional and named access requirements.

**Row Access**: Accessing a table by row index returns a TableRowView, a read-only view of the row that supports further indexing by column name or column index:
- `table[row_idx]` returns TableRowView
- `table[row_idx][col_name]` returns cell value by column name
- `table[row_idx][col_idx]` returns cell value by column index

**Read-Only Row Views**: TableRowView prevents direct assignment through indexing syntax. Attempting `table[row][col] = value` raises TypeError with guidance to use `set_cell()` instead. This design prevents accidental mutations through chained indexing while maintaining intuitive read access patterns.

**Direct Cell Access Methods**:
- **cell(row_idx, col)**: Returns the cell value at the specified row and column (name or index)
- **set_cell(value, row_idx, col)**: Sets a single cell value with type validation
- **row(row_idx)**: Alias for `table[row_idx]`, returns TableRowView

**Column Extraction**:
- **col(col)**: Returns the entire column as a list of values. Accepts column name (string) or column index (integer). Enables column-wise operations using Python built-ins: `sum(table.col('amount'))`, `max(table.col('score'))`.

**Iteration**: Tables support iteration over rows, yielding TableRowView for each row in order. This enables for-loop patterns: `for row in table: process(row['name'])`.

**Length and Metadata**:
- `len(table)` returns row count
- `row_count()` returns row count
- `col_count()` returns column count
- `cell_count()` returns total cells (rows * columns)
- `col_names()` returns list of column names
- `col_types()` returns list of column types as strings

**Boolean Evaluation**: Tables are truthy if they contain at least one row, falsy if empty. This enables conditional patterns: `if table: process(table)`.

#### REQ-3.5.5.4 Serialization and Deserialization

##### REQ-3.5.5.4 Data-driven test iteration

Tables support bidirectional serialization for storage, transmission, and interoperability with external data sources.

**Dictionary Serialization** (`to_dict()` / `from_dict()`):
The canonical serialization format represents tables as dictionaries with explicit schema:
```
{
  "columns": [
    {"name": "id", "type": "integer"},
    {"name": "name", "type": "string"},
    {"name": "active", "type": "boolean"}
  ],
  "rows": [
    [1, "Alice", true],
    [2, "Bob", false]
  ]
}
```
This format preserves complete type information and column ordering, enabling lossless round-trip serialization. The `from_dict()` class method reconstructs tables from this format.

**List-of-Dictionaries Serialization** (`to_list()` / `from_list()`):
For interoperability with common data formats (JSON APIs, pandas-style data), tables convert to/from lists of row dictionaries:
```
[
  {"id": 1, "name": "Alice", "active": true},
  {"id": 2, "name": "Bob", "active": false}
]
```
The `to_list()` method produces this format for output. The `from_list()` class method creates tables from this format with automatic type inference:
- Boolean values in first row → boolean column
- Integer values in first row → integer column
- Float values in first row → float column
- All other values → string column

Column order in `from_list()` matches key order from the first row dictionary.

**Type Inference Behavior**: When creating tables from list-of-dictionaries format, column types are inferred from the first row's values. Subsequent rows must contain values compatible with the inferred types. None values are allowed in any column regardless of inferred type. This inference enables convenient table construction from untyped data sources while maintaining type safety after construction.

#### Row Filtering and Ordering

Beyond the in-place mutation operations, the table type provides two non-mutating query operations that return a NEW table, leaving the source unchanged. Because each returns a table they chain fluently, and because one implementation backs them, a plugin's declarative sort/filter and a hand-written expression share identical semantics.

**Filtering**: filtering is two steps. `filter(column)` names a column, by name or zero-based index, and returns the comparison step; that step's operator selects the rows and returns a new table. The operator vocabulary is the type-appropriate set shared with the assertion control (equality, ordering, string containment / prefix / suffix, regex, emptiness, null), so a comparison means the same thing whether authored as a filter condition or an assertion, and an operator that does not apply to the column's type is refused by name. A null cell follows the check rule (REQ-3.1.3.25): equality, ordering, containment, prefix, suffix and regex operators do not keep its row, not equals and does not contain do, is empty keeps it and is not empty drops it, and is null and not null find or exclude it. Filters chain to express conjunction: each successive filter narrows the previous result. Naming the column before the comparison is what lets the expression editor offer only the operators that column's type accepts.

**Ordering**: `order_by(keys)` returns a new table sorted by one or more column keys in priority order, the first key most significant, each carrying an ascending or descending direction. Sorting is stable, so equal keys preserve source order; missing cells sort last regardless of direction. Multiple keys in a single call express a composite ordering (primary, then tiebreakers).

**Non-mutating contract**: Both operations deep-copy the rows they carry forward, so the returned table shares no row storage with the source - mutating one never affects the other. This is the property that makes fluent chaining safe, and it distinguishes these query operations from the in-place row / column operations (REQ-3.5.5.2).

**Expression availability**: Both are ordinary table methods, callable inside expressions with no additional registration (see below), and the expression editor offers them and the comparison step's operators as it does any other method. The ordering operation is named to avoid the mutation-guard vocabulary the expression validator applies to field expressions, so a chained filter-then-order expression is accepted on every expression surface.

#### Human-Readable Rendering

A table printed as text - in a log message, in the run console, in the persisted log, in a report - renders as aligned columns rather than a run of separated values. A title line states the table's true shape (its full row and column counts, whatever the rendering shows). Below it, one line of column names, one line of column types named in full, and one rule of dashes as wide as each column. Numeric columns (integer, float) are right-aligned; string and boolean columns are left-aligned. Every column is as wide as the widest thing in it - its name, its type name, or one of the shown cells - and the last column is never padded, so no line carries trailing whitespace.

**Bounded, and never silently**: four bounds keep one block's output from flooding a console, and each one states what it left out. The rendering shows the first ten rows and then says how many more there are; it shows the first twelve columns and then says how many more there are; a single column is never wider than forty characters, and a cell over that width ends in an ellipsis; and a cell whose text runs past two hundred characters is replaced by a plain statement of its size, because past that length a fragment misinforms more than it informs. The whole table is always available unshortened in the variable inspection surface.

**Masked before measured**: a cell is masked while it is being rendered, against the run's secret registry for the block doing the rendering, and its width is measured from the masked text. A protected value therefore contributes the width of the placeholder and nothing else, so no column width and no alignment can disclose how long the real value was. Because masking happens first, every bound above cuts text that is already masked and can never strand a fragment that a later masking step would no longer recognize. A rendered table shows its at-rest form: a block's own declared per-field reveal still shows that block's cells, while a temporary system-wide reveal does not reach inside a rendered table.

#### Integration with Expression Evaluation

Tables integrate with the computed variable and expression evaluation system, enabling tables to be used in expressions and enabling expression results to include table data.

**Table in Safe Builtins**: The Table class, TableRowView class, and ColumnType enum are included in the safe builtins whitelist for expression evaluation. This enables expressions to construct tables, access table data, and perform table operations within the sandboxed evaluation context.

**Column Operations in Expressions**: Table columns extracted via `col()` return standard Python lists, enabling full use of Python built-in functions:
- `sum(table.col('quantity'))` - sum column values
- `max(table.col('score'))` - find maximum
- `min(table.col('timestamp'))` - find minimum
- `len(table.col('items'))` - count non-null values
- `sorted(table.col('name'))` - sort column values
- `[row['name'] for row in table if row['active']]` - filtered extraction

**WebSocket Push Optimization**: When tables are watched variables, WebSocket updates send summary payloads rather than full table data to prevent flooding: `{rows: M, cols: N, changed: true}`. Full table data is fetched on-demand when the user expands the variable view.
