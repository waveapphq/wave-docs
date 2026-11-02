### 3.6.2 Test Variable Model

#### REQ-3.1.3.2 Self-contained test variables with namespace access

Test definitions contain only test-local variables, each with a name, a type, and a value where the author gave one. There are no external references, no `resolved_from` annotations, and no dependency tracking against higher-scope variables. Higher-scope variables (system, suite, regression) are accessed at runtime through namespace expressions resolved by namespace objects.

**Self-Contained Variables**: Every variable stored in a test definition includes a name and a type, and carries a value only when the author gave it one. A variable always holds a value: one declared without a value is stored with no value at all rather than an invented one and holds null, and the creation editors preset the type's default (empty string, zero, false, empty collection) as the starting value (see REQ-3.1.3.20). No entry is a placeholder for, or a reference to, a variable held at another scope. A test's variable set is complete and self-describing - loading a test does not require resolving any variables against other scopes.

**No External References**: Test definitions do not store references to variables from system, suite, or regression levels. The test file contains only what the test owns. Higher-scope variables are accessed through namespace expressions at runtime, not through stored references that require resolution.

**Namespace Expressions at Runtime**: When a test expression references `system.timeout`, the runtime constructs a namespace object for the system scope and resolves the attribute. Namespace objects provide attribute-based access to the variables defined at their respective level. This resolution happens at execution time, not at save or load time.

**Validation on Save**: A declaration with no value saves as declared and holds null; the save is not blocked. The creation editors offer the type's default as the starting value. What save validation does require is a complete declaration - a name and a type - and that every variable a workflow references is declared: a reference to an undeclared variable is an error the user resolves explicitly, by adding the variable or by removing the reference.

**No Shadowing**: Test-level variables cannot share names with reserved namespace identifiers (`system`, `suite`, `regression`). Suite and regression variables are read only through their namespaces, so each variable reference maps to exactly one source without shadowing rules.

**Load Pipeline**: Loading a test returns its variables directly with no assembly, merging, or annotation step. The variable set stored on disk is the variable set used in the editor. No resolution against other scopes occurs at load time.

**Save Pipeline**: On save, the system stores the test's variables as declared. There is no scan for external variable usage, no external reference extraction, and no hierarchy validation. The test's variable set is persisted as-is.
