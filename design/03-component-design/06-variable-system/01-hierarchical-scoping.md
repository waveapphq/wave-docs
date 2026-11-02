### 3.6.1 Variable Scoping

#### REQ-3.1.3.1 Variable scoping with namespace-based access

##### REQ-3.1.3.1 Variable scoping with namespace-based access

The variable system stores variables at multiple organizational levels - system, regression, suite, and test - where each level maintains its own independent variable set. There is no inheritance chain or implicit scope walking. Test-level variables are accessed directly by name in expressions. Variables at other levels are accessed through explicit namespace prefixes that make the source of each variable visible and unambiguous.

**Storage Levels**: System (managed through Settings), Regression (campaign-wide configuration), Suite (test grouping configuration), Test (workflow-specific parameters). Each level stores its own variables independently with name, type, and value. No level references or resolves variables from another level.

**Test-Level Variable Access**: Variables defined on the test itself are directly accessible by name in expressions. A test variable named `timeout` is referenced as `timeout` in any plugin parameter or computed expression within that test's workflow.

**Namespace-Based Access to Other Levels**: Variables from system, suite, and regression levels are accessed via explicit namespace prefixes in expressions. A system variable named `timeout` is referenced as `system.timeout`. A suite variable named `deploy_env` is referenced as `suite.deploy_env`. A regression variable named `release_version` is referenced as `regression.release_version`. The namespace prefix makes the variable's source immediately visible.

**Reserved Names**: The identifiers `system`, `suite`, and `regression` are reserved and cannot be used as test-level variable names. This prevents ambiguity between test-local variables and namespace references. Variable name validation rejects these reserved names at creation time with a clear error message.

**No Scope Chain Walking**: When a variable name is referenced without a namespace prefix, it resolves only against test-level variables. If no test-level variable with that name exists, the reference is unresolved - the system does not search suite, regression, or system levels. This eliminates implicit dependencies and makes every variable reference self-documenting.

**Read-Only Namespace Access**: Namespace objects provide read-only attribute-based access to their respective variable sets. Variables cannot be assigned to namespace targets - attempting to set a value on `system.x`, `suite.x`, or `regression.x` is rejected. Only test-level variables support mutation during workflow execution.

**Scope Isolation**: Each scope's variables are isolated from siblings at the same level. Variables in one suite are independent of another suite's variables. Variables in one regression are independent of another regression's variables.

**System Variable Management**: System variables are managed through a dedicated settings interface, independent of any regression, suite, or test. They use the same type system (string, integer, float, boolean, array, map, table) and support secret masking. Changes take effect when namespace objects are constructed at execution time.
