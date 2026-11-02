##### REQ-3.4.1.13 Excel parser plugin (XLSX/XLS support)

## Purpose

The Excel plugin converts between spreadsheet workbooks and WAVE's table type. It reads a sheet - optionally a cell range within it - into a typed table for downstream workflow consumption, and writes a table into a sheet of a workbook. It is the file peer of the CSV converter: the same table semantics, the same typing and transform controls, addressed at a file rather than at a text value.

Unlike the text parsers, the Excel plugin owns its file access. A workbook is a binary, multi-sheet document with no faithful representation as a string value, and every producer of a workbook inside a run - a downloaded response, a transferred file, a file written by an earlier step - already hands over a file. The plugin therefore resolves a workbook path through the same artifact, test-data, and absolute- or relative-path resolution the file I/O plugin uses, including its read-only Test Data rule, and performs no other file operations.

## Capabilities

- Read one sheet of a workbook, chosen by name or position, into a typed table.
- Restrict the read to a cell range, with the header row addressed independently of that range.
- Type each column from the cells' own stored types, keep everything as text, or conform to a supplied table schema.
- Represent empty cells faithfully and choose how empty cells and type mismatches are handled.
- Optionally sort and filter the table, on the way in or on the way out.
- Report the workbook's sheet names, so a workflow can iterate every sheet.
- Write a table into a named sheet, creating the workbook or adding to an existing one.

## Operations

The plugin performs one of two operations per instance. **Read** takes a workbook path and produces a table (plus row count, resolved schema, and the workbook's sheet names). **Write** takes a table and produces a workbook file. The workbook, sheet and sort/filter controls are shared; the range, typing, empty-cell and mismatch controls apply to Read, while the placement and existing-file controls apply to Write.

## Formats

The plugin reads and writes the modern spreadsheet formats, macro-enabled workbooks included. The pre-2007 binary format is an unrelated file format internally; a workbook in that format is rejected during validation with a message naming the remedy, rather than failing obscurely inside the reader. Cell formatting, styling, charts, images, and macros are outside the plugin's scope: it converts data, and a macro-enabled workbook it writes to keeps whatever it already held.

## Sheet and Range Selection

A workbook holds many sheets, so a sheet is selected per instance - by name, or by position when names are not stable. Selecting nothing reads the first sheet. Reading every sheet is expressed as a workflow: the sheet-names output feeds a loop that runs one read per sheet.

A range restricts the read to a rectangle in A1 notation; leaving it unset reads the sheet's used area. Real spreadsheets rarely place their data at the top-left corner, so the header row is addressed independently of the range: a read of a single data row far down the sheet still names its columns from the header row above. With no header row, columns take their spreadsheet letters, matching what the user sees in the spreadsheet application.

## Column Typing

Spreadsheet cells carry their own types, so typing is a mapping rather than an inference. A column-typing mode governs the result:

- **Auto** (default): each column takes the type of the cells it holds - numbers, whole numbers, booleans and text mapping to the corresponding column types, and a column of mixed types widening to the narrowest type that fits all of them. Dates and times become ISO 8601 text, WAVE having no date type. Columns that are entirely empty resolve to text.
- **Text**: every column stays a string, each cell rendered as the spreadsheet displays it.
- **From schema**: a supplied table variable provides the column names and types, and the read data conforms to them exactly. Columns map by header name when a header row is present, otherwise by position.

A formula cell is read as its stored result. A workbook written by a tool that never computed its formulas carries no result for such a cell; that condition is reported in the errors output rather than silently returned as an empty cell, so a missing calculation is never mistaken for missing data.

## Empty Values and Type Mismatch

An empty cell becomes a null cell in every column by default, distinct from an empty string; an empty-value mode can instead substitute the column type's default. A null cell in a filter follows the check rule every block uses (REQ-3.1.3.25), with is null and not null among the operators.

A declared type (from-schema mode, or a filter's declared type) is enforced against the data; automatic typing is derived from the cells and never fails. When a cell will not fit a declared type, a mismatch mode chooses the reaction: fail the run, skip the offending row, or replace the cell with the empty-value default. Under a continue mode the offending cells are reported per row without aborting, and the operation reports overall failure while returning the rows that succeeded. This mirrors the CSV, JSON and XML parsers' fatal-versus-collected distinction.

## Sorting and Filtering

Either operation can optionally sort and filter the table it handles, using the same declarative Transform the CSV converter offers: an ordered list of sort keys applied in priority order, and per-column filter blocks whose conditions combine with the assertion control's and/or grouping. Both are backed by the table type's own filtering and ordering operations, and both are stored and run in the one shape the CSV converter uses (CSV-12). Cross-column or set-based logic beyond this declarative surface is expressed with the table's fluent operations in an expression.

## Writing

Write places a table into a named sheet, starting at a chosen cell so a table can be dropped into a prepared template beneath its titles, and optionally writes the column names as a header row above it. When the target file does not exist it is created. When it does exist, the default is non-destructive: the named sheet is added, or replaced if a sheet of that name is already there, and the rest of the workbook is left intact - so a run can assemble a multi-sheet report across several steps, or fill a sheet in a workbook that carries a cover page and branding. Replacing the whole file is an explicit choice.

Cell values are written in their native types, so numbers stay numeric and booleans stay boolean in the resulting workbook.

## Resource Guards

A workbook is a compressed archive whose expanded size is not evident from the file. The plugin bounds what it will open and what it will produce, and fails with a clear message when a workbook exceeds those bounds, so a malformed or hostile file cannot exhaust the backend. Reading streams the sheet and writing streams rows out, so a large sheet is never held twice in memory.

## Structured Return Data

Read returns:

| Output | Type | Description |
|--------|------|-------------|
| table | table | The rows and typed columns read from the sheet |
| row_count | integer | Number of rows in the returned table (after sort/filter) |
| schema | array | The resolved columns as name / type pairs |
| sheets | array | The workbook's sheet names, in workbook order |
| success | boolean | True when reading and every declared-type check succeeded |
| errors | array | Per-row problems (type mismatches under a continue mode, uncomputed formulas); empty on full success |

Write returns the path of the written workbook, a success flag, and any errors. Each output binds to a workflow variable through the standard output-binding surface.

## Error Handling Strategy

- **Fatal errors**: a missing file, a path outside the permitted areas, an unsupported format, a sheet or range that does not exist, or a workbook exceeding the resource guards yields no table and fails the operation.
- **Per-row errors**: a declared-type mismatch under a continue mode, or a formula cell with no stored result, is collected in the errors output; reading continues and success reflects whether every row succeeded.

Secret masking and I/O logging are inherited from the plugin execution harness; the plugin carries no credentials and declares no secret fields.

## Frontend Integration

Input parameters and outputs render through the standard element-configuration table (info tooltip, parameter, type, value, expression toggle), with no bespoke Excel UI. The parameter list is flat: how cells become typed data is the element's purpose, not an advanced detail. The Transform section is the CSV converter's sort and filter builder, rendered for any element whose schema exposes a sort parameter, so the shared configuration body carries no per-plugin identity check.

## Design System Reference

UI components follow the Design System. Design decisions: XLS-1 through XLS-15.
