### 4.6 Test Workflow Endpoints

#### 4.6.1 Test CRUD Operations

##### REQ-5.2.4.3 Test workflow operations endpoints

Test workflow endpoints provide lifecycle operations for test entities enabling creation, modification, version control, workflow management, variable configuration, and validation. Tests are composable workflows built from blocks with sequential and parallel execution paths, control flow structures, and variable scoping, forming the executable units within suites and regressions.

**Lifecycle Management**: The system supports full CRUD operations with versioning. Test creation establishes metadata including name, description, workflow definition, variable declarations, and associated metadata. Tests maintain workflow graphs consisting of nodes (blocks, control flow elements) and edges (execution paths). Retrieval supports individual access by identifier and bulk listing with filtering by suite association, tags, and text search. Deletion enforces dependency checking to prevent orphaned suite references, and it removes the test's test data with the test. Batch operations enable efficient multi-test create, update, and delete workflows.

**Version Control**: Every test maintains version history following semantic versioning enabling temporal tracking and rollback. Workflow modifications create new versions preserving historical snapshots. Version creation validates workflow integrity and block reference compatibility. Version operations include adding new versions, deleting versions with dependency analysis, viewing complete version history, and restoring to previous versions. Published test versions become immutable, ensuring stable references for suites.

**Workflow Structure**: Tests define execution graphs through workflow endpoints. The workflow consists of nodes representing execution points (start, end, blocks, fork/join for parallelism, conditionals, loops, try/catch/finalize for error handling) and edges representing execution flow with optional conditions. Workflow retrieval returns the complete graph structure suitable for visualization and execution planning. Workflow validation ensures structural integrity, reachability, and termination guarantees.

**Variable Management**: Test-level variables define the variable scope for execution. Variable endpoints support retrieving current variable declarations and updating variable definitions. Variables support hierarchical scoping from regression through suite to test level with override capability. Variable updates validate type consistency and expression syntax for computed variables. Secret variables receive special handling with masking indicators.

**Discovery and Filtering**: List operations support page-based pagination for consistent navigation. Filtering includes suite association selection, tag-based grouping, and text search across names and descriptions. Responses include essential metadata enabling quick capability assessment including workflow summary and variable counts.

**Validation**: Context-aware validation verifies structural integrity through a two-phase architecture with three validation contexts. The persistence path (create and update) runs validation through a single unified pipeline so both endpoints behave identically.

**Validation Endpoint** (`POST /tests/{id}/validate?context={context}`): The `context` query parameter determines severity classification. Three contexts are accepted by this endpoint: `load` (opening a test in the editor), `import` (importing from external source), and `run` (pre-execution check). On `load` context, required-parameter and output-binding-type errors become warnings (allowing the editor to open despite incomplete configuration); on `import` and `run` contexts, the same errors are blocking. The endpoint is read-only and returns the structured `{fatal, errors, warnings}` shape: 200 on clean validation, 422 with the same shape on failure. The request may carry an unsaved workflow and its variables; the endpoint then validates that body under the stored test's identity, with the stored variables when none are sent - the same body a run of an edited canvas executes. A fourth context, `save`, exists at the service layer for the persistence pipeline described below; it is not exposed on this endpoint because clients perform save through `POST /tests` or `PUT /tests/{id}` directly, and the pipeline runs validation internally as part of those requests.

**Unified Persistence Pipeline**: Test creation (`POST /tests`) and test update (`PUT /tests/{id}`) share a single internal pipeline rather than independent inline checks. The pipeline validates the incoming workflow under save semantics, applies automatic fixes for the structural class of errors (auto-fixable graph and reference issues), re-validates, and rejects the request only when user-actionable errors remain. The user-actionable set comprises required-parameter errors (`REQUIRED_PARAM_MISSING`, `REQUIRED_PARAM_EMPTY`), the output-binding-type mismatch error (`INVALID_OUTPUT_BINDING_TYPE`), and any fatal structural error that survived auto-fix. On rejection, the response is 422 with the same structured error envelope used by the validation endpoint. A request with broken edges or missing node identifiers reaches storage in a structurally clean form, and a request that lacks required information is rejected with explicit feedback.

**Repair Reporting**: Automatic repair is not silent. When the pipeline removes a block or a connection to reach a structurally clean graph, the create and update responses carry a repair record alongside the saved test - each entry naming what was removed and why. A clean submission carries an empty list. Validating the saved test afterwards reports no errors, because the stored copy is already clean; the repair record is the only account of what the submission lost, so it travels with the response that performed the removal. The same record reaches recipe authoring over the assistant interface, where the author never sees the canvas and would otherwise have no way to learn that part of the submitted flow did not survive.

**Two-Phase Validation**: Phase 1 checks for fatal structural errors (missing workflow, invalid node array, cycle detection) and fails fast - these are unrecoverable without manual intervention. Phase 2 collects all fixable errors and warnings across the entire workflow, providing comprehensive feedback in a single response.

**Error Structure**: Each validation error includes: `path` (JSON path to the problematic element), `node_id` (affected workflow node), `node_label` (human-readable node name), `element_type` (plugin or control type), `code` (machine-readable error code), `message` (human-readable description), `severity` (error or warning), and `fix_action` (description of the automatic fix available, or null for non-fixable errors).

**Error Code Taxonomy**:

| Category | Error Codes | Description |
|----------|-------------|-------------|
| Structural | MISSING_WORKFLOW, INVALID_NODES, CYCLE_DETECTED | Fatal graph-level issues |
| Node identity | MISSING_NODE_ID, INVALID_NODE_TYPE, MISSING_BLOCK_TYPE, UNKNOWN_BLOCK_TYPE, NODE_TYPE_MISMATCH | Node metadata integrity |
| Node presentation | INVALID_POSITION, MISSING_POSITION, MISSING_LABEL, INVALID_VERSION | Visual and version metadata |
| Edge/graph | DANGLING_EDGE, SELF_LOOP_EDGE, MULTIPLE_EDGES, STALE_TARGET_HANDLE, INVALID_EDGE_TYPE, CROSS_SCOPE_EDGE | Graph topology issues |
| Parameters | REQUIRED_PARAM_MISSING, REQUIRED_PARAM_EMPTY | User-actionable; context-sensitive severity |
| Output bindings | INVALID_OUTPUT_BINDING_TYPE | User-actionable; type-incompatible binding to an existing variable |
| Variables | INVALID_VARIABLE, INVALID_VARIABLE_TYPE, UNDECLARED_VARIABLE, SETTER_TARGETS_NAMESPACE_VARIABLE, BINDING_TARGETS_NAMESPACE_VARIABLE | Variable integrity |
| Numbers | SETTING_TOO_MANY_DIGITS, INVALID_VARIABLE_VALUE, INVALID_COMPARE_VALUE, INVALID_COLUMN_INDEX | User-actionable; a Block setting, a variable value or default, a condition's compare-to value or a table column index breaks the one number rule (REQ-3.1.3.14) |
| Expressions | EXPRESSION_CHANGES_DATA | User-actionable; a field, template marker or variable expression would change data instead of only reading it. No automatic fix: the author rewrites the text and makes the change with a Variable Setter Block |

**Fix Endpoint** (`POST /tests/{id}/fix`): Validates the test, applies automatic fixes for the structural class of errors, saves the corrected test, and returns the fixed test data along with any post-fix warnings. Fix actions include: generating UUIDs for missing IDs, removing invalid nodes, setting default positions, generating labels from element type, filtering dangling/self-loop/duplicate edges, correcting invalid variable names (with reference updates), converting invalid variable types to string, and auto-creating undeclared variables with inferred types, holding null. User-actionable errors (`REQUIRED_PARAM_MISSING`, `REQUIRED_PARAM_EMPTY`, `INVALID_OUTPUT_BINDING_TYPE`) are not auto-fixed and surface in the post-fix warnings; the endpoint persists the structurally cleaned test even when these warnings remain, producing the valid-but-incomplete state that subsequent loads render with persistent alert cues. Refuses to fix fatal errors (returns error response).

**Standalone Workflow Validation** (`POST /workflows/validate`): Accepts a raw workflow payload (nodes and edges) and validates structural integrity without requiring a persisted test entity. Supports the same context parameter and returns the same error structure.

**Clone and History**: Clone operations create copies of existing tests with new names, supporting test derivation workflows; a clone receives a copy of the source's test data. History endpoints provide complete version audit trail with change descriptions. Restore operations create new versions from historical snapshots.

**Execution History**: A test's past runs come from the execution listing filtered to the test (see 4.12.1), enabling trend analysis and regression detection.

**Caching and Performance**: A directory cache mirrors the test storage, and tag and list responses carry a 60-second cache lifetime.

**Error Handling**: Operations enforce consistency through validation and transactional semantics. Naming conflicts return conflict status, deletion with suite references returns conflict with references, missing entities return not found, malformed requests return bad request with validation details, workflow validation failures return detailed graph analysis, and internal errors return server error with correlation identifiers. Optimistic concurrency control prevents conflicting modifications.

#### 4.6.2 Tags Support

##### REQ-9.5.3.1 Test tags aggregation endpoint

Tests support categorization through tags stored within the test metadata. Tags enable organization and filtering of tests by category, feature area, or any user-defined grouping.

**Tags Discovery**: A dedicated endpoint returns all unique tags across all tests, sorted alphabetically. This supports filter dropdowns in the UI enabling tag-based test filtering. The endpoint returns an array of tag strings representing the complete set of tags in use across the test collection.

**Caching**: Tags responses cache briefly with automatic invalidation when test entities are created, updated, or deleted. The cache supports efficient repeated queries for filter dropdown population without re-scanning the entire test collection.

**Response Format**: The endpoint returns a success indicator and the tags array. Empty collections return an empty array, not an error.
