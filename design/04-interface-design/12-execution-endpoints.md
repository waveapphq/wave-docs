### 4.12 Execution Control & Monitoring Endpoints

#### 4.12.1 Execution Operations

##### REQ-5.2.4.6 Execution control and monitoring endpoints

Execution control and monitoring endpoints provide lifecycle operations for test, suite, and regression executions enabling initiation, state management, progress tracking, and observability. Executions represent runtime instances of defined workflows with independent state, variable contexts, and result capture, supporting both automated CI/CD integration and interactive debugging scenarios.

**Lifecycle Management**: The system supports full execution lifecycle operations from initiation through completion or stop. Execution creation accepts the target entity type and identifier along with optional variable overrides and debug configuration. Each execution receives a unique identifier enabling subsequent state queries and control operations. Executions progress through defined states including pending during initialization, running during active processing, passed on successful termination, failed on error conditions, and stopped on explicit termination requests or orphan recovery after backend restart.

**Subset Execution for Suites and Regressions**: Suite and regression execution requests may include an optional test subset that restricts which tests run. For a suite, the subset is an ordered collection of test identifiers belonging to the suite. For a regression, the subset is an ordered collection of `(suite_id, test_id)` pairs, allowing per-context selection when the same test belongs to multiple suites within the regression. The execution service validates that every referenced identifier exists in the target entity before initiation and rejects malformed or unknown references with a validation error. The requested subset is persisted on the execution record so clients can reconstruct exact selection state for a subsequent re-run without recomputation. Executions created without an explicit subset run the full entity. Tests excluded by the subset are recorded in the timeline with status `skipped` (zero duration, no log entries, no variable capture) so the execution's shape reflects the parent entity while the runtime cost is confined to what the user asked for. The `skipped` status participates in timeline retrieval and history responses the same way as other terminal statuses.

**Regression Duplicate-Test Awareness**: Regression retrieval responses include a duplicate-test map identifying tests that appear in more than one suite of the regression. Each entry pairs a test identifier with the list of suite identifiers that reference it. This map supports client affordances such as in-dialog badges and execution-detail indicators explaining why a test may run multiple times within a single regression execution.

**State Transitions**: Execution state management follows a deterministic state machine model. State transitions validate preconditions ensuring operations apply only to executions in appropriate states.

**Execution Deletion**: Terminal executions (passed, failed, or stopped) can be permanently removed from storage via deletion operations. Delete requests for running or paused executions return conflict status requiring stop operation first. Deletion removes execution records, logs, variables, timelines, and breakpoints from storage. This operation is irreversible and intended for history cleanup.

**Progress Tracking**: Progress endpoints expose real-time execution advancement through plugin sequences. Progress responses include current position within the total plugin count and calculated percentage completion. Progress updates occur at plugin boundaries providing granular visibility into long-running executions. Current position endpoints return the actively executing block and plugin identifiers with their names and processing status enabling UI synchronization with execution flow.

**Runtime Variable Snapshot**: Execution-specific variable endpoints return the current runtime state of variables within an active execution context. Unlike static variable management endpoints that operate on entity definitions, runtime snapshots reflect values as modified during execution including plugin outputs and computed results. Secret variables return masked placeholders preserving security while indicating presence. Runtime snapshots support debugging scenarios where current state inspection guides troubleshooting decisions.

**Log Retrieval**: Log endpoints provide access to execution output captured during processing. Log responses include timestamped entries with severity levels spanning debug, info, warning, and error classifications. Log filtering by level enables focused retrieval of significant events. Log entries include source attribution identifying the originating block or plugin. Log retention follows artifact lifecycle policies.

**Timeline Analysis**: The report carries the execution timeline (see 4.14).

**Execution History**: Entity-specific execution history endpoints return previous executions for individual tests, suites, or regressions. History responses support pagination for entities with extensive execution records. History entries include summary information enabling quick assessment of recent execution patterns, success rates, and timing trends without requiring individual execution retrieval.

**Statistics Aggregation**: Statistics endpoints provide aggregate metrics across execution populations. Statistics responses include total execution counts, success and failure rates, and average durations over configurable time windows. Statistics filtering by entity type and identifier enables focused analysis of specific tests or suites. Statistics support dashboard displays and automated quality gate assessments.

**Listing and Filtering**: Execution listing supports comprehensive filtering enabling discovery of relevant executions. Filters include status selection, entity type restriction, specific entity targeting, and temporal range bounding. Pagination ensures scalable retrieval across large execution histories. Sort ordering supports chronological and reverse-chronological access patterns. List responses include essential metadata enabling assessment without individual retrieval.

**Error Handling**: Operations enforce validation through consistent error responses. Invalid execution identifiers return not found with context information. State transition violations return conflict status indicating current state and valid transitions. Invalid execution requests return bad request with parameter validation details. Entity references to non-existent tests, suites, or regressions return not found with resolution guidance. Internal execution failures return server error with correlation identifiers enabling log correlation and issue investigation.

**Concurrency Management**: The execution system supports concurrent execution instances with independent state isolation. Variable contexts maintain per-execution scope preventing cross-contamination.

#### 4.12.2 Entity Retrieval

##### REQ-5.2.4.6 Entity resolution for execution

When an execution request is received, the execution service must resolve the entity reference to retrieve the complete entity definition before execution can proceed. This entity retrieval bridges the execution layer with the entity management layer.

**Entity Type Dispatch**: Execution requests specify both an entity type (test, suite, or regression) and an entity identifier. The execution service uses the entity type to dispatch the retrieval request to the appropriate entity service. Test executions retrieve from the test service, suite executions from the suite service, and regression executions from the regression service. This dispatch pattern maintains separation of concerns between execution orchestration and entity storage.

**Service Integration**: The execution service integrates with entity services through their public interfaces. Each entity service provides a retrieval operation that accepts an identifier and returns the complete entity definition or raises an appropriate error if not found. The execution service delegates retrieval entirely to these services, avoiding direct storage access and ensuring consistent entity resolution behavior across all access paths.

**Entity Validation**: Upon successful retrieval, the execution service validates that the entity is in an executable state. Tests must have at least one workflow block defined. Suites must reference at least one test. Regressions must reference at least one suite. Invalid entities return appropriate error responses before execution initialization occurs.

**Error Handling**: When entity retrieval fails, the execution service returns HTTP 404 Not Found with context identifying the entity type and identifier that could not be resolved. This error response enables clients to distinguish between invalid execution requests (malformed parameters) and missing entity references (valid request for non-existent entity).
