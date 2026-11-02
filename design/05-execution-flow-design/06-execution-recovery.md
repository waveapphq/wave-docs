### 5.3 Reliability Design

#### 5.3.2 Execution Recovery

##### REQ-4.3.2 Test execution recovery

Test execution recovery defines how WAVE handles executions that do not complete normally, preserving accumulated artifacts. Recovery spans user-initiated Stop, backend process interruption, and orphan detection on startup, producing a uniform terminal state from which the user may inspect results.

**Unified Stopped Semantics**: The `stopped` status represents any non-successful early termination regardless of cause. The status applies uniformly to user-initiated Stop via the UI, user-initiated Stop via the API, backend process interruption, and orphan recovery on startup. The status is the same whatever the cause.

**Interruption Detection and Handling**: The system detects interruptions through process termination signals, unhandled exceptions in the execution engine, and external stop commands. The interrupted test's artifacts remain on disk for inspection.

**Stop**: An immediate stop ends every process group a running block started, leaving nothing behind, including a block running inside a Parallel branch, because a Stop cancels every lane (SHRUN-15).

**Orphan Recovery on Startup**: When the backend starts, the storage layer scans executions in `running` or `paused` state. Each such execution is transitioned to `stopped` with `ended_at` set to the current time and a log entry indicating recovery by orphan sweep. Artifacts, logs, and variable snapshots are preserved intact. No automatic resumption is attempted.

**User Control**: Users control execution lifecycle via Stop actions in the UI (Executions list, Workbench tracked execution viewer) and via the execution Stop API. Both paths route through the same service function and produce `stopped` state.
