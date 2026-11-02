##### REQ-3.4.1.11 XML processing with XPath support

The XML processing plugin provides comprehensive XML document parsing and querying capabilities through XPath expressions, enabling workflows to extract data from XML responses and validate XML structure.

**Plugin Purpose and Use Cases**: The XML plugin enables workflows to interact with XML data in structured ways. Primary use cases include parsing XML responses from SOAP APIs and REST services, extracting specific values from XML documents using XPath queries, validating XML document structure and content against expected patterns, transforming XML data into WAVE-compatible data types for workflow consumption, and querying complex nested XML hierarchies without manual string parsing. This plugin eliminates the need for brittle string manipulation when working with XML data.

**XML Parsing and Validation**: The plugin accepts XML content as string input and parses it into an internal representation for querying. The parser validates XML well-formedness during parsing, rejecting malformed XML with detailed error messages indicating parse failures, line numbers, and syntax issues. Namespace-aware parsing supports XML documents with namespace prefixes and declarations. XML parse errors are fatal - without a valid document, no queries can execute. The parse error is reported via the standard execution error mechanism.

**XPath Query Engine**: The plugin implements XPath expression evaluation enabling precise data extraction from parsed XML documents. Workflows provide one or more XPath expressions as a map parameter (query name to XPath expression). The plugin evaluates each XPath expression against the parsed XML document. XPath query results are automatically converted to WAVE-compatible types: text nodes become strings, numeric XPath results become integers or floats, boolean XPath results become booleans, and node-sets become arrays of strings (text content). This automatic type mapping enables natural consumption of XML data in subsequent workflow steps.

**Multiple Query Support**: A single plugin instance can execute multiple XPath queries against the same XML document, enabling efficient extraction of multiple values without re-parsing. Each query is configured with an alias (query name) and an XPath expression. All queries execute against the same parsed document. This multi-query capability supports workflows that need to extract several related values from complex XML responses such as extracting multiple fields from a SOAP response or gathering data from different sections of an XML configuration file.

**Output Modes**: The plugin supports two output modes that control how query results are structured:

- **Combined mode** (default): All query results are collected into a single `results` map. Each entry is keyed by the query alias and contains a map with `type` (the WAVE type) and `value` (the extracted data). Downstream steps access results via `results["alias"]["value"]`.

- **Individual mode**: Each query result is stored as a separate output variable using the query alias as the variable name. This mode supports optional type coercion - each query can declare an expected output type (string, integer, float, boolean) and the plugin converts the raw XPath result accordingly. Failed coercions produce an entry in the errors array without stopping other queries.

**Structured Return Data**: The plugin returns three standard output variables:

| Output | Type | Description |
|--------|------|-------------|
| success | boolean | True when all queries executed without errors |
| errors | array | Per-query error details as `[{query, error}]` - empty when all queries succeed |
| results | map | Query results map with `{alias: {type, value}}` entries (combined mode only) |

In individual mode, query results are stored directly as named output variables (one per query) rather than in the results map.

**Per-Query Error Collection**: XPath evaluation errors are handled per-query rather than failing the entire plugin. When a query produces an XPath syntax error, namespace resolution error, or other evaluation failure, that error is recorded in the `errors` array with the query name and error message. The remaining queries continue executing. The `success` flag is set to false when any query produces an error. This resilient approach ensures that one bad query does not prevent other valid queries from returning their results.

**Namespace Handling**: XML documents frequently use namespaces for element disambiguation. The plugin auto-extracts namespace declarations from the root element (e.g., `xmlns:ex="http://example.com"` produces the mapping `{ex: "http://example.com"}`). These auto-extracted namespaces are used to resolve prefixed names in XPath expressions without requiring any user configuration. For example, given `<root xmlns:inv="http://example.com/invoice"><inv:total>99.50</inv:total></root>`, the query `//inv:total/text()` resolves correctly because the `inv` prefix is auto-extracted from the root element. A query names a namespace by the prefix the document declares on its root element.

**XML Document Sources**: The plugin accepts XML content from multiple sources to support different workflow patterns. Direct string input enables processing of XML from previous plugin return values, HTTP response bodies, or file I/O operations. The input parameter accepts WAVE string variables containing XML text. Template mode allows constructing XML content with embedded expressions. This flexible input model integrates naturally with other plugins - HTTP plugins return response bodies as strings, file I/O plugins read files as strings, and variable references pass XML content between workflow steps.

**Error Handling and Reporting**: The plugin distinguishes between fatal and non-fatal errors. XML parse errors are fatal - without a parsed document, no queries can execute. Per-query XPath errors (invalid syntax, unresolved namespaces, evaluation failures) are non-fatal and collected in the errors array. Type coercion failures in individual mode are also non-fatal. This structured error reporting enables workflows to distinguish between document-level failures (requiring error handling blocks) and query-level issues (which can be inspected via the errors array).

**Performance Considerations**: The plugin parses XML documents once and evaluates all configured XPath queries against the same parsed DOM, avoiding redundant parsing overhead. For workflows that need to query the same XML document multiple times across different blocks, consider consolidating queries into a single plugin instance. XPath evaluation performance depends on query complexity and document size - simple path expressions execute quickly while complex predicates or recursive queries may have higher overhead.
