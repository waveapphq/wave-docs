### 3.6 Variable System

The variable system provides hierarchical scoping, type-safe storage, and expression evaluation for workflow data. Variables flow through the execution hierarchy with clear visibility rules and support both primitive and complex data types.

#### Supported Variable Types

- **Scalar Types**: string, integer, float, boolean
- **Collection Types**: array (ordered items), map/hash (key-value pairs); each item and value carries its own declared element type
- **Structured Types**: table (two-dimensional with named, typed columns)
- **Null**: every type also holds null, a value distinct from empty and zero that keeps the declared type (see [Null Values](06-return-values.md#null-values))

#### Section Index

- [3.6.1 Hierarchical Scoping](01-hierarchical-scoping.md) - Scope levels and visibility rules
- [3.6.2 Variable Inheritance](02-variable-inheritance.md) - Self-contained test variables with namespace access
- [3.6.3 Runtime Passing](03-runtime-passing.md) - Data flow between blocks
- [3.6.4 Environment Integration](04-environment-integration.md) - Environment variable access
- [3.6.5 Secret Masking](05-secret-masking.md) - Sensitive value protection
- [3.6.6 Return Values](06-return-values.md) - Plugin output and scope export
- [3.6.7 Computed Variables](07-computed-variables.md) - Expression evaluation and validation
- [3.6.8 Table Type](08-table-type.md) - Two-dimensional structured data with typed columns
- [3.6.9 Numeric Precision](09-numeric-precision.md) - Exact integer and float values without IEEE-754 rounding
- [3.6.10 Declared Element Types](10-element-types.md) - The declared type of each array item and map value, kept by every surface
