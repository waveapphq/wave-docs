#### 3.7.9 Try/Catch Control

##### REQ-3.2.2.6 Try-catch error handling per block

The try/catch control provides structured error handling within visual workflows, enabling tests to intercept failures from any block in a protected scope, inspect structured error context, and execute recovery or diagnostic logic through a catch path. An optional finally path guarantees cleanup execution regardless of success or failure outcomes.

**Control Purpose and Use Cases**: The try/catch control transforms errors from terminal events into manageable conditions. Primary use cases include wrapping unreliable operations (HTTP calls to external services, SSH commands on flaky hosts), validating error behavior (confirming a plugin produces the expected error type), implementing cleanup patterns (resource teardown that must run regardless of outcome), and building resilient test flows that continue executing after handling anticipated failures. The control integrates with WAVE's existing error infrastructure (ErrorClassifier, ErrorContext) to provide rich error diagnostics without introducing new concepts.

**Path Architecture**: Unlike branching controls (if/else, assert split) where exactly one branch executes and all branches converge at a merge point, try/catch is a scoped sequential control that may execute multiple paths in sequence (try, then catch on error, then finally). This fundamental difference drives a distinct egress model:

- **Path handles** (try, catch, finally): Each leads to a dead-end scoped chain of blocks. Terminal blocks in these paths have no outgoing edges. The engine traces from each handle and executes the chain until it reaches the dead-end. Paths are isolated - cross-path connections (e.g., a finally-path block connecting to a catch-path block) are prevented by edge validation at creation time.
- **Egress handle** (`then`): A dedicated output handle that connects to the next block in the main flow. This is the only exit point from the try/catch construct. After all applicable paths complete, the engine follows the `then` handle's edge to continue execution. The `then` handle renders as a default gray connector (like a normal block's output), visually distinct from the color-coded path handles.

This explicit egress model eliminates the need for convergence detection. The engine does not infer merge points from topology - it knows the exit point structurally via the `then` handle.

**Path Scope Isolation**: Each block within a try/catch construct belongs to exactly one scope: a path scope (try, catch, or finally) or the outer flow. Scope membership is established when a block connects downstream of a path handle and propagates through the chain. Path scopes are strictly isolated - no edges may cross between scopes. This isolation is enforced at edge-creation time with the following rules:

1. **Cross-path rejection**: An edge from a block in one path (e.g., try) to a block in another path (e.g., catch or finally) of the same try/catch is rejected.
2. **Inner-to-outer rejection**: An edge from a scoped path block to an outer-flow block is rejected. Path chains are dead-ends - the `then` handle is the only exit.
3. **Outer-to-inner rejection**: An edge from an outer-flow block to a scoped path block is rejected. Only try/catch handle edges create scope entry points.
4. **Same-scope allowed**: Edges between blocks in the same scope are allowed, including branching and merging within a path (e.g., an if/else inside the try path with branches that reconverge).

Visual feedback for rejected edges: the same red circle arrow indicator used for cycle detection. The user sees immediate feedback that the connection is structurally invalid.

Scope tracking uses the path color tags already maintained for the colored left border rendering. Each block stores its scope identity (null for outer flow, or try-catch ID + path name). Validation at edge-creation time is O(1) - compare source and target scope tags. When edges are created or deleted, scope tags propagate to affected downstream blocks, bounded by the number of blocks in the chain (typically small). For the backwards-check scenario where the target block has multiple incoming edges, all incoming paths must resolve to the same scope - if any incoming path leads to a different scope, the new edge is rejected.

**Canvas Rendering**: The try/catch block renders with a variable number of output handles based on configuration. The `try` handle and `then` egress handle are always present. The `catch` and `finally` handles appear based on their respective toggle parameters.

| Configuration | Handles | Count |
|---------------|---------|-------|
| Default (catch on, finally off) | try (blue) + catch (red) + then (gray) | 3 |
| Catch on, finally on | try (blue) + catch (red) + finally (amber) + then (gray) | 4 |
| Catch off, finally off | try (blue) + then (gray) | 2 |
| Catch off, finally on | try (blue) + finally (amber) + then (gray) | 3 |

Blocks connected downstream of each path handle inherit a colored left border matching the path color (blue for try-path blocks, red for catch-path blocks, amber for finally-path blocks). Edges between the try/catch control and downstream path blocks use the same color coding. The `then` edge uses default styling.

**Catch Path Toggle**: The catch path is enabled by default and can be disabled via the `enable_catch` parameter. When catch is disabled, the catch handle is hidden and no catch-path blocks execute on error. The error variable and `caught` output are still populated regardless of whether catch is enabled - the control always catches errors and records them. Disabling catch reduces visual clutter when the user only needs a protected scope (try + then) or a cleanup pattern (try + finally + then) without explicit error handling logic.

**Single Catch Path with Composability**: The control provides one catch path rather than multiple typed catch handlers. Users who need to branch on error type, retryability, or category place an if/else control inside the catch path and reference `error.type`, `error.is_retryable`, `error.category`, or any other field from the error variable. This reuses existing controls rather than adding multi-catch complexity to the try/catch control itself.

##### REQ-3.2.2.9 Continue execution after caught errors

**Continuation Semantics**: When the try/catch construct completes (all applicable paths finish without propagating an error), the engine follows the `then` handle's edge to continue normal execution. The handled error is recorded in execution history and artifacts but does not affect test success status unless catch-path blocks explicitly set failure state.

**Flow Resumption**: Execution always resumes via the `then` egress handle, never re-entering a completed path. Try-path blocks that were not executed due to an error remain marked as skipped. The resumption point is deterministic and structurally defined by the `then` handle's outgoing edge.

**Variable Scope Continuity**: Variables modified during try-path execution (before the error) and during catch-path execution remain in scope after the try/catch construct completes. Catch-path blocks can set recovery variables, status flags, or alternative values that subsequent blocks reference. This enables patterns like setting a `recovery_status` variable in catch and inspecting it downstream.

**What the catch path catches**: The catch path catches block failures - a block that fails and stops the flow, whatever the block is: a plugin error, a failed nested control, a single-mode Assert or a Wait Until failing under Fail Current Scope, or a Function instance that reports failed. A split-mode Assert that marks the current scope failed is not a block failure: its flow continues down its fail branch, so the try path completes normally and the catch path does not run. A Try/Catch path is not a scope, so the mark lands on the enclosing root Test or Function body, and a Try/Catch never clears it: the Test still ends failed, or the Function Block still reports failed. Wrapping a Function instance in a Try/Catch catches that Function's reported failure like any block failure, which is how a test goes on past a Function whose body was marked failed. Halt is NOT absorbed - a Halt control inside the try-body still propagates its halt signal through Try/Catch per its scope parameter.

##### REQ-3.2.2.10 Finally/finalize blocks for guaranteed cleanup
##### REQ-3.2.2.11 Finalize blocks execute on normal termination
##### REQ-3.2.2.12 Finalize blocks execute on abnormal termination

**Finally Path Behavior**: When `enable_finally` is true, the finally path executes unconditionally after either the try path (on success) or the catch path (on failure), including when a Break or Continue leaves either path. This guarantee holds regardless of whether the try path succeeded, the catch path succeeded, or the catch path itself failed. The finally path provides cleanup semantics for operations like closing connections, deleting temporary resources, or logging final state.

**Finally Failure Consequences**: If the finally path itself throws an error:

- If try/catch had succeeded: the success is converted to failure (REQ-3.2.2.11). The test fails despite the try path succeeding.
- If try/catch had already failed: the finally error is appended to the error context alongside the original error. Both errors are preserved in artifacts for diagnosis.

This follows the principle that finally failures are never silently swallowed - they always surface.

**Catch Failure with Finally**: If the catch path fails, the finally path still executes before the catch error propagates upward. This preserves the guarantee that finally always runs. After finally completes (or fails), the catch path's error propagates, causing the session to fail. Both the original try error and the catch error are preserved.

**Break and Continue Inside a Path**: A Break or Continue is not a failure. When one is issued anywhere inside a try, catch or finally path - directly, or inside a branch, a Switch/Case case or another container in that path - and targets a loop that encloses the Try/Catch, it leaves the Try/Catch the way `break` and `continue` leave `try ... finally` in a programming language:

- The path it is issued in stops at that point: nothing after it in that path runs.
- The catch path does not run because of it.
- The finally path runs once before the signal continues outward. A Break or Continue issued in the finally path itself stops the rest of that finally path and continues outward in place of any signal the finally path was carrying.
- A signal that crosses several nested Try/Catch constructs runs every finally path it passes, innermost first.
- The `then` successor of every Try/Catch it leaves does not run. The targeted loop then acts as for any Break or Continue: Break ends it and its `then` successor runs, Continue starts its next iteration. A loop between the Try/Catch and the target does not consume the signal.
- If a finally path fails while carrying the signal outward, the failure wins, as for any failed block: the Try/Catch fails and no loop acts on the signal. A Halt in that finally path wins the same way.
- If the user stops the run while a finally path is carrying the signal outward, the signal is dropped: no loop acts on it or logs it, and the run ends stopped.

**Execution Flow Summary**:

| Scenario | Try | Catch | Finally | Session Outcome |
|----------|-----|-------|---------|-----------------|
| Try succeeds | Executes | Skipped | Executes (if enabled) | Passes |
| Try fails (exception), catch succeeds | Partial, then skipped | Executes | Executes (if enabled) | Passes |
| Try fails (exception), catch disabled | Partial, then skipped | Disabled (error still caught) | Executes (if enabled) | Passes |
| Try fails (exception), catch also fails | Partial, then skipped | Fails | Executes (if enabled) | Fails (catch error propagates) |
| Try body marks the scope failed (split-mode Assert with Fail Current Scope) | Completes normally | Skipped | Executes (if enabled) | Fails (the mark stays on the enclosing Test or Function) |
| Try succeeds, finally fails | Executes | Skipped | Fails | Fails (success converted) |
| Try issues a Break or Continue | Stops at the signal | Skipped | Executes (if enabled), then the target loop acts | Passes |
| Try fails, catch issues a Break or Continue | Partial, then skipped | Stops at the signal | Executes (if enabled), then the target loop acts | Passes |
| Finally issues a Break or Continue | Executes | Skipped, or executes on a try failure | Stops at the signal, then the target loop acts | Passes |
| A Break or Continue leaves try or catch, finally fails | Stops at the signal | Skipped, or stops at the signal | Fails | Fails (the failure wins, no loop acts on the signal) |

**Execution Engine Integration**: The control returns a result that directs the engine to the try path first. The engine identifies path membership by tracing from each handle - each handle leads to an isolated dead-end chain of blocks. The engine wraps try-path block execution in error interception logic. On exception, the engine populates the error variable, marks remaining try-path blocks as skipped, and executes the catch-path chain (if catch is enabled). After catch (or after try if no error), the engine executes the finally-path chain (if finally is enabled). After all applicable paths complete, the engine follows the `then` handle's edge to continue the main flow. This reuses the existing sourceHandle-based edge routing infrastructure.

**Parameter Schema**:

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `enable_catch` | boolean | no | `true` | When enabled, shows the catch output handle for error handling path blocks |
| `enable_finally` | boolean | no | `false` | When enabled, shows the finally output handle for guaranteed cleanup path blocks |

**Output Variables**:

| Output | Type | Description |
|--------|------|-------------|
| `error` | map | The caught error object (null if try succeeded). Populated regardless of catch enabled/disabled. Bind to a variable via output bindings to access error fields in catch-path blocks. |
| `caught` | boolean | Whether an error was caught (true if try failed). Populated regardless of catch enabled/disabled. |

**Error Variable Schema**: When a try-path block throws an exception, the engine constructs an error object using the existing ErrorClassifier and ErrorContext infrastructure:

| Field | Type | Source | Description |
|-------|------|--------|-------------|
| `type` | string | `type(exception).__name__` | Python exception class name |
| `message` | string | `str(exception)` | Human-readable error message |
| `node` | string | The failed block's Name | Name of the failed block for user reference; never its Description |
| `node_id` | string | React Flow node ID | Unique identifier of the failed block |
| `element_type` | string | Plugin/control type | Type identifier of the failed element |
| `timestamp` | string | ISO 8601 | When the error occurred |
| `is_retryable` | boolean | ErrorClassifier | Whether the error is transient and retryable |
| `category` | string | ErrorClassifier | Error classification: transient, permanent, or critical |
| `stack_trace` | array | Python traceback | Full Python traceback for debugging, one entry per line |

This schema is documented in the control's Details dialog (accessible via the footer button in the config modal), providing in-UI reference for users composing catch-path logic.

**Config Modal Layout**: The configuration follows the standard single-scrollable-modal pattern with Name field, a PATH TOGGLES section with CATCH radio group (Enabled/Disabled) and FINALLY radio group (Disabled/Enabled), OUTPUT VARIABLES section with error (map) and caught (boolean) assignment rows, and the standard footer with Changelog, Details, Profiles, Cancel, and Save buttons. The radio groups follow the same pattern as the assert control's BRANCHING MODE radio group. The error object's variable binding is configured via the output binding combobox (same as all other control outputs), not via a separate parameter.
