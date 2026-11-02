### 3.6.7 Computed Variables

#### REQ-3.1.3.13 Computed variables with expressions and runtime evaluation

##### REQ-3.1.3.13 Computed variables with expressions and runtime evaluation

The computed variable system provides dynamic value composition capabilities enabling workflows to define variables whose values are derived from other variables through expression evaluation at runtime. This design establishes a declarative expression language for variable interpolation and computation while maintaining type safety, scope integration, and efficient evaluation semantics that align with the hierarchical variable scoping model.

**Core Concept and Purpose**: Computed variables represent values that are not directly assigned but instead calculated from expressions that reference other variables. Rather than requiring explicit assignment operations or intermediate plugin executions to compose strings, format URLs, compute derived values, or construct complex data from constituent parts, workflows declare computed variables with expressions that describe how the final value should be constructed. The variable system evaluates these expressions at runtime when the computed variable's value is accessed, resolving variable references from the current scope context and producing the final value. This declarative approach eliminates boilerplate variable manipulation logic, enables clear expression of derived data relationships, and maintains a clean separation between data definition and data transformation.

**Expression Syntax**: Computed variable expressions are pure Python expressions evaluated at runtime. Variables from the current scope are available directly by name in the expression - no special delimiters or interpolation syntax is needed. When the expression evaluates, all variables from the active scope context are injected into the evaluation namespace, and the expression is executed via `eval()` with safe builtins. For string composition, Python f-strings provide natural interpolation: `f"http://{host}:{port}/api"`. For arithmetic, standard Python operators work directly: `base_timeout * retry_count`. This approach uses standard Python syntax familiar to developers and test engineers, eliminating custom interpolation syntax in favor of the language's native capabilities.

**String Composition**: The foundational use case for computed variables involves string composition where multiple string variables or string literals combine into a composite string value. Using Python f-strings, a base URL variable and an endpoint path variable compose into a full URL: `f"{base_url}/{endpoint}"`. A protocol, hostname, port, and path combine into a connection string: `f"{protocol}://{host}:{port}/{path}"`. A prefix, identifier, and suffix merge into a formatted key: `f"{prefix}_{identifier}_{suffix}"`. These string composition patterns eliminate the need for dedicated string concatenation plugins or bash scripts, enabling clean declarative expression of derived string values directly in variable definitions.

**Numeric Expression Evaluation**: Beyond simple string interpolation, computed variables support arithmetic expressions enabling numeric calculations directly within variable definitions. Expressions incorporate standard arithmetic operators for addition, subtraction, multiplication, division, and modulo operations on numeric variables. Numeric expressions resolve variable references to their numeric values, apply operators according to standard mathematical precedence rules, and produce numeric results. This numeric computation capability enables workflows to derive calculated values such as timeouts computed from base values and multipliers, offsets calculated from reference points and deltas, or totals accumulated from component values, all without requiring dedicated calculation plugins or scripting.

**Type-Aware Expression Evaluation**: The expression evaluation engine operates with awareness of variable types, preserving type information through interpolation and computation operations. When expressions reference typed variables, each value keeps its type, enabling appropriate handling based on types. String interpolation of numeric variables converts numbers to string representation automatically. Boolean variables referenced in string interpolation convert to literal true/false string representations. This type-aware behavior prevents ambiguity and type errors while maintaining intuitive semantics aligned with common programming language expectations.

**Scope Resolution for Referenced Variables**: Computed variables exist only at test scope. Their expressions reference test variables by name and suite, regression and system variables through namespace objects (REQ-3.1.3.2), and each reference resolves at runtime when the expression evaluates.

**Evaluation Timing and Lazy Evaluation**: Computed variable expressions are evaluated lazily at the point when the variable's value is accessed rather than eagerly when the variable is defined or when referenced variables change. This lazy evaluation strategy provides several benefits: expressions only evaluate when actually needed, avoiding computation overhead for unused variables; an expression sees the values of referenced variables as they are at first access, including updates made before then; evaluation happens in the runtime scope context ensuring proper variable resolution from the active scope hierarchy. Lazy evaluation means that a computed variable defined early in a workflow but accessed later will use the values of referenced variables as they exist at access time, not definition time. If referenced variables are modified between definition and access, the computed variable's evaluation incorporates those modifications automatically. A computed variable is evaluated once, at first access, and the result is kept.

**Circular Dependency Detection and Prevention**: The computed variable system must prevent circular dependency scenarios where computed variable A's expression references computed variable B, and computed variable B's expression references computed variable A, either directly or through a chain of intermediate computed variables. Such circular dependencies would cause infinite evaluation loops where each variable's evaluation triggers the other's evaluation indefinitely. The system detects circular dependencies during workflow validation through static dependency graph analysis: the validation engine parses all computed variable expressions, identifies all variable references in each expression, constructs a directed graph where edges represent dependency relationships between computed variables, and applies cycle detection algorithms to identify any circular paths in the graph. When circular dependencies are detected, workflow validation fails immediately with a clear error message identifying the variables involved in the dependency cycle and the reference chain forming the circle. The same cycle detection runs at every other entry point where computed variable definitions are introduced or modified, including the variable management API endpoints that set or update variables on a scope. The storage layer therefore cannot reach a state with circular dependencies regardless of which path was used to introduce the definition, ensuring runtime evaluation can proceed without cycle checking overhead or risk of infinite loops.

**Error Handling for Undefined Variable References**: The system detects undefined variable references during workflow validation through static analysis of variable scopes and computed variable expressions. The validation engine examines each computed variable's expression to identify all variable references, and verifies that every referenced variable exists among the test's variables or is a namespace reference. When validation detects references to undefined variables, workflow validation fails immediately with a clear error message identifying the computed variable name, the undefined variable reference name, the expression containing the reference, and the scope context where the reference occurs. This static validation approach prevents misconfigured workflows from reaching execution, catching common errors such as misspelled variable names, references to variables not yet defined, or incorrect scope assumptions during workflow design rather than during test execution.

**Error Handling for Type Incompatibility**: When a computed variable's expression attempts operations that are incompatible with the types of referenced variables, the evaluation engine raises type compatibility errors with diagnostic details. Arithmetic operations on variables containing non-numeric values trigger clear errors identifying which variable contains incompatible data and what operation was attempted. Division by zero in arithmetic expressions triggers explicit division-by-zero errors. String interpolation succeeds for all variable types through automatic type-to-string conversion, so type errors arise primarily in arithmetic and logical operations. Error messages include the computed variable name, the expression being evaluated, the specific operation that failed, the variable references involved, and the actual types encountered, providing sufficient context for debugging and correction.

**Nested Computed Variable References**: Computed variables can reference other computed variables in their expressions, enabling layered composition where base computed variables provide intermediate values that higher-level computed variables build upon. When a computed variable A's expression references computed variable B, evaluating A triggers evaluation of B, which in turn produces its value based on current variable values in scope. This transitive evaluation propagates through multiple levels of computed variable dependencies, enabling complex derived value chains without requiring explicit ordering or manual evaluation sequencing. The evaluation engine handles nested evaluation transparently, ensuring each computed variable in the dependency chain evaluates based on current values of all transitively referenced variables.

**Operator Precedence and Expression Complexity**: The expression language supports standard operator precedence for arithmetic operations ensuring mathematical expressions evaluate correctly without requiring excessive parentheses. Multiplication and division operators have higher precedence than addition and subtraction, so expressions combine terms before adding them as mathematically expected. Operators of equal precedence evaluate left-to-right. Parentheses override default precedence enabling explicit grouping for complex calculations. While the expression language supports these standard mathematical semantics, the design intentionally limits expression complexity to maintain clarity and debuggability. Expressions spanning multiple lines, deeply nested parentheses, or complex logical operations are discouraged in favor of intermediate computed variables that break complex calculations into clear sequential steps. This design philosophy balances expressiveness with maintainability, ensuring expressions remain readable and understandable for test engineers.

**String Escaping**: Since computed variable expressions are pure Python, standard Python string escaping rules apply. Curly braces in f-strings are escaped by doubling: `f"value is {{not_a_var}}"` produces the literal string `value is {not_a_var}`. Backslashes, quotes, and other special characters follow standard Python string literal escaping conventions. No custom escape mechanism is needed.

**Integration with Secret Masking**: Computed variables integrate with the secret masking system following the same responsibility model established for all derived values: users must explicitly mark computed variables as secret if they derive values from secret variables. When a computed variable's expression references secret variables, evaluation proceeds normally regardless of whether the computed variable itself is marked as secret. If the computed variable is marked as secret, the evaluated result receives secret masking in all output contexts (logs, reports, artifacts, WebSocket messages) just like any other secret variable. If the computed variable is not marked as secret, the evaluated result appears unmasked in outputs, placing responsibility on test engineers to properly designate computed variables that derive from secrets. This approach maintains consistency with the broader secret masking design principle that secret designation does not automatically propagate through variable transformations or operations, and users control which derived values require protection.

**Integration with Variable Assignment**: Computed variables store expressions rather than direct values. When assignment operations target computed variables (unusual but permitted), the assignment replaces the expression with a direct value and converts the variable from computed type to the assigned value's type. This conversion enables workflows to initially define variables as computed but override them with direct assignments when specific values are needed, providing flexibility for default-with-override patterns.

**Validation and Static Analysis**: The computed variable system performs static validation during workflow validation phase before any execution begins, catching common errors early. Expression syntax validation ensures expressions are well-formed with balanced braces, valid operator usage, and correct escape sequences. Circular dependency detection runs during workflow validation by constructing the dependency graph from all computed variable expressions and checking for cycles. Variable reference validation confirms that all referenced variables are defined in accessible scopes at the points where computed variables will be evaluated, preventing runtime undefined variable errors for statically analyzable workflows. Type compatibility validation checks that arithmetic operations reference numeric variables, preventing type errors in expressions. These static validation steps provide early error detection and clear diagnostic feedback during workflow design and development, shifting error detection left and improving workflow quality.

**Debugging and Introspection Support**: The variable watching UI and debugging interfaces provide special support for computed variables, giving developers access to both the source expression and the evaluated result. A computed variable carries a clear expression indicator wherever it appears, and its per-variable inspect surface offers the source and the evaluated runtime value as explicit, switchable views: the source view shows the original expression text, while the evaluated views render the realized value exactly like a regular variable of the declared type. A computed variable that has not yet been referenced - and therefore has not evaluated - presents an explicit not-evaluated state rather than a stand-in value. During step-through debugging, when execution pauses and variable state is inspected, computed variables display with the same indicators, distinguishing them from regular assigned variables.

**Boundary Conditions and Edge Cases**: The computed variable system handles various boundary conditions with well-defined semantics. Computed variables with expressions that reference only literals and no variables evaluate to constant values, essentially functioning as constant definitions. Computed variables with empty expressions are invalid and rejected during validation. Computed variables that reference arrays or hashes perform string interpolation by converting complex structures to string representations, though this pattern is discouraged as the string representation may not be semantically meaningful. Computed variables in parallel execution branches resolve variables from the test's live variables, which branches share (REQ-3.2.3.3). These boundary cases receive explicit handling ensuring predictable behavior across all workflow patterns.

#### Expression Validator Architecture

The expression validator provides comprehensive static analysis of expressions before runtime evaluation, using Python's Abstract Syntax Tree (AST) parsing for accurate syntax validation and scope analysis.

**AST-Based Parsing**: Rather than using simple string matching or regular expressions for validation, the expression validator leverages Python's built-in `ast.parse()` function to parse expressions into Abstract Syntax Trees. This approach provides accurate syntax validation that matches Python's actual parsing behavior, correct identification of variable references versus string literals, proper handling of complex expressions including nested calls and operators, and accurate position tracking for error reporting.

**Expression Validation**: The validator validates pure Python expressions. Expressions are parsed as Python `eval`-mode AST nodes, enabling full scope analysis, type inference, and security checks. The validator receives the raw expression string and the set of available variable names, performing comprehensive analysis in a single pass.

**Validation Result Structure**: Validation produces a structured result containing:
- `valid`: Boolean indicating whether validation passed (no errors)
- `issues`: List of issue objects, each containing:
  - `type`: "error" (fails validation) or "warning" (informational only)
  - `code`: Issue classification (syntax_error, undefined_variable, blocked_pattern, statement_not_allowed)
  - `message`: Human-readable description
  - `line`, `column`: Start position within the expression
  - `end_line`, `end_column`: End position for range highlighting

**Comprehension Scope Handling**: Python comprehensions (list, dict, set, generator) create local scope for their loop variables. The validator correctly identifies that variables defined as loop targets are local to the comprehension and should not trigger undefined variable errors:

```
[item.name for item in items]  # 'item' is local, 'items' must be defined
{k: v for k, v in pairs}       # 'k' and 'v' are local, 'pairs' must be defined
```

The validator tracks nested comprehension scopes, handling complex patterns like nested comprehensions where outer loop variables are available to inner comprehensions.

**Lambda Expression Scope**: Lambda parameters are local to the lambda body. The validator detects lambda parameter names and excludes them from undefined variable checks within the lambda:

```
filter(lambda x: x > threshold, items)  # 'x' is local, 'threshold' and 'items' must be defined
```

**Tuple Unpacking Detection**: Loop targets and lambda parameters may use tuple unpacking syntax. The validator correctly extracts all unpacked names:

```
[a + b for a, b in pairs]           # Both 'a' and 'b' are local
sorted(items, key=lambda a, b: a)   # Both 'a' and 'b' are local
```

**Assignment Statement Detection**: Expressions should not contain assignment statements. The validator detects and rejects:
- Direct assignments: `x = 5`
- Augmented assignments: `x += 5`
- Walrus operator (assignment expressions): `x := 5`

**Blocked Pattern Detection**: The validator enforces security by detecting dangerous patterns:
- **Substring patterns**: `__` (dunder method access)
- **Function calls with word boundaries**: `import`, `exec`, `eval`, `compile`, `open`, `file`, `input`, `globals`, `locals`, `vars`, `getattr`, `setattr`, `delattr`, `breakpoint`

Blocked patterns are detected regardless of where they appear in the expression, preventing bypass attempts through method chaining or nested calls.

**Position Tracking for Error Highlighting**: The AST parser provides precise position information for each node, enabling accurate error highlighting in editor UIs. The validator preserves and reports:
- Line number (1-indexed for display)
- Column number (0-indexed)
- End line and end column for range selection

This position information enables editors to underline the specific portion of an expression that contains an error, rather than highlighting the entire expression.

#### Expressions Are Read-Only

Every place an author writes an expression is a read. That covers a Block's expression fields and `{{ }}` template markers, the per-entry values of a Variable Setter, expression and template variables in the Variable Manager, assertions and every If, Switch, While and Wait Until condition, and a Function Block's input fields. None of them may change the data they inspect, because the value a Block reads is shared with every other Block holding the same variable. Changing a variable is the job of the Variable Setter Block, or of the Python Block for anything a single expression cannot express.

The rule is a single check over the expression's parsed form: a call to a method that changes its receiver in place, or a call to one of the mutable-data constructors, is refused. Building data from a literal or a comprehension is unaffected, and so is any operation that returns a new value rather than changing an existing one.

The rule is enforced twice. At save, the validator walks the same fields, markers and variable declarations it already walks for undeclared references, and refuses the write with a user-actionable error that names the Block and the field. At run, the expression evaluator refuses again, so a test that reached storage another way still cannot change a variable through a read. The refusal an author reads names the offending method or function and says what to use instead; it never nests quotes and never cites security as the reason, because the reason is that the surface only reads.

#### Expression Security Model

The expression evaluation system implements a defense-in-depth security architecture to prevent malicious code execution while enabling legitimate variable expressions. Security operates at multiple layers, with each layer providing independent protection.

**Two-Layer Security Architecture**: Expression security uses a dual-layer approach:

1. **Blacklist Layer (Pre-Evaluation)**: Fast pattern rejection before any evaluation occurs. This layer catches obviously dangerous patterns quickly without the overhead of full expression parsing. Patterns are checked against the raw expression string.

2. **Whitelist Layer (Evaluation Context)**: The primary security boundary. Only explicitly whitelisted functions and types exist in the evaluation namespace. Even if a dangerous pattern somehow bypasses the blacklist, it cannot execute because the required names simply don't exist.

**Blacklist Patterns**: The blacklist detects two categories of dangerous patterns:

| Category | Pattern | Detection Method | Blocked Examples |
|----------|---------|------------------|------------------|
| Dunder access | `__` | Substring match | `__class__`, `__import__`, `obj.__dict__` |
| Dangerous functions | Function names | Word boundary match | `import()`, `eval()`, `exec()` |

Word boundary matching prevents false positives. The pattern `import` blocks `import(x)` and `__import__` but not `my_import_var` or `reimport`.

**Complete Blocked Function List**:

| Function | Risk | Example Attack Vector |
|----------|------|----------------------|
| `import` | Code import | `__import__('os').system('rm -rf /')` |
| `exec` | Code execution | `exec('malicious code')` |
| `eval` | Expression evaluation | `eval('__import__("os")')` |
| `compile` | Code compilation | `compile('code', '', 'exec')` |
| `open` | File access | `open('/etc/passwd').read()` |
| `file` | File access (Python 2) | Legacy file access |
| `input` | User input | `input()` in server context |
| `globals` | Global namespace access | `globals()['__builtins__']` |
| `locals` | Local namespace access | Scope inspection |
| `vars` | Object attribute access | `vars(obj)` |
| `getattr` | Dynamic attribute access | `getattr(obj, '__class__')` |
| `setattr` | Dynamic attribute setting | `setattr(obj, 'attr', value)` |
| `delattr` | Dynamic attribute deletion | `delattr(obj, 'attr')` |
| `breakpoint` | Debugger invocation | `breakpoint()` stops execution |

**Safe Builtins Whitelist**: The evaluation context contains only these explicitly allowed names:

| Category | Allowed Names |
|----------|---------------|
| Types | `str`, `int`, `float`, `bool`, `type`, `list`, `dict`, `set` |
| Collection operations | `len`, `sorted`, `reversed`, `range`, `slice` |
| Math functions | `abs`, `round`, `min`, `max`, `sum` |
| Iteration helpers | `map`, `filter`, `any`, `all` |
| Formatting | `format` |
| Constants | `True`, `False`, `None` |
| Table types | `Table`, `TableRowView`, `ColumnType` |

Any name not in this whitelist raises `NameError` during evaluation, even if it's a valid Python builtin (`tuple`, `enumerate`, `zip`, `pow`, `divmod`, `chr` and `ord` among them). The canvas checks, the suggestion list and the Unused scan read the same list, so a name the canvas accepts is a name the run accepts.

**Table Types in Safe Context**: The Table class and related types are included in safe builtins to enable table operations within expressions:

```python
# Allowed: Table operations
len(my_table)                    # Row count
sum(my_table.col('amount'))      # Column aggregation
[row['name'] for row in table]   # Row iteration
```

**Error Message Formatting**: When evaluation fails due to an undefined name, the system provides user-friendly error messages rather than raw Python tracebacks:

| Scenario | Raw Error | Formatted Message |
|----------|-----------|-------------------|
| Blocked function | `NameError: name 'eval' is not defined` | "Function 'eval' is not supported in expressions (security restriction)" |
| Unknown variable | `NameError: name 'my_vra' is not defined` | "Undefined variable 'my_vra'. Did you mean: my_var?" |
| Unknown function | `NameError: name 'custom_func' is not defined` | "Unknown function 'custom_func'. Available functions: len, sum, min, max, ..." |

**Defense in Depth Rationale**: The dual-layer approach provides defense in depth:

1. **If blacklist fails**: Whitelist still blocks dangerous operations (primary security)
2. **If whitelist has gap**: Blacklist catches common attack patterns (secondary security)
3. **If both fail**: Python's sandboxed eval with restricted `__builtins__` limits damage

This layered approach ensures that multiple independent failures would be required to compromise security, rather than a single point of failure.

#### REQ-3.1.3.15 Template variables

##### REQ-3.1.3.15 Template variables

The template variable system establishes a string value kind whose content is composed from literal text interleaved with `{{ expression }}` markers that render when the value is used. Templates are a peer of literal and expression values: where a literal string is used verbatim and an expression yields a typed value computed from a single Python expression, a template yields a string produced by substituting each `{{ }}` marker with the rendered result of the expression it contains while preserving the surrounding literal text. This gives workflows a readable, text-first way to compose mostly-static strings with a few dynamic insertions, complementing the code-first expression form.

**Template as a value kind**: Template is intrinsic to the value and chosen where the value is authored, not imposed by the surface that consumes it. A string value carries exactly one kind - literal, expression, or template - as part of its definition, so a template renders identically wherever it is referenced. Because a rendered template always produces a string, the template kind is available only for string-typed values; non-string values are literal or expression. A value's kind and type are chosen in the variable's window when it is defined and stay editable there, like every other field of the variable (VMGR-108); a value edit leaves them as they are and changes only the source text.

**Render-on-use and lazy evaluation**: A template renders lazily at the point its value is accessed, on the same model computed (expression) variables use (REQ-3.1.3.13). A template variable that is never referenced never renders; one that is referenced renders against the variable values in scope at first access and caches the result. Rendering substitutes each `{{ expression }}` marker by evaluating the contained expression through the shared expression evaluator - the same evaluator, security model (safe builtins, blocked patterns), and scope resolution described in REQ-3.1.3.13 - and converting the result to its string representation; the literal text between markers is preserved unchanged. A `{{ }}` piece whose result is null fails and names the piece; only the Log message prints `null` for it, and the OpenSearch Query DSL writes JSON `null` where the piece stands alone as a value (see REQ-3.1.3.22). A template's value is therefore always a string regardless of the types its markers evaluate to.

**Single-pass substitution and reference recursion**: Marker substitution is a single pass over the template text - the rendered output is not re-scanned for further markers, so literal brace sequences that appear inside a substituted value are never reinterpreted. Layered composition across templates is achieved through variable references rather than output re-scanning: when a `{{ }}` marker references another variable that is itself a template or an expression, resolving that reference renders it and its result is substituted. This keeps each template's own rendering bounded to a single pass while still allowing templates to build on one another transitively.

**Literal braces and empty markers**: Single braces and empty marker pairs are not rendered. An empty `{{}}` is left as the literal text `{{}}`, and single `{` or `}` characters are preserved, so JSON, YAML, and XML payloads that legitimately contain braces remain safe in template values without escaping.

**Circular dependency and reference validation**: Templates participate in the same static dependency analysis as computed variables. The validation engine extracts the variable references inside each template's `{{ }}` markers, includes templates in the dependency graph alongside expression variables, and rejects any cycle or undefined reference at validation time with a clear diagnostic - so a template referencing a missing variable, or a template/expression cycle, is caught during workflow design rather than at execution. The runtime evaluation guard additionally detects any cycle introduced after validation.

**Debugging and introspection**: A template variable carries a distinct template-kind indicator wherever it appears and, in its per-variable inspect surface, offers the source and the rendered result as switchable views: the source view shows the `{{ }}`-highlighted template text, while the rendered view shows the realized string. A template that has not yet been referenced - and therefore has not rendered - presents an explicit not-rendered state rather than a stand-in value, mirroring the unevaluated-expression presentation.

**Templates in explicit assignment**: Beyond the lazy template-variable kind, the same template syntax and renderer are available as a per-assignment value kind in the Variable Setter control (REQ-3.4.1.14), alongside its per-assignment expression kind. There a template is rendered eagerly at the moment the setter executes, substituting the variable values then in scope, and the assigned target receives the resulting string. This reuses the identical marker substitution, reference recursion, and security rules described above and mirrors the setter's eager expression evaluation; it differs from a template variable only in timing and binding - rendered once at assignment rather than lazily on each reference - so the target holds a plain string rather than a template kind. Authoring a setter assignment as a template is offered only for string-typed assignments and is mutually exclusive with the expression kind, consistent with the template-as-a-value-kind rule.
