### 5.3 Reliability Design

#### 5.3.3 Artifact Persistence

##### REQ-4.3.3 Artifact persistence

Artifact persistence ensures that execution results, diagnostic data, and runtime state are reliably captured even during failures and abnormal termination, guaranteeing critical execution information survives for analysis and historical tracking.

**Initialization Validation**: During workflow initialization, the system creates the artifact directory and its parent hierarchy if needed and checks available disk space. Below a fixed 1 GiB it issues a warning and execution proceeds. This upfront check catches configuration issues early without imposing per-write overhead.

**Direct Artifact Writes**: Artifacts are written directly to their final locations within the execution's artifact directory. JSON artifacts are serialized and validated in memory before writing, catching serialization errors before I/O operations. WAVE-generated artifacts are written under their plain logical name and are single-instance per execution (no version suffixes): an execution's report, failure summary, and per-test logs are immutable for that execution and are replaced only by an explicit delete-then-regenerate, never by silent overwrite. Write failures are logged with clear error context but do not halt execution unless critical artifacts cannot be written.

**Failure-Resilient Generation**: The persistence layer continues artifact generation during execution failures. When errors occur, the writer captures the execution identifier and its failures in the failure summary. Critical artifact generation occurs within exception handlers, ensuring diagnostic data survives even when failures prevent normal completion.

**Artifact Organization**: Artifacts are organized within timestamped execution directories. Subdirectories may be used to group artifacts by type or workflow stage as the writing party sees fit. The directory contents are the complete source of artifact metadata - there is no separate manifest or index - keeping every execution directory self-describing and relocatable.

**Concurrent Write Safety**: Multiple components may write artifacts concurrently during parallel execution. The artifact directory structure isolates artifacts by type and execution phase into separate files, minimizing write contention. Individual artifact files are independent with no shared state requiring coordination beyond filesystem guarantees.

**Error Handling Integration**: Error handlers invoke artifact persistence to ensure error context is written before cleanup. Failure-capture artifacts are written synchronously when a run reaches a failed state, so diagnostic data is on disk before execution termination even when normal completion paths are skipped. The failure-capture artifact (error.json) is a single per-execution causal summary: it aggregates the error and fatal log entries that caused tests to fail, correlated to the failing block, rather than duplicating the full per-test log stream.

**Log Artifacts**: On execution completion each test's captured log stream is persisted as a per-test file within the execution's artifact directory under a `logs/<suite>/<test>` layout, making execution traces first-class, browsable artifacts alongside reports and plugin outputs.

**Deletion and Tombstones**: Artifact deletion is a soft operation. Removing an artifact frees its bytes but records a tombstone (logical name, relative path, size, removed-at, removed-by) so the record of what an execution produced is never silently lost. Tombstones are themselves directory entries, keeping the directory self-describing with no separate index, and are surfaced read-only as an audit trail; there is no restore.

**Observability**: All artifact writes are logged at DEBUG level with path, size, and timing information. Write failures are logged at ERROR level with filesystem error details and available space diagnostics. This logging provides visibility into artifact generation and aids troubleshooting when artifacts are missing or incomplete.
