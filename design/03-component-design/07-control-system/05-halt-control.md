#### 3.7.5 Halt Control

##### REQ-3.5.1.4 Halt block (stop test/suite/regression execution)

The halt control provides controlled execution termination at configurable scope levels, enabling workflows to stop execution programmatically based on runtime conditions while maintaining proper cleanup and artifact generation.

**Control Purpose and Use Cases**: The halt control allows workflows to terminate execution explicitly when conditions warrant stopping. Primary use cases include early termination when preconditions fail (environment not ready, required services unavailable), stopping execution when critical errors occur that make further testing meaningless, implementing circuit-breaker patterns where failures trigger immediate halt to prevent cascading issues, enforcing quality gates where test results determine whether to continue or stop, and enabling conditional execution paths where certain outcomes should halt the entire workflow. The halt control provides explicit, self-documenting termination points in workflow visualizations.

**Canvas Rendering**: The halt block is a terminal dead-end. It renders with a top target handle (input) but no bottom source handle (output). No edges can originate from a halt block. This visually communicates that execution stops here - there is no downstream path. The block displays the stop/octagon icon consistent with its flow-terminating nature.

##### REQ-3.5.1.14 Halt scope configuration (test/suite/regression level)

**Configurable Halt Scope**: The control accepts a scope parameter determining the extent of execution termination. Test scope halts only the current test execution, allowing subsequent tests in a suite to run normally. Suite scope halts the current suite execution, stopping all remaining tests in the suite while allowing subsequent suites in a regression to continue. Regression scope halts the entire regression execution, stopping all remaining suites and tests immediately. Function scope halts only the innermost enclosing Function body and reports the halt's status as the Function Block's result in the flow it sits in, so a Function can terminate its own execution early without stopping the Test that runs it. Because function scope is only meaningful inside a Function, the canvas offers it only when the active surface is a Function body and validation rejects a function-scope halt placed at the Test top level. This scope configurability enables workflows to implement appropriate termination granularity based on failure severity - a Function aborts only itself, minor issues halt only the failing test, significant issues halt the suite, and critical environment failures halt the entire regression.

**Halt Status**: The control accepts a status parameter (passed, failed, skipped) that determines both the visual presentation and the semantic meaning of the halt. A halt with status "passed" represents an intentional, successful early termination (e.g., "all preconditions verified, no further testing needed"). A halt with status "failed" represents a failure-driven termination (e.g., "environment is broken, cannot continue"). A halt with status "skipped" represents a conditional bypass (e.g., "this test path is not applicable to this configuration"). The status directly controls execution visualization and log level as described below.

**Halt Message**: The control accepts an optional message parameter providing human-readable explanation appearing in execution logs and reports. The message populates the execution artifact metadata, ensuring halt context is preserved for post-execution analysis and debugging.

**Execution Visualization**: The halt block's post-execution appearance varies by the configured status parameter:

| Status | Block Border | Icon | Log Level | Semantics |
|--------|------------|------|-----------|-----------|
| passed | Green | Checkmark | INFO | Intentional successful termination |
| failed | Red | X | ERROR | Failure-driven termination |
| skipped | Amber | Warning | WARN | Conditional bypass |

This follows the same color/level conventions used by the assert control for consistency across the control system.

**Post-Halt Behavior**: When the execution engine encounters a halt signal, it behaves identically to a manual stop initiated by the user. All blocks remaining in the execution queue are marked as skipped and paint Skipped on the live canvas as they are marked, the same paint a page reload restores. The execution loop breaks immediately - no further blocks are visited. The overall session status maps directly from the halt status parameter: passed maps to session PASSED, failed maps to session FAILED, and skipped maps to session SKIPPED (a dedicated terminal state shown in amber). This ensures consistent UX between programmatic halts and user-initiated stops.

**Halt Signal Mechanism**: The halt control communicates with the execution engine through a reserved key in the result dictionary, following the same pattern used by branching controls for edge routing and by the assert control for test failure signaling. The engine consumes this signal before processing output bindings, preventing halt metadata from leaking into user variables.

**Halt vs Error vs Skip**: The halt control is semantically distinct from error failures and block skipping. Errors indicate unexpected failures during execution and trigger error handling logic (retries, catch blocks). Skips bypass individual blocks without executing them. Halts are explicit, intentional terminations based on evaluated conditions. Workflows should use errors for unexpected failures, skips for conditional block bypass, and halts for deliberate termination decisions.

**Parameter Schema**:

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| scope | string enum (test, suite, regression, function) | no | test | Scope level at which to halt execution; function halts the innermost enclosing Function and reports its status as its Function Block's result |
| status | string enum (passed, failed, skipped) | no | failed | Status to assign to the halt - controls visualization and log level |
| message | string | no | - | Human-readable reason for halting, appears in logs and artifacts |

**Suite/Regression Scope Propagation**: When a halt with `scope=suite` or `scope=regression` fires inside a test that is part of a suite or regression execution, the halt signal is forwarded from the workbench session's terminal event to the parent execution's orchestrator. The orchestrator applies the halt's status to the current entry and fans the halt out at the indicated scope: `scope=suite` marks every remaining test in the current suite as `skipped` and sets the suite rollup to the halt's status (for a regression, the next suite still runs); `scope=regression` marks every remaining test in the current suite AND every remaining suite's tests as `skipped` and sets the regression rollup to the halt's status. `scope=test` does not fan out beyond the current test; the suite/regression continues with the next entry.

**Function Scope Absorption**: A halt with `scope=function` is absorbed at the innermost enclosing Function boundary instead of draining outward the way the other scopes do. The boundary translates the halt's status into the Function Block's reported result on the canvas it sits in: `passed` and `skipped` complete the invocation normally - the flow the Block sits in continues to the Block after the Function Block, which paints the matching badge - while `failed` raises through the boundary as an invocation failure that flow handles like any other failed Block, including being caught by a Try/Catch wrapping the Function Block. The halt is not re-emitted as a drain past the Function, so the downstream blocks of the flow the Block sits in are governed only by the status the Function reported (continue on passed/skipped, error handling on failed). The engine distinguishes the cases by scope: for test, suite, and regression it re-emits the halt drain described above; for function it consumes the signal at the boundary and converts it into the result or exception that matches the chosen status, so a Function's self-termination composes with the flow its Function Block sits in exactly like a plugin or control that finishes, is skipped, or fails.

**Related Behaviors**:

- **Parallel branch interruption**: A halt inside one branch of a parallel fork ends the whole section. Sibling branches stop at their next block boundary and reach a terminal state there: the block a branch held when it stopped paints Stopped, and the blocks it never reached paint Skipped. The halt then carries its declared scope and status outward from the section exactly as it does elsewhere.
