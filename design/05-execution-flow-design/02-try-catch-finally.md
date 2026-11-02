### 5.2 Error Handling Design

#### 5.2.1 Try-Catch-Finally Semantics

##### REQ-3.2.2.6 Try-catch error handling per block

Try-catch error handling provides structured exception management at the block level, enabling test workflows to intercept failures, implement recovery strategies, and control error propagation through explicit handling logic. This transforms errors from terminal events into manageable conditions that tests can inspect, respond to, and recover from, enabling sophisticated error handling patterns while preserving the visual workflow paradigm.

**Error Handling Semantics**: The try-catch construct establishes a protected execution scope around one or more blocks where errors are intercepted before propagating upward. When a block encounters an error - from plugin failure, assertion failure, timeout, or resource unavailability - the engine captures error context and transfers control to the catch handler instead of terminating the test. The handler receives a structured error object containing type, message, origin location, and execution context, then executes within the same workflow with access to the current variable scope for inspection, recovery, logging, or propagation decisions. This mirrors traditional exception handling while adapting to workflow execution where handlers are sequences of blocks.

**Scope of Protection**: Try-catch protects against operational errors including plugin failures, assertion failures, timeouts, and resource unavailability. Protection encompasses all nested blocks within the try construct, enabling hierarchical handling where inner constructs address specific errors while outer constructs provide fallback handling. Errors match to the innermost applicable handler based on execution stack. System-level errors like engine failures, memory exhaustion, or context corruption are not catchable.

**Error Context Capture**: An intercepted error produces a structured error object holding its type, message, originating node, category, timestamp and stack trace. The object is stored as a variable in the handler scope for inspection, and the run log carries the rest.

**Handler Execution Model**: Catch handlers execute as conditional workflow branches with access to the same variable scope and execution context as the try block. Handlers perform operations including logging errors, executing recovery blocks, setting outcome variables, invoking cleanup, or suppressing errors by completing normally. Special variables containing the error object are accessible via standard variable references. Handler execution follows normal state management and scoping rules. Normal handler completion indicates successful error handling and continues execution after the try-catch. A failing handler fails the run and sets its error text, unless an outer try-catch catches that failure.

**Execution Flow Control**: Try-catch introduces conditional flow based on error occurrence. Without errors, try blocks execute sequentially, catch is skipped, and execution proceeds after the construct. With errors, try execution halts immediately, control transfers to catch, handler blocks execute sequentially, and outcome determines next steps. Successful handling suppresses propagation and continues execution. A failing handler fails the run and sets its error text, unless an outer try-catch catches that failure. This ensures deterministic, predictable execution state after try-catch completion.

**Integration with Hierarchy**: Try-catch integrates with other control mechanisms and hierarchical structure. Constructs nest for layered handling. Variable scoping follows hierarchical rules where handlers access parent scopes including try scope. Parallel branches have independent try-catch constructs.

##### REQ-3.2.2.9 Continue execution after caught errors

Execution continuation after caught errors transforms failures into handled events that enable workflow recovery, allowing tests to implement resilient patterns where anticipated errors are managed gracefully while maintaining complete diagnostic context in execution history.

**Continuation Semantics**: When a catch handler completes successfully, the engine treats the error as fully handled and resumes normal execution immediately following the try-catch construct. The test transitions back to running state, execution proceeds to the next block, and the error is recorded in history but does not affect test success status unless explicitly set by the handler. The handled error is preserved in the run log for analysis but does not propagate to parent levels, allowing tests to handle multiple errors during execution while continuing to completion.

**Flow Resumption Point**: Execution resumes at the first block following the try-catch construct, not at the point of failure within the try block. Blocks within the try scope that were not executed due to the error remain skipped. The resumption point is deterministic and always the next sequential element after the entire construct, preventing inadvertent re-entry or execution loops.

**State Normalization**: Before resuming, the engine normalizes state by clearing the current error object from active context while preserving it in history, marking the try-catch as completed successfully, and ensuring variable scopes reflect handler modifications. Error details remain accessible in execution history and the run log but do not affect subsequent block execution.

**Variable Scope Continuity**: Variables modified during the try block and catch handler remain in scope with their values preserved during continuation. This enables handlers to set recovery variables, status flags, or alternative result values that subsequent blocks can reference. The variable context remains intact, allowing conditional logic after try-catch to inspect handler-set variables and adapt behavior based on whether errors were handled.

**Success Status Management**: Continuation ensures execution proceeds but does not guarantee test success. Final success or failure is determined by all execution outcomes including handler status assignments, subsequent assertion results, and final completion state. Handlers may explicitly set outcome variables to propagate error impact or suppress it by not setting failure status. This separates continuation (control flow) from outcome determination (success evaluation), enabling patterns where some errors are informational, some are warnings, and some are failures despite being handled.

**Execution History and Artifacts**: The run log carries the error handling lifecycle, which enables post-execution analysis to distinguish tests that completed without errors from those that handled errors or failed.

**Integration with Workflow Control**: Continuation respects all workflow semantics including sequential ordering, nested constructs, and state management. Handlers may set variables for subsequent conditional inspection, prepare alternative resources for continued execution, or set breakpoints for interactive analysis.

**Nested Try-Catch Continuation**: Continuation applies independently at each nesting level. Inner try-catch handling allows execution to proceed within outer try blocks. Outer try-catch handling errors from inner constructs continues after the outer construct. This hierarchical model enables layered resilience where inner handlers address specific errors while outer handlers provide fallback strategies, with each level's continuation decision independent and based on handler completion.
