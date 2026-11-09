### 3.2 Backend Components

#### 3.2.1 API Layer Structure
The backend provides a RESTful API with WebSocket support, organized into the following logical components:

- **Workflow Management API**: CRUD operations for workflows and test suites
- **Execution Control API**: Test execution orchestration, debugging, and real-time monitoring
- **Validation API**: Workflow and recipe validation
- **Reporting API**: Artifact generation, report creation, and result aggregation
- **WebSocket Layer**: Real-time execution updates, variable monitoring, and debug events

#### 3.2.2 Core Backend Services

**Workflow Engine**
- Manages workflow composition and dependencies
- Validates workflow integrity
- Manages version control
- Handles block nesting and hierarchy

**Execution Engine**
- Orchestrates test execution flow
- Manages execution state and context
- Handles breakpoints and debugging
- Coordinates parallel execution

**Variable Manager**
- Implements hierarchical scoping
- Manages variable inheritance
- Handles computed variables
- Provides variable watching

**Plugin System**
- Parameter validation
- Return value management
- Error handling and retry logic

**Entity Lifecycle Services**
- CRUD orchestration for tests, suites, and regressions
- Active-execution guard on delete: rejects with `Conflict` when any tracked execution referencing the entity is in a non-terminal state (running or paused), returning the blocking identifiers so the caller can route the user to stop them. For a test, the guard also counts a running or paused suite or regression execution that carries the test, and a workbench session running or paused on it
- Cascade on successful delete: removes the deleted child from every parent that references it (test removed from suites, suite removed from regressions) so no dangling parent-to-child references remain
- Executions are never mutated by entity lifecycle operations. Historical execution records carry their own immutable workflow and name snapshots, keeping the Historical Test Canvas and stored execution metadata valid after the source entity is gone

**Entity Relations Index**
- In-memory reverse-lookup index for parent-child relationships: test -> suites, suite -> regressions
- Provides membership queries consumed by list decoration, validation, and the delete cascade
- Updated in-band on every parent mutation (add, remove, delete, rename); rebuilt lazily on first access after process start
- Rename hooks refresh cached child-visible names so list decoration never renders stale identifiers

**Workbench Administration**
- Administrative inspection and teardown of the active Workbench session: a read endpoint returns the current session's identity and status, a delete endpoint terminates it
- Terminates the underlying task (if any), purges session state, and clears the user-active-session mapping so subsequent operations start fresh
- Serves operational cleanup needs, primarily automated test teardown, without requiring the owning browser client to be alive