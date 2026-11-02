##### REQ-3.4.1.12 CSV parser plugin (read and parse to table)

## Purpose

The CSV plugin converts between delimited text and WAVE's table type. It parses CSV/TSV text - from a prior plugin's output, an HTTP response body, or a variable - into a typed table for downstream workflow consumption, and serializes a table back into CSV text. It is the table peer of the JSON and XML parsers: a pure format converter that reads and writes variables, never files (file access is the File I/O plugin's responsibility) and never persists anything of its own.

## Capabilities

- Parse delimited text into a WAVE table with a configurable delimiter and header handling.
- Assign column names from a header row, from an explicit name list, or positionally.
- Type each column by inference, keep everything as text, or conform to a supplied table schema.
- Represent missing fields faithfully and choose how empty fields and type mismatches are handled.
- Optionally sort and filter the table by one or more columns, on the way in or on the way out.
- Serialize a table back to CSV text with the same delimiter and header controls.

## Operations

The plugin performs one of two operations per instance. **Parse** takes CSV text and produces a table (plus row count and resolved schema). **Serialize** takes a table and produces CSV text. The two operations share the delimiter, header and sort/filter controls; the typing, empty-value and mismatch controls apply to Parse.

## Column Typing

A column-typing mode governs the parsed table's column types, because CSV is untyped on the wire:

- **Auto** (default): each column is typed from its data in a single pass, taking the narrowest type that fits and widening as needed - integer to float to string, with boolean treated as its own type that falls back to string on any non-boolean value. Columns that are entirely missing resolve to text. Guards keep ambiguous values as text rather than silently losing fidelity: leading-zero (and leading-sign) numeric strings stay text, booleans are recognized only from explicit true/false words, and non-finite floats are not auto-detected.
- **Text**: every column stays a string. Predictable and lossless.
- **From schema**: a supplied table variable provides the column names and types, and the parsed data conforms to them exactly. Columns map by header name when a header row is present, otherwise by position. This is the complete answer to auto-typing ambiguity, and the same variable may receive the parsed result.

Column typing is best-effort and validation is the user's responsibility, consistent with the table type contract - except where a type is explicitly declared (see Type Mismatch), which is enforced.

## Empty Values and Type Mismatch

A missing field becomes a null cell in every column by default, distinct from an empty string; standard CSV cannot distinguish a missing field from a quoted empty one, so missing is the honest default. An empty-value mode can instead substitute the column type's default. A null cell in a filter follows the check rule every block uses (REQ-3.1.3.25), with is null and not null among the operators.

A declared type (from-schema mode, or a filter's declared type) is enforced against the data; auto-typing is inferred and never fails. When a cell will not fit a declared type, a mismatch mode chooses the reaction: fail the run, skip the offending row, or replace the cell with the empty-value default. Under a continue mode the offending cells are reported per row without aborting, and the operation reports overall failure while returning the rows that succeeded.

## Sorting and Filtering

Either operation can optionally sort and filter the table it handles: Parse shapes the table it returns, Serialize shapes the table it writes out, leaving the source variable untouched. Sorting is an ordered list of column keys, each with a direction, applied in priority order. Filtering is expressed per column: each filter targets one column (by name or zero-based index) with a declared type, and holds one or more conditions on that column's value - either a comparison (operator plus value) or a per-row expression over the column's value. Conditions within a column combine with the same and/or grouping the assertion control offers; separate columns combine conjunctively. A filter or sort built in the configuration sheet is stored in one shape - a filter condition is the same condition item the check blocks use - and runs exactly as built (CSV-12); the block catalog describes that shape to an assistant with its fields typed, so a recipe writes a filter the way the sheet builds it. Both sorting and filtering are backed by the table type's own filtering and ordering operations, so the same semantics are available to expressions. Cross-column or set-based logic beyond this declarative surface is expressed with the table's fluent operations in an expression.

## Input Sources

Parse accepts its CSV text as a string value - a literal, a variable reference, or an expression - exactly as the JSON and XML parsers accept their content. Serialize accepts its table as a table-typed variable reference. The plugin performs no file access; a workflow that needs to read or write a file pairs the CSV plugin with the File I/O plugin.

## Structured Return Data

Parse returns:

| Output | Type | Description |
|--------|------|-------------|
| table | table | The parsed rows and typed columns |
| row_count | integer | Number of rows in the returned table (after sort/filter) |
| schema | array | The resolved columns as name / type pairs |
| success | boolean | True when parsing and every declared-type check succeeded |
| errors | array | Per-row problems (type mismatches under a continue mode); empty on full success |

Serialize returns the CSV text, a success flag, and any errors. Each output binds to a workflow variable through the standard output-binding surface.

## Error Handling Strategy

- **Fatal errors**: input that cannot be read as delimited text at all yields no table and fails the operation.
- **Per-row errors**: a declared-type mismatch under a continue mode is collected in the errors output; parsing continues, success reflects whether every row succeeded. This mirrors the JSON and XML parsers' fatal-versus-collected distinction.

Secret masking and I/O logging are inherited from the plugin execution harness; the plugin carries no credentials and declares no secret fields.

## Frontend Integration

Input parameters and outputs render through the standard element-configuration table (info tooltip, parameter, type, value, expression toggle). The sort and filter controls render as a Transform section of two collapsible sub-sections built on the assertion control's condition-building and grouping primitives. Type-driven behavior is visible in the form: a column reference is chosen by name or by index, becoming a searchable picker only when a schema supplies the column set, and a filter's declared type is fixed automatically under schema mode.

## Design System Reference

UI components follow the Design System. Design decisions: CSV-1 through CSV-11.
