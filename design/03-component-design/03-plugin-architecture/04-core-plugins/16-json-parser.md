##### REQ-3.4.1.16 JSON parser plugin (read, parse, query with JSONPath)

## Purpose

The JSON Parser plugin provides JSON document parsing and JSONPath query capabilities for workflow automation. It enables workflows to extract structured data from JSON API responses, configuration files, and other JSON sources using standard JSONPath expressions.

## Capabilities

- Parse JSON documents and validate well-formedness
- Execute multiple JSONPath queries against a single parsed document
- Return typed results with optional type coercion
- Collect per-query errors without stopping execution of other queries

## Query Execution Model

The plugin follows a parse-once, query-many approach:

1. The JSON document is parsed once at the start of execution
2. All queries execute against the parsed document structure
3. Each query runs independently - a failure in one does not stop others
4. Results are collected and returned according to the configured output mode

Parse failures are fatal since there is no document to query. Individual query failures are non-fatal and captured in the errors array.

## Output Modes

### Combined Mode (Default)

All query results are returned in a single `results` map. Each entry contains type metadata and the extracted value. This mode is suitable when downstream logic processes all results together.

### Individual Mode

Each query result is returned as a separate top-level output variable. Optional type coercion converts extracted values to the expected type (string, integer, float, boolean). This mode enables direct variable binding to query results.

## Type Coercion

When operating in individual mode, query results can be coerced to target types:

- **string**: Default, no conversion needed for scalar values. Arrays cannot be converted.
- **integer**: Converts numeric strings and floats to integers. Arrays cannot be converted.
- **float**: Converts numeric strings and integers to floats. Arrays cannot be converted.
- **boolean**: Converts string values (true/false, yes/no, 1/0) and numeric values to boolean. Arrays cannot be converted.

Coercion failures are captured as per-query errors.

## Error Handling Strategy

Two levels of error handling:

- **Fatal errors**: JSON parse failures halt execution immediately with a descriptive error. No queries run.
- **Per-query errors**: Invalid JSONPath expressions or type coercion failures are captured in the errors array. Other queries continue execution. The `success` flag reflects whether all queries succeeded.

## Frontend Integration

The plugin uses a dedicated query editor component in the configuration modal, following the same pattern as the XML Parser. The editor provides:

- Tabular query editing with alias and JSONPath expression per row
- Expression toggle per query for dynamic JSONPath expressions
- Client-side JSONPath testing using a browser-compatible evaluation library
- Result display dialog showing extracted values with type information
- JSONPath syntax validation with inline error feedback

## Design System Reference

UI components follow the Design System.
