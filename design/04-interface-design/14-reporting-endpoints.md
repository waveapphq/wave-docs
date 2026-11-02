### 4.14 Reporting Endpoints

#### 4.14.1 Report Generation & Retrieval

##### REQ-5.2.4.13 Reporting endpoints

Reporting endpoints transform a finished execution's append-only record into human-readable reports for analysis, sharing, and archival. Reports are artifacts: generation writes them into the source execution's artifact directory through the artifact storage layer, where they are listed, streamed, and deleted by the standard artifact endpoints. There is no separate report store and no report entity; the execution record remains the single source of truth and a report is a presentation of it.

**Generation Model**: Report generation is on demand and per format. A generation request names the desired format; when a report artifact of that format already exists for the execution, the request returns it immediately and nothing is generated or written. Only when the format is absent does the request start an asynchronous job, keyed by the execution identifier and format, that reads the execution record, builds the canonical report model, renders the requested format, and writes it as an unversioned `report.html` or `report.json` artifact. Requesting generation while a job for the same execution and format is in flight returns the existing job unchanged. Regeneration is an explicit act: the report artifact is deleted through the standard artifact deletion endpoint and then requested again; the fresh output is written under the same name.

**Progress Contract**: Job state exposes the requested format, a lifecycle status (pending, running, completed, failed), and determinate progress expressed as runs processed out of total runs, enabling clients to render meaningful progress for large suite and regression executions while small test executions complete near-instantly. Completed jobs reference the produced report artifact by artifact identifier. Failed jobs carry the failure reason. Job state lives in process memory only: it does not survive a backend restart, and a client that finds no job for an execution simply requests the report again - which returns the persisted artifact when it exists. The report artifacts themselves persist independently of job state.

**Format Support**: Every format is an independent rendering of the same canonical model, generated only when requested and only when absent. The HTML report is a fully self-contained document - inline styles, inline behavior, no external references - suitable for browser viewing, archival, and sharing outside WAVE. The JSON report carries the canonical model in machine-readable form for CI/CD integration and custom analysis. Each format is written uncompressed so the artifact content endpoint streams it directly viewable.

**Eligibility**: Reports are generated only for executions in a terminal state. Generation requests for running or paused executions are rejected with a conflict. Generation for executions that predate artifact directory allocation is rejected with a conflict.

**Listing Model**: A report listing endpoint presents the execution-centric report inventory: one row per execution eligible for reporting (every execution not currently running or paused - exactly the set the generation endpoint accepts), joined with the latest report artifact metadata per format. Each row carries the execution summary (identity, entity type, status, trigger attribution, start/end, duration) and, per format, either the latest report artifact's identity, version, size, and generation time, or nothing when that format has not been produced. The listing supports the standard list contract - text search, status / entity-type / trigger filters, date range, sorting, and pagination - mirroring the execution listing's conventions, and validates filter values strictly, rejecting unknown values by naming the valid ones. Report metadata is resolved from the artifact directories of only the returned page, keeping the join cost proportional to the page size rather than the execution count.

**Workbench Session Reports**: The current workbench session's finished run is reportable on demand through a session-scoped endpoint with the same per-format, exists-first semantics. Session report generation is synchronous - the run's state and logs are in memory - and writes the requested format into the session's artifact directory, where the session artifact endpoints list and stream it and the session lifecycle wipes it. A session report counts as existing only when it was produced during the current run: an artifact inherited from a predecessor session is treated as stale and a fresh version is generated. Only the active session is reportable; sessions viewing a tracked execution are redirected to the execution-scoped endpoint, which owns that execution's record.

**Error Handling**: Invalid execution identifiers return bad request with UUID format details; unknown report formats return bad request naming the valid formats. Unknown executions return not found. Generation requests for non-terminal executions return conflict naming the current state. Status retrieval for an execution without a job in the current process lifetime returns not found, signaling the client to request the report again. Session report requests without an active session return not found; requests while the session run is active return conflict.

**Security Considerations**: Reports contain execution data including variable values. Variables marked as secrets are masked at report build time using the same masking applied by execution data endpoints, and log content is persisted post-masking by the execution engine. HTML report responses carry security headers preventing cross-site scripting, and all dynamic content is escaped during rendering.

#### 4.14.2 Report Content

The canonical report model carries execution identity and outcome, a summary, a timeline, error details, and the merged log history. Every output format presents the same model.

##### REQ-3.7.2.1 HTML report generation

The HTML report is a single self-contained document rendered from the canonical model. All styling and interactive behavior are inlined; the document references no external resources and renders identically from the artifact endpoint, a saved file, or an email attachment. The document presents the execution header, pass/fail summary, timeline, error details, and merged logs in one scrollable page styled consistently with the WAVE design language, including an embedded light/dark theme switch that defaults to the reader's system preference.

##### REQ-3.7.2.2 Pass/fail summary

The summary section presents the execution's overall status and duration together with per-test outcome counts: total, passed, failed, skipped, stopped, excluded, and pending. Counts aggregate the execution's runs, matching the counts shown by the execution detail surface.

For test executions and workbench session runs the summary counts steps instead of runs: a total followed by the outcomes in severity order - passed, failed, warning, skipped, stopped. Every step that ran falls under exactly one outcome, and the total is their sum, so the total, the outcome counts and the step list are three views of one set of steps and can never disagree. Steps the run never reached are counted nowhere, consistent with their absence from the step list; they survive only in the machine-readable model. A step still marked as in flight when the report is written counts as stopped, matching how the engine settles a step interrupted by a stop or a timeout.

##### REQ-3.7.2.3 Execution timeline

The timeline presents when each unit of work ran, as an ordered table plus a collapsible full-width chart of time-positioned bars referenced by order number. For suite executions the rows are test runs in dispatch order with status, start, and duration. For regression executions the rows are grouped per suite, and the chart carries one sub-section per suite with its own time axis. For test executions and workbench session runs the rows are blocks, with names resolved from the workflow snapshot and nested blocks carrying their container path. Timeline granularity is structural: one row per workflow node regardless of runtime repetition - a looping block's bar spans its first start to its last end across all iterations, and parallel branches appear as overlapping bars. Iteration-level detail belongs to the log content, not the timeline.

A test timeline lists only the blocks the run recorded an outcome for. Deliberately skipped and disabled blocks are recorded outcomes and stay listed, because the report is the record of what was and was not checked. Blocks the run never reached carry no outcome, are not listed, and are counted nowhere in the summary. Row numbering is contiguous over the rows on show, and the chart presents exactly the rows the table presents. The canonical model keeps a row for every block of the workflow whether or not the run reached it, so machine consumers retain the full record - the omission is a presentation rule, applied identically by the generated document and the in-app viewer.

##### REQ-3.7.2.4 Error details with context

Each failure surfaces as an error entry carrying its context: the failing scope path (suite, test, and block as applicable to the entity type), the error severity, the failure timestamp, and the error message. The full surrounding narrative lives in the merged log section of the same document; error entries do not duplicate log content. Executions without failures present an explicit no-errors state.

##### REQ-3.7.2.5 Custom logging from tests/blocks

Log entries emitted by test authors through the logging plugin and block-level loggers appear in report content with their original timestamp, level, source attribution, and block context, exactly as captured in the execution record.

##### REQ-3.7.2.6 Log levels DEBUG/INFO/WARN/ERROR/FATAL

Report log content preserves the five-level severity model. Every rendered log entry carries its level, and level styling follows the same visual vocabulary as the execution console.

##### REQ-3.7.2.7 Merge custom and framework logs in reports

Report log content is the merged chronological stream of framework entries (system source) and custom entries (plugin and user sources) across all scopes of the execution - regression and suite scope events interleaved with per-test console output - in the order they occurred. Logs render as a columnar table (time, level, source, message): the source column wraps at a bounded width and the message wraps in the remaining width, and the report renders every message exactly as it was recorded, shortening nothing of its own. A message is bounded once, where the log entry is created, and any bound that fired is stated in the message text itself, so a report shows either the whole message or a message that says what was left out.

##### REQ-3.7.2.8 Filter logs by level in report viewer

The HTML report embeds its own log controls: a level threshold filter that shows entries at or above the selected severity with a live count of visible entries, and a column selector toggling the time, level, and source columns. Both operate entirely inside the document with no server dependency, so they work wherever the report is opened.
