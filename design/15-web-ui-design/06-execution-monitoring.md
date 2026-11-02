### 15.6 Execution Monitoring

WAVE supports two distinct execution contexts with different interaction patterns:

| Context | Entry Point | Environment Selection | Creates Execution Entity |
|---------|-------------|----------------------|--------------------------|
| Workbench | Canvas toolbar Run button | Persistent toolbar selector | No (ephemeral) |
| Entity Lists | Row action or bulk action | Modal popup | Yes |

**Workbench runs are ephemeral** - they provide live feedback during test development but do not create Execution entities. This keeps the Executions list clean of development noise, as test engineers may run dozens of iterations while building a workflow.

**Entity List runs are tracked** - they create Execution entities for historical reference, CI/CD integration, and formal test reporting.

#### 15.6.0 Workbench Execution

##### REQ-3.2.1.1 Normal execution (run to completion)

Workbench execution enables users to run workflows directly from the canvas during test development. This is the primary execution mode for test engineers iterating on workflow design, where rapid feedback cycles require minimal friction between editing and running.

**Persistent Environment Selector**: The Workbench toolbar includes a persistent environment selector that maintains the user's environment choice across run actions. This eliminates the need to select an environment for every execution, supporting rapid iteration during test development.

| Selector State | Run Button Behavior |
|----------------|---------------------|
| Environment selected | Run executes immediately with selected environment |
| No environment selected | Run opens environment selection popup |

**Environment Selector Popup**: The selector presents a searchable interface organized into sections:

| Section | Content | Purpose |
|---------|---------|---------|
| Recent | Last 5 used environments | Quick access to frequently used contexts |
| All Environments | Complete environment list | Full selection when recent doesn't suffice |

The popup includes search filtering across all environments. A "Remember selection" checkbox allows users to persist their choice for future sessions.

**Run Without Pre-selected Environment**: When the user clicks Run without having selected an environment:

1. Environment selection popup opens (same as toolbar selector)
2. User selects environment from recent or all environments
3. "Remember selection" option available to set as default
4. Clicking Run in popup initiates execution
5. Selected environment populates the toolbar selector for subsequent runs

This flow ensures first-time users are guided to select an environment while experienced users with a pre-selected environment experience instant execution.

**Execution Scope**: The Run control uses a split button design supporting scope selection:

| Click Target | Behavior |
|--------------|----------|
| Primary area | Run the currently-opened scope (sub-block when drilled in, the test otherwise) |
| Dropdown arrow | Shows scope options: Run Scope, Run All |

"Run Scope" targets only the currently-opened scope (a sub-block when the user has drilled in, the test otherwise); "Run All" targets the outermost parent test from its Start node regardless of where the user is viewing. Per-element execution (single-element runs and partial-graph "from Here" runs) is offered on the right-click context menu, not on the split button - see § 15.6.5 below.

**Debug Action**: The Debug control follows the identical split button pattern with corresponding scope options (Debug Scope, Debug All). Debug mode enables breakpoint support and step execution controls.

**Execution Request**: When execution triggers, the Workbench submits:

| Parameter | Source | Purpose |
|-----------|--------|---------|
| Entity type | Always "test" | Workbench operates on test workflows |
| Entity ID | Current test ID | Target test to execute |
| Environment ID | Toolbar selector | Runtime context for profile resolution |
| Debug mode | Debug button used | Enables breakpoint support |

**Canvas State Transitions**: After triggering execution, the canvas transitions through states:

| State | Toolbar Controls | Canvas Behavior |
|-------|------------------|-----------------|
| Idle | Environment, Debug, Run | Normal editing mode |
| Running | Stop, Focus Current | Blocks show execution progress |
| Paused | Step Over, Step Into, Continue, Stop | Breakpoint reached |
| Passed/Failed/Stopped | Environment, Debug, Run, Reset | Results visible |

**Reset Action**: The Reset link in the breadcrumb bar clears all execution state visuals, returning blocks to idle appearance. This does not affect workflow structure or the selected environment.

**Ephemeral Execution**: Workbench runs do not create Execution entities. The execution state is maintained by the backend in-memory for the duration of the session:

| Aspect | Workbench Behavior |
|--------|-------------------|
| Backend State | Maintained in-memory per user (one active session per user) |
| Persistence | No database persistence - lost on backend restart |
| Recovery | Frontend queries backend on mount to recover state |
| Expiration | Terminal states (passed/failed/stopped/skipped) expire after 1 hour |
| History | Not recorded in Executions list |
| Logs | Displayed in Console panel during run, not persisted |
| Variables | Shown in Variable Manager during run, not persisted |

**Session Recovery**: When the frontend mounts the Workbench page (after navigation, refresh, or browser reopen), it queries the backend for the current user's active session:

| Scenario | Frontend Behavior |
|----------|-------------------|
| No active session | Show idle state, ready for new run |
| Session running/paused | Hydrate UI state, subscribe to WebSocket for updates |
| Session passed/failed/stopped/skipped | Show terminal state with results, Reset available |

This design enables state recovery across:
- Page refresh
- Navigation away and back
- Browser crash and reopen (within session expiration window)
- Opening new tabs (all tabs see the same session state)

**Single Session per User**: Only one workbench session can be active per user at a time. Starting a new run while one is active requires stopping the current run first. This simplifies state management and prevents confusion about which execution is being monitored.

This ephemeral model supports the rapid iteration workflow where test engineers may run dozens of times while developing a single test. Creating tracked executions for each would clutter the system with development noise.

To create a tracked Execution for reporting or CI/CD purposes, users run from the Entity Lists (Tests, Suites, Regressions pages).

#### 15.6.0.2 Entity List and Detail Execution

Running tests, suites, or regressions creates tracked Execution entities for historical reference, CI/CD integration, and formal test reporting. This flow uses a modal-based approach distinct from the ephemeral Workbench execution.

**Run Entry Points**: A Run affordance exists on both the list and detail surfaces for every entity type. On list pages it appears as a row action. On detail pages it appears as a primary-styled Run button in the DetailHeader, rendered before Export and Delete. Both entry points open the same RunDialog component and flow through the same tracked-execution path.

**Run Flow**:

| Step | User Action | System Response |
|------|-------------|-----------------|
| 1 | Click Run on a row or in a detail header | RunDialog opens |
| 2 | Select environment; for suites and regressions, adjust test subset if desired | Selection validated |
| 3 | Click Run in dialog | Tracked Execution created via POST /executions |
| 4 | - | Navigate to Execution detail view (/executions/{id}) |

**RunDialog Variants**: A shared modal component used by all entity types. The dialog always presents environment selection using the same searchable interface as the Workbench environment selector (recent environments, all environments, search filtering). For suites and regressions, the dialog additionally presents a test-subset panel:

| Entity | Subset Panel |
|--------|--------------|
| Test | Not shown - tests are atomic |
| Suite | Flat list of the suite's tests with per-test checkboxes and a Select-All control. Default: all selected |
| Regression | Suites rendered as collapsible groups, each containing its tests. Selection is per-(suite, test) pair so the same test appearing in multiple suites of the regression can be independently included or excluded per context. Default: all selected |

The Run button remains disabled until an environment is chosen and at least one test is selected.

**Regression Duplicate-Test Badge**: Within a regression Run dialog, a test identifier that appears in 2+ suites of the regression is rendered with a duplicate badge. Hovering the badge opens a popover listing the suites of the regression that reference the test, helping the user understand why the same test will run multiple times within a single execution if they leave all instances selected. The same badge appears in the regression Execution Detail tests list for consistency.

**Execution Request**: When the user confirms the run, the system submits a tracked execution request:

| Parameter | Source | Purpose |
|-----------|--------|---------|
| Entity type | "test", "suite", or "regression" | Determines execution scope |
| Entity ID | Target row's or detail page's entity ID | Target entity to execute |
| Environment ID | RunDialog selector (required) | Runtime context for profile resolution |
| Selected tests | Subset panel state (suites and regressions only) | Test IDs (suite) or `(suite_id, test_id)` pairs (regression) to run; omitted for tests; absent for a run of the full entity |

**Subset and Excluded Status**: When a subset is supplied, only the listed tests run. Every other test declared by the entity is recorded in the execution timeline with status `excluded` (zero duration, no logs, no variable capture). `excluded` is distinct from `skipped`: `excluded` means the user chose not to include the test at dispatch time; `skipped` means the test was eligible to run but was bypassed mid-run (by a halt with scope=suite or scope=regression fanning out to the remaining entries at that scope, by a user Stop fanning out to remaining not-yet-started entries, or by a halt with scope=test and status=skipped applied to a single test). The execution's shape still reflects the entity's full definition; the runtime cost is confined to the selected tests. The Executions list and Execution Detail tests list filter out both `excluded` and `skipped` entries by default; the Status filter offers both values as independent toggles so the user can surface either class on demand.

**Sequential Execution Semantics**: Running a suite executes every non-excluded test of that suite in the Suite's declared order, one test at a time. Running a regression executes every non-excluded suite of that regression in the Regression's declared order; each suite follows the same test-by-test ordering rule. A suite with zero selected tests in a regression is fully `excluded` and contributes no execution cost.

| Event | Timeline Effect | Parent Rollup |
|-------|-----------------|---------------|
| Test passes | Entry status `passed`, duration recorded | Continues with the next entry |
| Test fails (regular failure) | Entry status `failed`, duration recorded | Continues with the next entry; parent rollup ultimately `failed` if any entry failed |
| User Stop | In-flight entry status `stopped`; remaining not-yet-started entries become `skipped` | Parent rollup `stopped` |
| Halt, scope=test | Entry status mirrors the halt's status (`passed`, `failed`, or `skipped`) | Continues with the next entry |
| Halt, scope=suite | Current entry status mirrors the halt's status; remaining tests in the current suite become `skipped` | Suite rollup equals the halt's status; if inside a regression, the next suite still runs |
| Halt, scope=regression | Current entry status mirrors the halt's status; remaining tests in the current suite AND every remaining suite's tests become `skipped` | Regression rollup equals the halt's status |

Each test starts with a fresh variable scope composed of system, suite, and (when applicable) regression variables resolved at runtime. Runtime values produced by an earlier test in the run do NOT propagate to later tests; dependencies between tests must be expressed through persisted state (environment, profile, external systems), not through in-memory variable flow.

Progressive per-test transitions are delivered via the same live-update path that drives the Execution Detail view (see 15.6.4 "Live Updates"). The RunDialog and websocket subscription are not modified by the sequential semantics.

**Tracked Execution**: Unlike ephemeral Workbench runs, entity list or detail executions create persistent Execution entities that appear in the Executions list, support filtering and history, and provide formal test reporting. The selected subset is persisted on the execution record so re-run can pre-populate the RunDialog with the exact prior selection.

**Re-run Pre-population**: The Re-run action on the Execution Detail toolbar opens the same RunDialog pre-populated with the original environment and the original selected subset. The user can customize any field (environment, which tests to include) before confirming. Confirming creates a new tracked execution and navigates to its detail view.

**Executions List Filtering**: The Executions list provides these filter dimensions:

| Filter | Type | Behavior |
|--------|------|----------|
| Search | Text | Case-insensitive match against execution name and triggered-by user |
| Status | Multi-select | Running, Passed, Failed, Stopped, Skipped (OR within group). Skipped is off by default |
| Type | Multi-select | Test, Suite, Regression (OR within group) |
| User | Multi-select, dynamic | Populated from unique triggered-by values across all executions (OR within group) |
| Date range | Preset popover + Custom | A filter pill opens a popover with a single-select list of presets: All time (default), Today, Last 7 days, Last 30 days, Last 90 days, Custom range. Picking a preset applies immediately and closes the popover. Picking Custom range replaces the preset list with a single-month range calendar plus two time inputs (From and To) stacked beneath it, and an Apply / Cancel footer. Apply is disabled until both range ends are selected; Cancel (and click-outside) returns to the preset list without applying. Default times are 00:00 for From and 23:59 for To; users may narrow to a sub-day window. Applied values map to the backend `from_date` / `to_date` query parameters as local-timezone ISO timestamps. Preset values resolve to concrete `from_date` / `to_date` at request time, anchored to the current instant. |

Filters compose with AND between groups: selecting Status=Running + User=dev + Date range=Last 7 days shows only running executions triggered by dev in the last 7 days.

**Single Entity Runs**: Row-action runs and detail-page Run buttons execute a single entity (test, suite, or regression). Multi-entity bulk runs from list pages are not part of this design.

**Post-Run Navigation**: After starting the execution, the user is navigated to the Execution detail view (/executions/{id}) where they can observe progress, logs, and results in real-time.

#### 15.6.0.3 Tracked Execution Viewer in Workbench

A tracked execution uses the same workflow engine as Workbench. The execution identifier doubles as the workbench session identifier, so a user may navigate to the Workbench at any time during a tracked run to watch live progress on the canvas without additional setup.

**Entry Points**: A user reaches the viewer by navigating to the Workbench while a tracked execution is running. The Workbench session-recovery flow loads the in-progress session automatically; no explicit "watch this execution" control is required.

**Viewer Mode Presentation**: When the loaded workbench session is tied to a tracked execution, the Workbench breadcrumb bar renders a Tracked Execution prefix before the workflow path. The prefix reads `<Entity-Type> Execution: <Entity-Name>` where entity type is one of `Test`, `Suite`, or `Regression` and entity name is the name of the entity being executed. The name portion is rendered as a link that navigates to the execution detail view for that execution. Full breadcrumb with the prefix: `Test Execution: Smoke Test > Smoke Test` (the trailing token is the regular workflow path, clickable per breadcrumb conventions). This gives the user the context (what kind of run they are observing and which entity) and a one-click path to the full execution record with its progress, logs, and timeline, without a dedicated toolbar row. Canvas behavior remains read-only during any running session, so no additional locking is required on top of this prefix.

**Control Unification**: The Workbench Stop control acts on the tracked execution when the loaded session represents one. Stopping routes through the execution lifecycle, ensuring the execution record transitions to the `stopped` state via the same pipeline as a Stop initiated from the Executions list. Ephemeral workbench runs continue to route Stop through the workbench session endpoint.

**Stop Confirmation Policy**: Stop is confirmation-gated whenever the target is a tracked execution - whether the Stop is invoked from this Workbench tracked viewer, from the Executions list, or from the Execution Detail view. Ephemeral workbench runs do not require confirmation; the rapid-iteration workflow would be degraded by a dialog for every run.

**Multi-Tab Observation**: A tracked execution may be viewed by multiple browser tabs concurrently. Each tab subscribes independently to the execution's WebSocket topic and renders live progress; only one execution runs at a time on the server per user, regardless of viewer count. This supports the common workflow of running a test in one tab while continuing exploratory work in another.

**Terminal State**: When the engine reaches a terminal state (passed, failed, stopped), every Workbench tab currently viewing the tracked execution clears its canvas to an empty Draft Workflow state and redirects the user to the execution detail view at `/executions/{execution_id}`. The stale post-run canvas is not preserved because the detail view provides the authoritative record of progress, logs, timeline, and results. The Executions list reflects the terminal status simultaneously. This behavior is specific to tracked executions; ephemeral Workbench runs continue to leave the canvas in its terminal state with a Reset control for the user to invoke when ready.

**Detail View vs Viewer**: The execution detail view provides static inspection of a completed run. The Workbench viewer provides the live, canvas-centric perspective during a running execution. The two are complementary; the Workbench is not a replacement for the detail view.

#### 15.6.1 Real-time Execution Visualization

The canvas provides live visual feedback during workflow execution, enabling test engineers to observe progress, identify failures, and understand execution flow without switching contexts.

##### REQ-3.2.2.15 Live flow visualization during execution

The canvas updates in real-time as execution progresses through workflow blocks. This visualization leverages the WebSocket infrastructure to push execution state changes to the frontend.

**Execution Event Flow**:

| Event | Source | Canvas Response |
|-------|--------|-----------------|
| Block Started | Backend execution engine | Highlight block as running, update breadcrumb status |
| Block Completed | Backend execution engine | Mark block as passed/failed, advance visual indicator |
| Execution Paused | Breakpoint, pause on first block, or pause on errors | Show paused state, enable debug controls |
| Execution Completed | All blocks finished | Show final status, enable Reset action |

**State Synchronization**: The execution store maintains the current state of each block in the workflow. Components subscribe to this store and re-render when states change. The store reconciles WebSocket events to ensure consistent state even with network latency or message reordering.

**Session Recovery**: When a user returns to an in-progress execution (page refresh, navigation), the canvas queries the backend for current execution state and hydrates the visual representation accordingly. Running executions resume visual updates via WebSocket reconnection.

##### REQ-3.2.2.16 Highlight currently executing blocks

Workflow blocks display visual indicators for their execution state, providing immediate recognition of progress and outcomes.

**Visual State Mapping**:

| Execution State | Visual Treatment | Purpose |
|-----------------|------------------|---------|
| Pending | Default styling, muted border | Block not yet reached |
| Running | Blue border with pulse animation | Currently executing |
| Passed | Green border with checkmark icon | Completed successfully |
| Warning | Amber border with alert-triangle icon | Assertion failed (informational) |
| Failed | Red border with X icon | Execution error occurred |
| Skipped | Gray styling, muted appearance | Conditionally bypassed |
| Stopped | Amber border | Execution stopped (user Stop or backend restart) |

**Skipped Paints Live**: Every block a run marks skipped paints Skipped on the live canvas at the moment the engine marks it: the rest of a sub-flow that a failure, a Break or a Continue cut off; the exit of a container that failed or that a Break or Continue aimed at an outer loop left; the catch and finally paths a Try/Catch did not run; the cases and default a Switch/Case did not run; the blocks a Halt left behind; and the blocks inside a disabled or skipped container. A canvas restored from the stored block states after a page reload therefore shows the same result the live canvas showed. Blocks shown muted because they are disabled, marked Skip or unreachable keep their muted look and show no execution status.

**Animation Characteristics**: The running state uses a pulsing border animation to draw attention to the actively executing block. State transitions employ smooth color fades to avoid jarring visual changes. Animation durations follow the design system specifications for consistent feel.

**Breadcrumb Status Integration**: The breadcrumb bar displays the overall execution status alongside the workflow path. Status indicators match the canonical definition in Section 15.4.0.2:

| State | Display | Color | Additional Elements |
|-------|---------|-------|---------------------|
| Idle | "Idle" | Muted gray | None |
| Running | "● Running" | Blue | Pulsing dot animation |
| Paused | "● Paused" | Amber | Static dot |
| Passed | "● Passed" | Green | "Reset" action link |
| Failed | "● Failed" | Red | "Reset" action link |
| Stopped | "Stopped" | Muted gray | "Reset" action link |
| Skipped | "Skipped" | Amber | "Reset" action link |

**Reset Behavior**: The Reset action clears all execution state visuals, returning all blocks to their pending appearance, and ends the Workbench session with the state ad-hoc runs start from (variable values, block states, open SSH sessions and every Function instance's last runtime scope). This action is available only when execution has reached a terminal state (passed, failed, stopped, skipped) and does not affect the workflow structure.

##### REQ-3.2.2.17 Real-time dashboard with metrics

The Dashboard's summary metrics, Recent Activity, and Recent Executions surfaces keep themselves current while a user is looking at the page. State transitions in the backend - an execution starting, progressing, or reaching a terminal status - become visible on the Dashboard without any explicit user action.

**Refresh Mechanism**: The Dashboard uses interval polling of its aggregated summary endpoint. WebSocket push is intentionally not used here: the Dashboard is a glance surface that aggregates system-wide state, not a per-session live view, and binding it to engine-emitted events would couple the execution engine's step latency to the number of active Dashboard viewers. Polling keeps the read path fully decoupled from the execution path, is naturally resilient to network blips (the next tick recovers), and matches the coarse granularity users actually need on a summary page.

**Refresh Cadence**: The polling interval is aligned with the summary endpoint's server-side cache window, so each tick either returns fresh data or is served from cache without triggering recomputation. This keeps backend load independent of how many Dashboard clients are connected. The interval is deliberately coarser than the Workbench canvas refresh rate (which paints per-block transitions); a summary card flipping from "running" to "passed" within a few seconds of the underlying state change is indistinguishable to users from an instantaneous update.

**Request Serialization**: Ticks are strictly serial. The Dashboard never issues a second request while the previous one is still in flight, so a stalled or slow backend cannot cause a pile-up of concurrent in-flight requests on the client or at the server. The cadence is measured between request *starts*, not between response and next request: if a response takes 2 seconds, the next request is issued 3 seconds later; if it takes 5 seconds or more, the next request is issued immediately after the previous one completes. This keeps the effective tick-to-tick cadence stable at the poll-interval regardless of response latency, avoiding both starvation (slow responses stretching time between refreshes) and overload (fast responses firing too often).

**Request Timeout**: Each tick is bounded by the same poll interval: if a response has not been received by the next scheduled tick start, the in-flight request is aborted. This guarantees that a single stalled request cannot hold up refresh indefinitely - the cycle always progresses. Because requests are serial, the abort is also what unblocks the next tick.

**Refresh Scope**: All three Dashboard data surfaces - summary metric cards, Recent Activity, and Recent Executions - refresh together on every tick. They share a single aggregated response from the summary endpoint, so there is no cost to refreshing them as one unit, and keeping them coherent (all cards reflect the same point in time) is a requirement for the page to read correctly.

**Visibility-Based Pausing**: Polling pauses while the Dashboard tab is hidden and resumes immediately when the tab becomes visible again. A hidden tab that continues to poll wastes client bandwidth and server request budget for a view nobody is looking at. On becoming visible, an immediate refresh runs so the user sees a current snapshot rather than a snapshot from when they last looked at the tab.

**Gating**: Polling is not gated on whether any executions are currently running. A run started from another tab or by another user on a shared deployment must appear on the Dashboard without the current viewer having done anything; gating on locally-observed running state would miss these cases.

**Live-Refresh Indicator**: The Dashboard page header displays a small state-only status pill in place of a manual Refresh control. The pill conveys one of three qualitative states through a short fixed label and a status dot; it does not tick, count up, or otherwise animate its label, and it does not expose a relative-time readout on hover. On a healthy 5-second poll a running "Updated N ago" counter would produce a uninformative looping countdown that adds visual noise without telling the user anything they cannot already see from the live dot, and it would pull attention away from the actual Dashboard content. Users therefore learn at a glance whether the page is live, paused, or struggling, and nothing more - which is exactly what a summary surface needs. Users do not need a manual Refresh button because the data is always moving on its own, and the pill makes the auto-refresh behavior self-evident without paying any attention cost on the healthy path.

| Indicator State | Dot | Label | Meaning |
|-----------------|-----|-------|---------|
| Live | Pulsing | "Live" | Tab visible, last tick succeeded |
| Paused | Static, muted | "Paused" | Tab hidden, polling suspended |
| Reconnecting | Static, amber | "Connection issue - retrying" | Tab visible but recent ticks are failing; last good data is preserved |

**Error Semantics**: A failed tick does not degrade the Dashboard into a full error state. The previously-rendered data is retained and only the header pill transitions to the Reconnecting state, so the user keeps the last good view of the system. The pill flips to Reconnecting in two situations: (a) immediately when a tick fails (network error, HTTP error, or the request's own timeout), and (b) pre-emptively, mid-tick, via a watchdog that fires at the poll-interval mark while a request is still in flight - this is what makes a hung backend visible to the user without waiting for the underlying abort to surface as an error. A subsequent successful tick returns the pill to Live. The page only enters the hard error state when the very first page-load fetch fails outright, matching user expectations that a transient network blip should not wipe a screen full of data.

**Session Recovery**: A full navigation away from and back to the Dashboard performs the standard initial fetch, then resumes the normal polling cadence. There is no separate "session recovery" path because the Dashboard is stateless across visits; every mount is a fresh read of aggregate state.

##### REQ-3.2.2.18 Live variable values display

The Variable Manager panel displays runtime variable values during workflow execution. Values update in real-time as blocks execute and modify variables, providing immediate visibility into workflow state without requiring log inspection or manual refresh.

**Variable Display Structure**: Each variable renders as a compact two-row card:

| Row | Left Content | Right Content |
|-----|-------------|---------------|
| Row 1 | Variable name (mono, truncated) | Action cluster: eye, expand, edit, delete |
| Row 2 | Value text (type-colored, truncated) | Secondary flag badges in this order (`Unused`, `IN` or `OUT`, `{x}` or `{{ }}`, `SECRET`) followed by the type badge. The type badge is always the rightmost element on the row. |

**Eye Toggle**: The eye icon controls which value the row shows. ON (default) shows the runtime value; OFF shows the definition value rendered italic and dimmed. For literals the runtime value is the current/last-known value and the definition is the initial value. For expressions the runtime value is the evaluated result and the definition is the expression text itself. When an expression has no runtime value yet (it has not been referenced during execution), the runtime view shows a muted `<not evaluated yet>` placeholder. The eye is disabled on secret rows; secrets are always rendered as `********` and have no runtime/definition distinction.

**Truncation and Overflow**: The panel uses CSS truncation to handle long variable names and values at narrow widths. The ResizablePanel inner wrapper requires `min-width: 0` to allow the flex item to shrink below its content's intrinsic width - without this, `white-space: nowrap` text from the `truncate` utility prevents the panel from narrowing, pushing all content (search bar, action buttons) off-screen. Both variable names and values show the full text in a native browser tooltip (`title` attribute) on hover when truncated.

**Full-Value Live Inspection**: The per-variable inspect modal (see REQ-3.1.3.11) is the untruncated live surface: its Current mode renders the complete runtime value and re-renders in real time as variable-change events arrive, both during live execution and from the end-of-run snapshot in the history workbench. The WebSocket push carries full values for scalar types, so the modal displays scalar runtime values without any additional fetch.

For complex types (array, map, table) the stream carries only count summaries, so full values come from a read-only workbench session variables resource that exposes the live session's current variable values, filtered to the requested names and with secret values masked. An open inspect modal fetches the full value when it opens and re-fetches whenever a change summary for its variable arrives - the summary acts as an invalidation signal. This keeps the refresh event-driven with no polling loop; the per-variable stream throttle naturally bounds the fetch rate. Table values are served in the canonical column-schema shape (named, typed columns plus row arrays) so an empty table still carries its column structure; the table body also accepts the row-object list shape used by persisted definitions. The history workbench needs no fetch at all - its end-of-run snapshot already contains full values - and an idle workbench renders the persisted definition, which is also already full.

**Value Formatting**: Values display with type-appropriate formatting in the list view:

| Type | List Display | Truncation |
|------|--------------|------------|
| String | Quoted text | 100 chars, ellipsis |
| Integer | Full value | None (rarely long) |
| Float | Full value | None (rarely long) |
| Boolean | `true` / `false` badge | None |
| Array | `[n items]` summary | Count only |
| Map | `{n keys}` summary | Count only |
| Table | `5×3 table` dimensions | Dims only |
| Secret | `********` with lock icon | Fixed mask |

**Type Colors**: Value text and the type badge use the type's color. Expression-typed value text overrides to cyan; secret value text overrides to slate `********`.

The type badge is one shared pill wherever a variable's type is shown - the Variable Manager rows, the inspect panel, the variable pickers in element configuration, and the condition rows of the assertion builder. A kind that is not one of the variable types (a namespace member, for instance) shows the same pill in a neutral tone rather than borrowing another type's color.

| Type | Color |
|------|--------|
| string | sky `#7dd3fc` |
| integer | violet `#c084fc` |
| float | pink `#f472b6` |
| boolean | orange `#fb923c` |
| array | amber `#fcd34d` |
| map | emerald `#34d399` |
| table | blue `#3b82f6` |

**Flag Colors**: `OUT` indigo `#818cf8`, `SECRET` red `#f87171`, `{x}` (expression marker) cyan `#22d3ee`, `Unused` amber `#fbbf24` (VMGR-105).

A variable counts as used when a Block names it in an expression row, or inside a `{{ }}` marker in a template row; the literal text around a marker does not name anything.

#### 15.6.2 Variable Manager Panel

The Variable Manager panel is the single surface for browsing, inspecting, editing and creating workflow variables during development and execution. The panel presents a flat, user-sorted list. Per-row controls let the user toggle between the runtime and definition views of each value, watch the value in the variable's inspect modal, open the variable's window to edit it, or remove the variable.

##### REQ-3.1.3.11 Variable Manager UI panel

The Variable Manager panel appears on the right edge of the canvas, sharing the same collapse and resize behavior as the other canvas panels. The type-aware value bodies it uses to view and edit a variable are the single shared editing surface used across every variable and parameter surface in the application (component-specification Variable Type Editors, § 15.15.5) - the Variable Setter, profile parameters, plugin parameters, and the Suite / Regression / System variable tables all open the same bodies.

**Panel Dimensions**: The panel follows the standard collapsible pattern with user resizability:

| State | Width | Content |
|-------|-------|---------|
| Collapsed | 64px | Icon with vertical label (matches palette style) |
| Expanded (default) | 320px | Full variable manager surface |
| Expanded (resizable) | 240px-60% | User-adjustable via resize handle |

**List Performance**: The variable list uses virtual scrolling for performance:

| Threshold | Behavior |
|-----------|----------|
| 50 or fewer variables | Standard DOM rendering |
| more than 50 variables | Virtual scroll (only visible rows in DOM) |

**Panel Structure**: Top to bottom:

| Region | Content |
|--------|---------|
| Header | Title `VARIABLE MANAGER` (uppercase, left) and `+ Add` button (right) |
| Toolbar | Filter input (left, ~60% width) and sort dropdown (right, ~40% width) |
| List | Flat list of variable rows, no section grouping |

**Filter Input**: A search input filters the list by variable name. Filtering applies incrementally as the user types. Existing filter persistence semantics are unchanged.

**Sort Dropdown**: A shadcn `Select` exposes three sort options:

| Option | Order |
|--------|-------|
| Name A-Z (default) | Alphabetical ascending |
| Name Z-A | Alphabetical descending |
| Type | Grouped by type, alphabetical within group |

The selected option persists to localStorage under key `var-mgr.sort` so a user's preferred order survives reloads and tab swaps.

**Variable Row**: Each row is a two-line grid:

| Line | Left | Right |
|------|------|-------|
| 1 | Variable name (mono, truncated) | Action cluster: eye, expand, edit, delete |
| 2 | Value text (type-colored, truncated; a null value reads `null` in the null look, VMGR-93) | Secondary flag badges in this order (`Unused`, `IN` or `OUT`, `{x}` or `{{ }}`, `SECRET`) followed by the type badge. Type is always rightmost. |

Hovering a row shows a subtle background highlight. Clicking the row body is a no-op. ArrowUp / ArrowDown moves keyboard focus across rows. Both name and value show the full text in a `title` tooltip when truncated.

**Action Cluster**: Four icons, fixed left-to-right order:

| Icon | Behavior | Disabled |
|------|----------|----------|
| eye | Toggles the row's value display between runtime (default ON) and definition (OFF, italic + dim). For literals: runtime = current/last-known, definition = initial. For expressions: runtime = evaluated value, definition = expression text. | Always disabled on secret rows (tooltip: secrets are always masked). |
| expand | Opens (or focuses, if already open) the variable's floating inspect modal. The icon renders indigo while the variable's modal is open. | Disabled on a secret row that holds a hidden value (tooltip: secret values cannot be expanded). A secret holding no value opens; the modal only shows values, and the secret's value is set in the variable's window from the pencil (REQ-3.1.2.10). |
| edit (pencil) | Opens the variable's window, filled, in Edit (see Variable and Parameter Window below); the pencil on a variable whose window is open focuses it. Rename is the window's Name field. | While the run is actively running, visually disabled with a tooltip; in the history (read-only) workbench and while mirroring another person's session. Before a run, paused and after a run it is enabled (VMGR-107). On a read-only definition (a Function opened in View, or a Shared Function drilled into) it stays enabled and opens the window with only the Current value editable: the Initial view stays selectable and shows the saved value read-only, and every other field is locked. |
| delete (trash) | Routes to the existing delete flow, which always asks to confirm first; when output bindings or input references exist, the confirm names what will break. | Follows the pencil: while the run is actively running; in the history (read-only) workbench and while mirroring (VMGR-107). Always disabled on a read-only definition. |

**Eye Behavior Details**: The signal "row has runtime data" is membership in the runtime values map populated by the WebSocket variable-change events; it is independent of the value itself (a row whose runtime value is `null` still counts as "has runtime data"). When an expression row has no runtime entry yet, the runtime-side display shows a muted `<not evaluated yet>` placeholder; the eye stays enabled so the user can flip to the definition (expression text) and back.

**Eye State Lifecycle**: Per-row eye state is held in memory, default ON. It resets only when the runtime values map is cleared - the same Reset signal that clears the runtime values themselves (workbench session-id transition to null). Pause, Stop, and mid-session events do NOT clear runtime values, so they do NOT clear the user's per-row eye choices.

**Variable Rename**: Rename happens in the variable's window, the same for a variable, a Function local, an input and an output: the Name field is editable whenever the window edits, whichever value the switch picks (VMGR-108). The name rules are:

| Aspect | Behavior |
|--------|----------|
| Name validation | Python identifier rules (`[a-zA-Z_][a-zA-Z0-9_]*`), max 64 characters |
| Keyword rejection | Cannot use Python keywords (if, else, for, etc.) |
| Duplicate detection | Cannot rename to a name already used in scope; inside a Function, inputs, outputs and locals share one namespace |
| Reference updates | Every reference is refactored to the new name automatically |

**Rename Implications**: A rename reaches every place that names the variable, the same places the Unused badge counts:

| Location | Update |
|----------|--------|
| Expressions and templates in block settings and in other variables | Automatic |
| Condition rows of Assert, If/Else, Switch, While and Wait Until | Automatic |
| Variable Setter rows (values and targets) | Automatic |
| Output bindings and Function Block input fields | Automatic |
| The Secret mark a renamed secret output carries on each Function Block's outputs table | Automatic |
| Query aliases | Automatic |
| Expression validation | Flags any reference that no longer resolves |

**Variable and Parameter Window**: One window adds and edits a Test variable, a Function local, an input or an output (VMGR-106). The `+ Add` button opens it empty, titled "New ...", with no mode pill; the row pencil opens it filled, titled "Edit ...". Each Add opens a new window, cascaded like watchers and with no cap, and a second Add never replaces a window that is open; a window in Add keeps creating into the Function it was opened from: leaving that Function, drilling into another or returning to the Test while it is open never changes where the new input, output or variable lands (VMGR-101, VMGR-102). An Edit window is one per variable: the pencil on a variable whose window is open focuses it, and an Edit window left open when the canvas drills to another scope closes. While drilled into a Function the `+ Add` control is a menu - Input parameter / Output parameter / Variable - and every choice opens the same window, with the same layout and the same value field.

The window carries, top to bottom: Name; a resizable Description, on inputs and outputs only; the Type row (Type and Kind; inputs offer no Kind); the checkboxes together (inputs: Required, Mark as secret, Set to null, Create enum; variables, locals and outputs: Mark as secret, Set to null); Allowed values right under Create enum when it is on; and the value last. The window opens tall enough for its fields, up to the screen height, and scrolls. The title is 18px semibold and the field labels 16px medium; checkbox labels and hints keep their size (VMGR-114). The footer is Cancel / Create in Add and Cancel / Save in Edit, and Save shows busy while writing. Kind selects Literal / Expression / Template (Template for strings only) and drives the value field.

In Edit, a pill on the Value label line, right after the label, is the value switch: Initial / Current (inputs: Default / Current) picks which value the window edits (VMGR-108). Every other field belongs to the variable and is editable in both: name, type, kind, Mark as secret, Description, Required, Create enum and Allowed values. Save writes the definition when it changed and the Current value when it changed, whichever value the switch shows. Switching the Type or the Kind asks to confirm, then starts both values over as a new variable starts (the new type's default, except a Required input's Default, which restarts with no default; an empty source for Expression or Template; Set to null unticked), and clears a saved secret's stored value with them; the save check flags uses that no longer fit. Mark as secret is editable in Add and in Edit; in Edit, turning it on for a saved plain variable or off for a saved secret asks to confirm first. Turning it on seals the value; turning it off clears the value, so the secret is never shown (VMGR-109). A Current value of a Function's own input, output or local lands in that Function's scope, before a run, paused or after (REQ-3.2.1.6).

**Value Field**: The window shows the value as one same-height field per type (VMGR-110). A list, map or table is written out as text and can be typed into; a long value, or a text with line breaks, shows cut with "…", and line breaks are added in the larger editor. Text, list, map and table values carry an icon-only "Open editor" button - two outward arrows, styled like the expression toggle - that opens the shared "Edit value" window with its Visual / Code views and the JSON / XML / YAML pickers, which live only there; integer, float and boolean values get no button. An expression or a template keeps its expression box. A secret shows dots and is replaced, never shown. The field is preset with the type's default; an emptied number box never means 0.

**Inputs and Outputs**: An input's value is its default: the window labels it "Default value" in Add, and the Edit pill reads Default / Current (VMGR-113). A Required input may have a default or none, shown as an empty field; a Function Block never shows or uses it and must set the value, so the default is used only when the Function runs on its own, which uses a Current value first, when there is one. The exception is a choice-list or true / false input: a new Function Block starts on its default, Required included, and holds it as its own value (FIELD-8). A true/false input, Required or not, has no empty value, so, as for an enum input, its default is required: one side is always picked, and Add starts on False. A free-entry input that is not Required may have any default, null included; a Function Block shows it in gray and uses it when the Block sets nothing (FIELD-2). A secret input has no default. Window copy names the Function Block, never "the caller": "Required (the Function Block must set a value)", "Create enum (limit the Function Block to a fixed set of values)". An output's value is the fallback returned when nothing wrote it, and outputs offer the variable Kind choice (an expression output is evaluated at its first read or when the Function returns, once, and counts as written).

**Enum Inputs**: An enum input's Type is the value's type - String, Integer or Float - and Create enum, the last checkbox, adds an Allowed values list of that type right under it, with no per-item type column; there is no Array enum and no separate value type (VMGR-112). With Create enum on, Required is locked on, and Mark as secret and Set to null are off and locked; while Mark as secret is on, Create enum is off and locked ("off while secret is set"). The enum's default is required and picked from the allowed values: until it is set the select is empty, outlined in red with "Default value is required" under it, and Save stays disabled. While the allowed values list is empty it says so in red right away ("Enum allowed values cannot be empty"), so every field that blocks Save shows why. Turning an input whose value is null into an enum unticks Set to null and keeps Save disabled until the default and the Current value are set; setting the default also sets the Current value to it when the Current value is null or not an allowed value. The Current value is one allowed value. A Function Block's dropdown for the input starts on the default, and an expression bound to it is checked against the allowed values.

**Set to null**: The window offers `Set to null` in Add and in Edit, for either value (VMGR-111). Ticked, the Kind select and the value field give way to one grayed `null` field with no editor button, whatever the kind, and the window saves a plain null: for a variable, a null value; for an input, a null default, so a Function Block that sets nothing passes null; for an output, a null fallback, so an output nothing wrote returns null. Unticking brings back the kind and the value from before the tick, unless the null was saved. An emptied number box ticks it when it loses focus, where it is not locked. It is hidden while Kind is Expression or Template, so Kind goes back to Literal first. On a Required input it is off and locked ("off while Required is set"); on an enum input it is off and locked ("off while enum is set"); on a secret input's default it is ticked and locked, because a secret input has no default (VMGR-103, VMGR-104). A value that is wholly null opens with `Set to null` ticked where the window offers it. A whole `null` typed as the text of a list, map or table Code view is refused with that view's existing message (`Must be a JSON array`, `Must be a JSON object`, and for a table `Table must be a list of rows`, or `Must be a JSON array of objects` in the suite, regression and system editors) in every editor, the checkbox being the one way to a null whole value; `null` inside the text is a null item, map value or cell. See 15.15.3 for the component specification and REQ-3.1.3.24 for the rule.

**Empty State**: When the test has zero variables the panel shows centered, muted text `No variables yet. Click + Add to create one.` The filter input and the Add button remain visible.

**Editing**: Editing any variable - scalar or complex, literal or computed - happens in its window (see Variable and Parameter Window above), opened from the row pencil; the inspect modal only shows values. There are no separate per-type edit dialogs: the window's value field and the shared "Edit value" window it opens cover every type.

**Secret Variable Handling**:

| Aspect | Behavior |
|--------|----------|
| Value display | Masked as `********` (slate text) |
| `SECRET` flag badge | Red, sits left of the type badge |
| Eye toggle | Disabled with tooltip explaining secrets are always masked |
| Expand / inspect | Disabled while the secret holds a hidden value; a secret holding no value opens the inspect modal, which only shows values |
| Window | The window's `Mark as secret` checkbox declares a variable or parameter secret, in Add and in Edit; in Edit, turning it on for a saved plain variable or off for a saved secret asks to confirm first (VMGR-109) |
| Editing | The pencil opens the window, where a secret value shows as dots and is replaced, never shown, so secret values stay masked everywhere they appear |

**Variable Inspect Modal**: The row expand icon opens a floating, non-blocking inspect
surface dedicated to that one variable. The modal is draggable by its header bar, resizable
on both axes from a bottom-right corner grip (every floating sheet in the application shares
this both-axis resize with a per-surface minimum size), and never blocks the canvas or the
panel behind it.

| Aspect | Behavior |
|--------|----------|
| Instancing | One modal per variable; expanding an already-inspected variable focuses its existing modal. Modals for different variables stack freely with newest-on-top z-ordering and a cascade offset |
| Lifecycle | The modal is a live view: a value saved from any other surface appears in it immediately; deleting the variable closes its modal; leaving the workbench closes all modals |
| Source row indicator | While a variable's modal is open its panel row carries a 2px indigo inset left edge, a faint indigo background tint, and an indigo expand icon |
| Header | Drag grip, variable name (mono), flag badge for expression variables (`{x}`), type badge in the panel's badge palette, close button |
| Body | The full, untruncated value rendered by a type-shaped body (scalar text, array rows, map rows, table grid - see below). The list-view truncation rules do not apply inside the modal |
| Footer | Two zones: mode and view selectors (left; a string watcher shows a Format selector in the view slot) and the type-aware metric (center). The watcher only shows values, so it carries no action zone (VMGR-18) |

**Default Sizing by Type**: Integer, float, and boolean literal watchers open at the compact width; strings, expressions, arrays, and maps open at the standard width; tables open wider to give the grid room. All watchers share the same minimum size and both-axis resizability.

**Complex Value Bodies**: Array, map, and table variables render type-shaped Visual bodies instead of a flat text value:

| Type | Visual body |
|------|-------------|
| Array | One row per element: a muted, right-aligned, 0-based index column followed by the element value; hairline separators between rows |
| Map | One row per entry: the key in plain mono (wrapping, capped at just under half the width) followed by the value |
| Table | A mini-grid with a sticky uppercase muted header row. The first column is a dedicated muted 0-based row-index `#` column; the data columns follow. The grid scrolls horizontally when columns overflow |

Every cell is tinted by the value's own type using the panel palette (strings sky and quoted, integers violet, floats pink, booleans orange). Values that are themselves complex render as compact one-line JSON tinted by their kind (nested maps emerald, nested arrays amber).

**String Value Formatting**: A string variable's value is often structured text - JSON, XML, or YAML - or a template with interpolation placeholders. The string body carries a Format control that governs how the value is rendered in the watcher and how it is edited in the shared "Edit value" window. The control is a footer dropdown in the same left-zone slot that complex and expression watchers use for their View dropdown; a literal string has no View dropdown, so the slot is free. It uses the same text-label trigger with a chevron and an upward-opening text + check menu as the other footer selectors.

| Format | Rendering |
|--------|-----------|
| Auto (default) | The format is detected from the value when the modal opens and the matching highlighting applies; the trigger shows the detected format, for example `Auto (JSON)` |
| Plain | No highlighting; the value is flat monospace text |
| JSON / XML / YAML | Syntax highlighting and advisory linting for that language |
| Template | Interpolation mode: `{{ }}` regions are highlighted and validated, with variable autocomplete drawn from the variables in scope |

Detection runs once when the modal opens, and again only if the user re-selects Auto; it never re-runs while the user types, and a manual choice holds for that watcher. The format is a display and edit aid, not part of the value, so it is never persisted - a watcher reopens in Auto.

Linting in the data formats is advisory: a malformed region is underlined, but a string value is valid as a string whether or not it parses, so the lint never blocks Save. This is the deliberate contrast with the complex Code view, where invalid JSON does block Save because there the value is the parsed structure. In Template mode the data linters are suppressed, since `{{ }}` interpolation is not valid JSON or XML.

While editing in a data format in the "Edit value" window, a Format button sits beside the dropdown and pretty-prints the current text in the active format in place; because a string is freeform text the button reformats the current draft rather than reverting it to a last-valid structure. The button is offered only for the data formats - it is absent for Plain and Template.

**Inspect Views**: The footer carries a View dropdown immediately right of Mode - the same text-label + chevron trigger style and an upward-opening text + check menu - whenever the watcher has more than one way to render its value:

| Watcher | View menu |
|---------|-----------|
| Literal scalar | No View dropdown - a literal scalar has exactly one rendering |
| Expression scalar | `Literal` / `Expression` |
| Literal complex (array / map / table) | `Visual` / `Code` |
| Expression complex | `Visual` / `Code` / `Expression` |

| View | Shows |
|------|-------|
| Literal | The evaluated runtime value, rendered exactly like a literal of the declared type |
| Visual | The type-shaped body above |
| Code | The value as pretty-printed two-space JSON in the application's read-only code editor surface, with the same syntax tinting, line numbers, and code folding as the existing JSON editors. Tables serialize as a list of row objects |
| Expression | The expression source, raw, in the expression cyan |

The View selection is never persisted - a watcher always reopens in its evaluated view (Literal for scalars, Visual for complex types). Menus are plain text with a check on the selected entry; they contain no icons and no permanently disabled entries. A literal string scalar likewise shows no View dropdown, but its footer view slot carries the Format selector described under String Value Formatting.

**Complex Metrics**: The footer metric for complex types is identical in both views: arrays show `N items`, maps `N keys`, tables `N rows x M cols`.

**Complex Body States**:

| State | Rendering |
|-------|-----------|
| Empty value | A centered muted italic line: `no items` (array), `no entries` (map), `no rows` (table); a table with columns keeps its column header above that line; the metric shows the zero count |
| Null value | `null` in the null look, centered both ways between header and footer and staying centered as the watcher is resized; the view select stays, Code reads `null`, no count shows (VMGR-93, VMGR-94) |
| Initial mode | The whole body dims and italicizes; the data rendering is otherwise identical to Current |
| First full-value fetch in flight | A skeleton placeholder fills the body |
| Subsequent refreshes | Stale-while-revalidate: the previous value stays rendered until the fresh one lands; no flicker |
| Very large value | The Visual body renders the first 500 rows / items / entries followed by a muted `+ N more - switch to Code view` line; the Code view always renders the complete value |

**Inspect Modes**: A footer dropdown switches the modal between three views of the variable.
The trigger is a text label with a chevron; the menu lists the mode names with a check on
the selected entry and opens upward.

| Mode | Shows | Styling |
|------|-------|---------|
| Initial | The persisted definition value: the literal the author wrote, or the expression source text | Value dimmed + italic (matches the rows' definition view treatment) |
| Current (default) | The runtime value when execution data exists; identical to the definition when none does | Normal value styling |
| Compare | The Initial and Current values side by side in two panes | Initial pane dimmed + italic; Current pane normal |

Because Current degrades to the definition when no runtime data exists, the modal can be
staged as a watcher before a run: the user opens inspectors while idle and they go live the
moment execution produces values.

**Compare Mode**: Compare splits the body into two plain side-by-side panes - the persisted
definition on the left, the live value on the right. Each pane renders exactly what its mode
renders on its own: the Initial pane is the dim + italic definition body, the Current pane is
the full runtime body with its fetch pipeline, empty state, and type-mismatch guard
unchanged. There is no diff highlighting - a deliberate performance stance for large complex
values; the two renderings themselves show the delta. Each pane carries a small uppercase
muted label (`Initial` / `Current`); expression watchers annotate them `Initial - source` and
`Current - evaluated`. A hairline divider separates the panes, both share one vertical
scroll, and table mini-grids render and scroll inside their own pane rather than spanning the
sheet. The first-fetch skeleton and the large-value cap apply per pane. While idle, both
panes render the definition, consistent with staging a watcher before a run.

The footer metric becomes a delta describing both values, and the arrow always renders while
in Compare even when the two sides are equal:

| Watcher | Compare metric |
|---------|----------------|
| string | `42 -> 84 chars` |
| integer / float / boolean | empty |
| array | `3 -> 4 items` |
| map | `2 -> 3 keys` |
| table | `1 -> 3 rows` (rows only; the column delta is visible in the panes) |
| a null side | `null` in place of its count, e.g. `null -> 3 items` |
| expression (evaluated) | `expression -> 84 chars` / `expression -> 4 items` |
| expression (no runtime value) | `expression -> no value yet` |
| type mismatch | `type mismatch` in the warning amber, with no arrow |

In Compare the View dropdown drives the panes the same way it drives a single mode. Literal
complex watchers keep `Visual` / `Code` and the choice applies to both panes. Expression
watchers always render the Initial pane as the cyan source - the definition IS the source -
so the View dropdown describes the Current pane only: the `Expression` option is absent
because it would merely duplicate the Initial pane. An expression scalar is therefore left
with a single `Literal` option, so its dropdown renders disabled showing `Literal`;
expression complex watchers keep `Visual` / `Code`. A view selection of `Expression` carried
in from another mode coerces to the evaluated view while in Compare and is restored on
leaving. Compare is never persisted - a watcher always reopens in Current.

**Editing in the Window**: The inspect modal only shows values; it carries no Edit and no
rename (VMGR-106). The variable's window is the editing surface. The row pencil opens it
whenever the run is not actively running - before a run, paused or stopped - and is shown
disabled with an explanatory tooltip while a run is in flight (VMGR-107). The window's value
switch picks which value the window shows and edits: Initial (an input's Default) is the
definition, a design-time edit; Current is the live value (REQ-3.2.1.6). Save writes the
definition when it changed and the Current value when it changed, whichever value the switch
shows. Compare stays a watcher view.

Every field of the variable is editable whichever value the switch picks (VMGR-108): its name,
type, kind and secrecy and, on inputs and outputs, its declaration fields. Changing the type
or the kind asks to confirm and starts both values over as a new variable starts.

| Edit affordance | Behavior |
|-----------------|----------|
| Name | An inline mono field; renaming refactors all references and is validated live with the same name rules used across the application. Available whenever the window edits, in either value |
| Scalar value | The value field keeps the type's value color; a boolean is a true / false toggle |
| Text, list, map, table value | One typeable line; the Open editor button opens the shared "Edit value" window, which carries the bodies below |
| Array body | Inline per-element inputs with per-row reorder and delete and an `+ Add item` line |
| Map body | Inline key and value inputs with per-row delete and an `+ Add entry` line |
| Table body | Editable column headers (a name input spanning the cell width plus a full-name type select - string, integer, float, boolean), editable cells tinted by column type (boolean cells use a true/false selector), a per-column menu (move left, move right, delete column - delete disabled at one column), a trailing add-column control, an `+ Add row` line, and per-row delete |
| Code editing | The Code view is a writable JSON editor on the same editor surface as the read-only Code view; a `Format` button pretty-prints the JSON two-space |
| Expression editing | The expression source is edited in the window's expression box, with variable autocomplete and an expand-to-full-editor affordance |

In the "Edit value" window the View dropdown moves between the Visual rows and the Code JSON:
switching from Visual to Code is always safe, while switching from Code back to Visual is
blocked only when the value cannot become the grid - invalid JSON, or a value that is not a
list of rows - in which case the disabled `Visual` option carries an alert icon and explanatory
tooltip inside the View menu, where the user sees it on trying to switch, and Save is disabled
until it is fixed. A cell-type mismatch does not block the switch: it carries into the Visual
view, where it stays flagged and Save stays disabled, and is fixed in the grid rather than
silently coerced. For a table the two views divide the work: the Visual view owns the column
structure - names, types, and adding, removing, or reordering columns - while the Code view
edits the cell data within that structure. The column schema defined in the Visual view travels
with the value, so reshaping the JSON to an unknown or missing column is reported as an error
that points back to the Visual view rather than silently redefining the table. For an
expression, editing operates on the source in the window's expression box; the evaluated
Literal, Visual, and Code views are runtime output, shown in the watcher, and are not editable.

Validation is design-time only and lives in shared validators: name rules, scalar value
parsing, table-schema validation, JSON parsing, and the expression-cycle check. The edit bodies
are controlled inputs that report their draft and validity upward; the window owns the
validation result, renders inline errors, and keeps Save disabled while any field is invalid.
There is no runtime undefined-variable flow - a saved value is validated before it is written,
and the structured table value enforces its own column types when the engine mutates it. Saving
writes the definition when it changed and the Current value when it changed, whichever value
the switch shows, through the standard variable save path, plus the reference refactor when the
name changed, and Save shows busy while writing.

A table is validated against its own column schema - the named, typed columns - consistently
in both the Visual and the Code view. The schema is authoritative: the Visual view defines the
column names and types, and the same schema governs the Code view, whose JSON edits change cell
data within it. Validation enforces the contract the engine enforces at runtime: every column
name is present and unique, and every cell matches its column's type. A cell holds a typed
value, an empty string, or an explicit null, and the three are distinct states. An explicit
null is valid for any column type; an empty string is valid only for a string column; a new
row or column seeds the column type's default (`0`, `false`, or the empty string) rather than
null. The null state is set through a per-cell affordance and rendered distinctly as a muted
italic `null`, and invalid text is kept and flagged in place rather than silently coerced to
null. Both views gate Save on the same result, so the Code view is never a path to save
a table the Visual view would reject. When something is wrong, every offending cell and column
header is marked in place and the editor names the first problem - which cell or column, and the
type expected - with a count when several remain.

**Inspect Metric**: The footer metric describes the value the body currently displays:
string values show `N chars`; integer, float, and boolean values show no metric; an
expression source shows `expression`; an expression with no runtime value shows
`no value yet`; a mismatched runtime value shows `type mismatch` in the warning amber; a null value shows no metric.

**Expression Variables in the Inspect Modal**: Expression-ness is an explicit view, not an
implicit fallback. The evaluated views (Literal for scalars, Visual and Code for complex
types) render the evaluated runtime value exactly like a literal of the declared type; the
Expression view renders the cyan source and is reachable in any mode. While the expression
has no runtime value - idle, not yet referenced by the run, or a history run that never
referenced it - the evaluated views show a single centered muted italic `Not evaluated yet.`
line. Evaluation state needs no dedicated signal: the engine realizes an expression on first
reference and pushes the realized value like any other variable change, so "evaluated" is
exactly "a runtime value exists".

In Initial mode the View dropdown is disabled showing `Expression`: the persisted definition
of an expression variable IS its source. Switching back to Current restores the previous
view selection. Cyan is reserved for expression source everywhere in the modal - an
evaluated value never renders cyan.

**Type-Mismatch Safeguard**: Every write path enforces declared types (design-time
validation, the engine's expression type gate, and the Variable Setter's runtime check), so
a runtime value whose type contradicts the declaration cannot occur in normal operation. If
one ever slips through, the watcher refuses to guess: instead of the typed body it renders
an italic warning line in the warning amber naming both types (`received map - declared
integer`) with the raw value below, tinted by its actual type, and the metric shows `type
mismatch`. The Code view of complex watchers is type-agnostic and continues to render the
raw payload. The guard is display-only; the write-path gates remain the enforcement points.

##### REQ-3.1.3.12 Auto-refresh variables on change

Variable values update automatically during workflow execution without user intervention. The update mechanism uses WebSocket push from the backend to the frontend.

**WebSocket Push Architecture**: The backend pushes variable changes to all subscribed workbench clients for the active session:

| Component | Responsibility |
|-----------|----------------|
| Execution engine | Emits `variable_changed` event when a block modifies a variable, when an expression is evaluated, or when any other engine path writes a variable |
| WS server | Forwards every emitted change to the workbench session's subscribers |
| Frontend | Records the new value in a runtime values map and re-renders matching rows in real-time |

**Throttle Strategy**: To prevent flooding during rapid updates (e.g., fast counters):

| Rule | Value | Rationale |
|------|-------|-----------|
| Max push rate | 1 per variable per 200ms | Prevents flood, maintains real-time feel |
| Coalescing | Last value wins | Only latest value pushed within window |
| Per-variable | Independent throttle | One fast var doesn't delay others |

**Throttle Implementation Details**: The backend `variable_push` service implements intelligent throttling:

| Aspect | Implementation |
|--------|----------------|
| Time tracking | Uses `time.monotonic()` for accurate elapsed time (immune to clock adjustments) |
| Immediate push | If outside throttle window, push immediately |
| Pending storage | If inside window, store value as pending (overwrites any previous pending) |
| Scheduled flush | After initial push, schedule async task to flush pending after remaining delay |
| Last-value-wins | Multiple rapid updates within window result in single push of final value |
| Per-variable state | Each variable path maintains independent last-push timestamp |

**Throttle Timeline Example**: Variable `counter` updated rapidly:

```
t=0ms    counter=1  → Push immediately (no prior push)
t=50ms   counter=2  → Store pending (within 200ms window)
t=100ms  counter=3  → Overwrite pending (still in window)
t=150ms  counter=4  → Overwrite pending (still in window)
t=200ms  (scheduled) → Push counter=4 (flush pending)
t=250ms  counter=5  → Push immediately (new window)
```

**Push Content by Type**: Different data types push different content for efficiency:

| Type | Push Content | Rationale |
|------|--------------|-----------|
| Scalar (string, integer, float, boolean) | Full value | Small payload |
| Array | `{count: n, changed: true}` | Summary only, FE fetches on expand |
| Map | `{count: n, changed: true}` | Summary only, FE fetches on expand |
| Table | `{rows: n, cols: m, changed: true}` | Summary only, FE fetches on expand |

**WebSocket Message Format**:

| Message Type | Content | Panel Response |
|--------------|---------|----------------|
| `variable_changed` | Variable path, value (or summary), type | Update matching row(s) |
| `block_completed` | Block ID, output variable summaries | Update affected variables |
| `execution_completed` | Final state flag | Refresh all variables via API |

**Visual Feedback for Updates**: When a variable value changes, the row provides subtle visual feedback:

| Feedback | Duration | Purpose |
|----------|----------|---------|
| Background highlight | 300ms fade | Draws attention to change |
| Value animation | Crossfade transition | Smooth value transition |

**No Polling**: WebSocket push eliminates polling overhead:

| Context | Strategy |
|---------|----------|
| Execution running | WS push only (throttled) |
| Execution paused | WS push (no polling needed) |
| Idle/No execution | Static (no updates) |

**Full-Value Refresh for Open Inspectors**: A complex-type change summary on the stream doubles as the invalidation signal for that variable's open inspect modal: the modal re-fetches the full value from the workbench session variables resource when a summary for its name arrives (see REQ-3.2.2.18). Rows never fetch full complex values - they render the summary itself.

**Secret Handling**: Secret variables maintain masking during updates. Changed secret values trigger the highlight animation without revealing the actual value.

**Error States**: Variable fetch failures display gracefully:

| Error | Display | Recovery |
|-------|---------|----------|
| Variable not found | Muted "undefined" text | Auto-retry on next event |
| Network error | Error icon with tooltip | Manual refresh available |

##### REQ-3.1.3.13 Computed variables with expressions and runtime evaluation

The Variable Manager surfaces computed (expression) variables with a cyan `{x}` flag badge
on the row and in the inspect modal header. The inspect modal gives every expression
variable two complementary renderings - the expression source and the evaluated runtime
value - switched through the explicit View dropdown (see the Inspect Views and Expression
Variables sections under REQ-3.1.3.11). Evaluation follows the lazy model of the variable
system design (Computed Variables):
an expression evaluates when first referenced, the realized value is pushed like any other
variable change, and an unreferenced expression keeps its `Not evaluated yet.` display
through run end and into the execution's history view.

The same expression capability is available on plugin parameter values, including the
structured-content fields (a JSON or XML body): the parameter row's expression toggle drives
the whole value from an expression that resolves to a string at runtime, in which case the
literal data-format validation is skipped (the resolved text is validated when it is produced).
See configuration-editors REQ-3.4.2.2 for the parameter-side editing contract.

Editing a computed variable edits its expression source, never its evaluated result: the
variable's window presents the source in its expression box with variable autocomplete, and the
expression-cycle check is part of the design-time validation that gates Save (see Editing in
the Window under REQ-3.1.3.11).

#### 15.6.3 Console Panel

The Console panel provides real-time visibility into execution logs, enabling test engineers to monitor workflow progress, diagnose issues, and understand execution behavior without leaving the Workbench.

##### REQ-3.4.2.7 Plugin runtime UI with console for logs

The Console panel displays streaming log output during workflow execution. It occupies the bottom edge of the canvas area, sharing horizontal space with the workflow canvas.

**Panel Dimensions**:

| State | Height | Behavior |
|-------|--------|----------|
| Collapsed | 45px | Header bar only with title and toggle |
| Expanded | 200px (default) | Resizable 100-600px via resize handle |

**Panel Structure**:

| Region | Position | Content | Purpose |
|--------|----------|---------|---------|
| Header | Top | Toggle, title, filter controls | Panel control and log filtering |
| Log area | Below header | Scrollable log entries | Display execution output |

**Header Components**:

| Component | Position | Behavior |
|-----------|----------|----------|
| Collapse toggle | Left | Chevron up/down toggles panel state |
| Terminal icon | After toggle | Visual identifier |
| "Console" label | After icon | Panel title |
| Error badge | After label | Red ⚠ + count, visible only when collapsed and errors exist |
| Level dropdown | Right | Dropdown to filter by minimum level (default: INFO) |
| Settings button | After level | ⚙️ icon, opens display options popover |
| Clear button | Far right | Clears log buffer (available during idle state) |

**Settings Popover** (⚙️ button):

| Option | Default | Behavior |
|--------|---------|----------|
| Show Timestamps | ON | Toggle timestamp column visibility |
| Show Level | ON | Toggle level badge visibility |
| Show Source | ON | Toggle source column visibility |
| Show System Logs | ON | Toggle visibility of SYSTEM source logs |
| Text Size | 12px | Adjustable 10-18px with +/- buttons |

Display settings are a remembered per-browser preference shared by every console surface - the live workbench console, the Execution History console, and the report log panel - so a choice made once holds across pages and reloads.

**Message Bound**: The console renders a message exactly as it was recorded and shortens nothing of its own. The engine bounds a message once, where the log entry is created, so a single block can never flood the stream: past that bound the message ends with a line stating how many characters were left out. Every shortening the engine applies - to a message, to a rendered table's rows, columns, cells or column widths - says so in the text it produces, so the console never shows a value cut without notice.

**Source Column**: The Source column names what wrote the line. The engine's own lines carry `SYSTEM`. The lines a Function writes about itself carry `FUNCTION`. The notice the console writes itself when it cannot connect carries `CONSOLE`. A block's lines carry the block type's name as the palette shows it; when the block's Name differs from the type's name, the Name follows after a colon with one space on each side: `SQL` for a block still named SQL, `SQL : Run PSQL Proc` for a block the author named. A block's Description never appears in the column. The text is set once, where the line is recorded, and every console surface, the copied text and the report show it unchanged.

**Source Column Resizing**: A subtle vertical drag handle at the right edge of the source column allows users to resize the column width. Min: 10ch, Max: 50ch, Default: 20ch. With the handle focused, the Left and Right arrow keys change the width by one character, and the handle shows the same highlight as on hover. The column keeps its width at every console width, and the message takes the rest. Long source names are truncated with ellipsis and show full name on hover tooltip.

**Level Dropdown**: Selecting a level shows entries at that level and above. Options: DEBUG, INFO (default), WARN, ERROR, FATAL. Standard log hierarchy - "INFO" shows INFO + WARN + ERROR + FATAL.

**Error Badge (Collapsed State)**: When the panel is collapsed and errors exist in the log buffer, display a red triangle with exclamation mark (⚠) and error count in the header. This alerts users to errors without requiring the panel to be expanded.

| Panel State | Errors Present | Badge Display |
|-------------|----------------|---------------|
| Collapsed | No | No badge |
| Collapsed | Yes | Red ⚠ triangle + count (e.g., "⚠ 2") |
| Expanded | Any | No badge (errors visible in stream) |

**Error Badge Click Behavior**: Clicking the error badge expands the panel and activates a temporary "errors only" filter. This is a client-side filter (WebSocket still sends all logs at the subscribed level) that shows only ERROR and FATAL entries.

**Temporary Error Filter UI**: A dismissible chip appears in the header:

```
┌─────────────────────────────────────────────────────────────────────┐
│ ▼ Console   [Errors only ✕]      [Level: INFO ▼]  [⚙️]  [Clear]    │
└─────────────────────────────────────────────────────────────────────┘
```

| Element | Behavior |
|---------|----------|
| Chip | Appears in header after console label when error filter is active |
| "✕" button | Dismisses chip, returns to normal view with current level |
| Level dropdown | Remains unchanged, filter is additive |

**Filter behavior**: When active, only ERROR and FATAL entries display regardless of the level dropdown setting. New logs continue arriving via WebSocket; non-error logs are buffered but hidden. Dismissing the filter reveals all buffered logs at the current level.

##### REQ-3.7.2.5 Custom logging from tests/blocks

Log entries originate from multiple sources during execution:

| Source | Description | Example |
|--------|-------------|---------|
| Log plugin | Explicit user-defined messages | "Starting data validation phase" |
| Block execution | Framework messages about block lifecycle | "Executing block: HTTP Request" |
| Plugin output | Plugin-specific diagnostic output | "Response status: 200 OK" |
| Error capture | Exception and error details | "Connection timeout after 30s" |

**Log Entry Structure**:

| Field | Type | Description |
|-------|------|-------------|
| timestamp | ISO 8601 | When the log entry was created |
| level | string | DEBUG, INFO, WARN, ERROR, FATAL |
| message | string | Log message content |
| source | string | What wrote the line: `SYSTEM`, `FUNCTION`, or the block's type and Name as the Source column shows them |
| blockId | string (optional) | Associated workflow block ID |

##### REQ-3.7.2.6 Log levels (DEBUG, INFO, WARN, ERROR, FATAL)

Log entries are categorized by severity level, following the standard hierarchy from least to most severe:

**Row Coloring by Level** (entire row, regardless of which columns are visible):

| Level | Text Color | Background | Rationale |
|-------|------------|------------|-----------|
| DEBUG | Muted gray (#71717A) | None | Verbose, fades into background |
| INFO | Cyan (#22D3EE) | None | Standard, readable |
| WARN | Amber (#FBBF24) | None | Noticeable, not alarming |
| ERROR | Red (#F87171) | None | Clearly indicates problem |
| FATAL | Red (#F87171) | Subtle red tint | Maximum attention, rare event |

**Element-Specific Styling Within a Row**:

| Element | Style |
|---------|-------|
| Timestamp | Slightly muted from row color |
| Level badge | Row color with subtle background pill |
| Source | Slightly muted from row color |
| Message | Full row color |

**Color Independence**: Row coloring applies based on the entry's level regardless of column visibility. If the user hides the Level column via settings, the row still displays in the level's color scheme.

##### REQ-3.7.2.8 Filter logs by level in report viewer

The Console header provides a level dropdown for filtering visible log entries:

**Level Dropdown**:

| Selection | Entries Shown |
|-----------|---------------|
| DEBUG | DEBUG + INFO + WARN + ERROR + FATAL (all) |
| INFO (default) | INFO + WARN + ERROR + FATAL |
| WARN | WARN + ERROR + FATAL |
| ERROR | ERROR + FATAL |
| FATAL | FATAL only |

**Filter Behavior**:

- Selecting a level shows entries at that level and above (standard log hierarchy)
- Default selection is INFO (hides verbose DEBUG output)
- Filter state persists within the session
- Changing filter does not clear the log buffer, only visibility

**Log Entry Rendering**:

Each log entry renders as a single row in the log area:

| Element | Position | Style |
|---------|----------|-------|
| Timestamp | Left | Monospace, muted color, condensed format (HH:MM:SS.mmm) |
| Level badge | After timestamp | Colored badge with level abbreviation |
| Source | After badge | Muted text, truncated with tooltip |
| Message | Right (flex-grow) | Primary text, preserves whitespace |

**Auto-scroll Behavior**:

| Condition | Behavior |
|-----------|----------|
| User at bottom | Auto-scroll to new entries as they arrive |
| User scrolls up | Pause auto-scroll, show "Jump to bottom" button |
| Click "Jump to bottom" | Scroll to bottom, resume auto-scroll, hide button |
| Manual scroll to bottom | Resume auto-scroll, hide button |

"At bottom" detection uses a ~20px threshold to account for slight scroll imprecision. The "Jump to bottom" button appears floating at the bottom-center of the log area when auto-scroll is paused.

**Log Buffer Management**:

| Aspect | Specification |
|--------|---------------|
| Maximum entries | 10,000 entries (default) |
| Overflow behavior | Remove oldest entries (FIFO) |
| Clear button | Manually clears buffer (available during idle state) |
| Execution Reset | Clears buffer automatically |

The buffer size is fixed; it is not a user setting.

**Virtualization**: The log area uses virtualized rendering to maintain performance with large log volumes. Only visible rows plus a small buffer are rendered to the DOM. This is essential for 10K entries with variable-height multiline messages. Implementation uses a virtualization library (e.g., `@tanstack/react-virtual`).

**Log Persistence Architecture**:

Tracked executions persist their logs to storage for the execution's lifetime (subject to the retention policy). Ephemeral Workbench runs do not persist: they stream and render live, but their log stream is released when the session ends or the backend restarts.

| Layer | Responsibility |
|-------|----------------|
| Backend storage | Persist all logs (all levels) for tracked-execution lifetime. Ephemeral workbench sessions are not persisted. |
| WebSocket | Push new logs at subscribed level (real-time stream) |
| REST/WS query | Fetch historical logs on-demand (paginated) |
| Frontend | Display up to 10K entries, virtual scroll, request more as needed |

**Key principle**: For tracked executions, storage is the source of truth - not in-memory buffers. The 10K limit is a frontend display cap, not a storage cap.

**Every tracked run reads back the same way**: The merged-console reader serves any tracked execution, whether it dispatched child runs or is a single test run of its own. A suite or regression walks its recorded scope chronology; a single test run has none to walk, so the reader falls back to the run recorded on the execution itself. Both the paged history read and the history a client receives when it subscribes mid-run go through that one reader, so a viewer joining a run already in progress always receives the lines produced before it arrived.

**Restart durability**: A backend restart does not erase the Execution Detail console for tracked runs. Every tracked test execution and every sequential suite/regression parent execution flushes its log entries to the execution's `logs.yaml` file on disk as they are produced, through the same write path that updates the in-memory log stream. When a client subscribes to a tracked execution's log topic after a restart, the WebSocket subscribe handler primes the in-memory stream from the persisted file before sending the initial history page. Sequential children forward their entries to the parent execution's log stream (so the parent's console shows the full run), and those forwarded entries are part of the parent's persisted `logs.yaml` - no child-level log files exist. Ephemeral Workbench sessions explicitly bypass this write path, matching their non-tracked nature.

**WebSocket Integration**:

The Console subscribes to log events via the WebSocket infrastructure:

| Topic Pattern | Content | When Received |
|---------------|---------|---------------|
| `log:{session_id}` | Log entry payload | During active execution |

**Log Event Payload**:

```
{
  "type": "log",
  "payload": {
    "timestamp": "2025-01-22T10:30:45.123Z",
    "level": "INFO",
    "message": "Processing record 42 of 100",
    "source": "Log : Data Processor",
    "blockId": "block-uuid-123"
  }
}
```

**Server-Side Level Filtering**:

To optimize bandwidth, the backend filters logs at the subscribed level before sending. The client specifies the minimum level when subscribing. Only logs at that level and above are pushed via WebSocket.

**Level Switch Flow** (e.g., WARN → INFO):

| Step | Actor | Action |
|------|-------|--------|
| 1 | FE | Sends level change request via WebSocket |
| 2 | BE | Acknowledges, switches to pushing INFO+ logs |
| 3 | FE | Clears console display |
| 4 | BE | Continues pushing new real-time logs (no gap) |
| 5 | BE | Simultaneously queries storage backwards for historical INFO+ logs |
| 6 | BE | Pushes historical page (newest first, working backwards) |
| 7 | FE | Renders: real-time logs at bottom + historical logs above |
| 8 | User | Scrolls up to see more history |
| 9 | FE | Requests next page of historical logs |
| 10 | BE | Queries more history from storage, returns page |
| 11 | - | Continue until 10K display limit or beginning of execution |

**Critical guarantee**: No logs are lost during level switch. Backend buffers briefly if needed to ensure continuity between the switch acknowledgment and new level streaming.

**Pagination Strategy**:

| Aspect | Specification |
|--------|---------------|
| Page size | ~100 entries per request |
| Direction | Backwards from current position (older logs) |
| Trigger | User scrolls near top of loaded content |
| Limit | Stop at 10K total displayed entries or execution start |
| Deduplication | Use log sequence numbers to prevent duplicates |

**Accessibility**:

| Feature | Implementation |
|---------|----------------|
| Screen reader | Log entries announced as they arrive (optional, respects reduced motion) |
| Keyboard navigation | Tab to panel, arrow keys to navigate entries |
| Focus management | New entries don't steal focus from current interaction |

#### 15.6.4 Execution Detail View

The Execution Detail view is the authoritative page for observing a tracked execution. It serves both as the live progress surface while an execution is running and as the historical record after it completes. One shared page component serves Test, Suite, and Regression executions; per-type variation is confined to the Components table contents.

**Entry Points**:

| Entry Point | Behavior |
|-------------|----------|
| Executions list row click | Navigate to `/executions/{id}` |
| RunDialog post-start | After a tracked execution is created, the user is navigated to `/executions/{id}` |
| Workbench breadcrumb tracked-execution prefix | Link target is `/executions/{id}`; see section 15.6.0.3 |
| Re-run action on the detail view | Creates a new tracked execution, then navigates to the new execution's `/executions/{new_id}` |

**Layout**: The page reuses the visual language of Suite Detail and Regression Detail, with four regions:

| Region | Position | Content |
|--------|----------|---------|
| Top bar | Top | Breadcrumb `Executions / {Name}`, status indicator (pulsing dot while Running, static dot while Terminal), action buttons (see below) |
| EXECUTION INFO | Left | Read-only panel: Status, counts (Total / Passed / Failed / Pending / Skipped / Excluded), Started, Duration, Triggered user, Environment, Tags, Description |
| Components table | Center | Per-type content (see below) |
| Console | Bottom | Live log stream while Running; full historical log when Terminal |

The detail view does not include a Variable Manager. Live variable inspection is a Workbench-only capability (see 15.6.0.3); historical variable post-mortem lives in the per-test canvas reached through the per-row View action, not in this detail view.

**Components Table by Entity Type**:

| Entity | Rows |
|--------|------|
| Test execution | A single row representing the test itself |
| Suite execution | A flat list of the suite's tests |
| Regression execution | Suite rows as collapsible parents; when expanded, the suite's tests render as children. Suites are collapsed by default. |

Shared columns: Name, Status, Duration, Actions. Regression suite parent rows display aggregate Status (for example `2/3 passed`) and aggregate Duration.

**Table Controls**:

| Control | Behavior |
|---------|----------|
| Search filter | Matches suite AND test names. A match inside a collapsed suite auto-expands that suite in the search result view. |
| Status filter | Multi-select over Running / Passed / Failed / Pending / Stopped / Skipped / Excluded. Excluded is hidden by default (those rows are noise for subset runs); Skipped is visible by default because mid-run skips describe something that actually happened during the run. Excluded = not selected at dispatch time. Skipped = bypassed mid-run (halt scope fan-out, Stop fan-out, or per-test halt with status=skipped). |
| Column selector | Standard show/hide columns widget, matching the Executions list pattern. |
| Expand All / Collapse All | Regression view only. Operates on the current page of suites. |
| Pagination | DataTable pagination on top-level rows. Suite view paginates tests. Regression view paginates suites; expanded suites reveal all their tests at once with no nested pager. Test view has one row and no pagination. |
| Selection mode | Standard DataTable selection with checkboxes on test rows only. Suite parent rows (Regression view) are not selectable. Enables Mass Action Export Selected. |

**Row Actions**:

| Action | Visibility | Behavior |
|--------|-----------|----------|
| View | Completed test rows only (Passed / Failed / Stopped) | Navigates to `/executions/history/{run_id}`, which renders a historical read-only canvas of that test with full painted per-block state sourced from this execution's stored timeline and variables. |
| Export | All test rows regardless of status | Downloads a JSON file for that single test's slice of the execution. Pending tests export as a minimal record so downstream consumers can parse every row uniformly. |

Clicking anywhere else on a row has no effect; only the checkbox, View, and Export controls are interactive.

**Top Bar Actions**:

| State | Actions |
|-------|---------|
| Running | Stop (confirmation required); View Running (opens the Workbench on the current tracked session per 15.6.0.3); Export (whole execution as JSON) |
| Terminal (Passed / Failed / Stopped) | Re-run (opens the prepopulated RunDialog with the entity and environment; the user confirms or cancels; confirming creates a new tracked execution and navigates to its detail); Export (whole execution as JSON); Delete (only when not Running / Paused) |

Stop confirmation follows the unified policy in 15.6.0.3 (tracked executions are confirmation-gated; ephemeral Workbench runs are not).

**Mass Action**: Selection mode exposes a single operation, Export Selected, which downloads a JSON bundle containing the selected test rows. All rows are selectable regardless of status, matching the per-row Export behavior.

**Re-run Flow**:

| Step | Actor | Action |
|------|-------|--------|
| 1 | User | Clicks Re-run on a Terminal execution |
| 2 | UI | Opens RunDialog prepopulated with the execution's entity type, entity id, and last used environment |
| 3 | User | Confirms (Run) or dismisses (Cancel) |
| 4 | UI | On Run: POSTs to `/executions` and navigates to `/executions/{new_id}` |

**Live Updates**: Until the execution reaches a terminal state, the page polls the execution detail endpoint every 1.5 seconds. Each response updates the toolbar, the EXECUTION INFO counts and the Components table. The Console receives its entries over the execution's live log topic. On the transition to a terminal state, the top bar swaps its action set (Stop / View Running / Export becomes Re-run / Export / Delete).

**Export Format**: JSON. The file contains the execution metadata (id, entity type, entity id, environment, timings, status, counts) and a per-test record array. Each per-test record carries status, duration, and a slice of logs and timeline entries relevant to that test. Pending tests carry only their identifier and status, so downstream consumers can parse every row with the same schema.

**Workflow Snapshot**: Every tracked execution persists a snapshot of the test's workflow (nodes and edges) at dispatch time on the execution record. The snapshot is immutable and authoritative: subsequent edits to the source test never alter what a past execution shows, and deleting the source test does not remove or invalidate the snapshot - the execution record, including the snapshot, lives on in persistent history. This is deliberate: execution history is an audit surface, so a past run must always be reviewable even after the test has been modified or removed. The historical test canvas (15.6.4.1) renders from this snapshot; the per-test export slice includes the snapshot so downstream consumers can reconstruct the exact graph.

**Error and Edge States**:

| Condition | Behavior |
|-----------|----------|
| Execution id not found | Dedicated empty-state page with a link back to the Executions list |
| Network disconnect during Running | The last known execution stays on screen with the failure named beside it, and polling continues |
| Execution deleted mid-view | Redirect to the Executions list with a toast notification |
| Browser refresh mid-execution | Re-hydrate from the detail endpoint, then resume polling |

**Workbench vs Execution Detail**: The Workbench shows the live canvas-centric view of whatever the user is currently running (ephemeral or tracked). The Execution Detail shows the aggregate observational view of a specific tracked execution (Test, Suite, or Regression) with cross-test context and persistent history. The two views are complementary: the Workbench is the debugger surface; the Execution Detail is the record-and-drill-in surface.

#### 15.6.4.1 Historical Test Canvas

The Historical Test Canvas is the per-test drill-in page reached from the Execution Detail Components table's row-level View action on a terminal test row. It is a pure post-mortem surface: read-only, decoupled from any live Workbench or tracked execution, and source-of-truth for what a given test actually did inside a given execution.

**Route**: `/executions/history/{run_id}`. Opening multiple historical canvases in multiple tabs is explicitly supported, including in parallel with a live ephemeral Workbench run or a running tracked execution - the page is independent of the Workbench session registry and cannot collide.

**Entry Points**: The View (eye) icon on a terminal test row in the Execution Detail Components table (Passed / Failed / Stopped). The icon is absent for non-terminal rows. Hyperlinks from the Executions list also target this route.

**Layout**: The page reuses the Workbench canvas composition (canvas + Variable Manager + Console) in a read-only mode. The palette shows only as a disabled icon strip. No run / debug / stop controls. No environment selector. No reset action. The breadcrumb bar shows the test's status. The action bar shows the run's environment as a read-only label, the Report button and the Open in Workbench button.

| Region | Content | Read-only Mode |
|--------|---------|----------------|
| Canvas | The test's snapshotted workflow painted with per-block status (passed / failed / skipped / stopped / pending) from the execution's stored timeline | No palette, no drag, no connect, no block-drop, no context menu, block inspection dialogs are view-only |
| Variable Manager | The test's declared variables (type, initial value, `isExpression` / secret flags) overlaid with the execution's end-of-run runtime values (secrets masked). Same flat list, sort, filter, and per-row eye toggle as the live panel. | `+ Add` hidden; edit (pencil) and delete (trash) disabled (read-only mode); eye toggle remains usable - eye-ON renders the snapshot's runtime value (the evaluated outcome for expressions), eye-OFF renders the test's declared definition (the initial value for plain variables, the expression source text for `isExpression`). When the source test was deleted after the run the panel falls back to the runtime snapshot only and the toggle becomes a no-op for that case. |
| Console | The slice of log entries belonging to this test within the execution | Clear button visible but disabled, all other controls (level dropdown, auto-scroll, settings popover) remain functional; the "Jump to bottom" affordance still works |

**Painting Source**: The canvas shape is the workflow snapshot persisted on the execution record at dispatch time (see 15.6.4 "Workflow Snapshot"). Per-block status is sourced from the stored timeline. The Variable Manager carries two views: declared definitions resolved from the source test entity (type, initial value, `isExpression` / secret flags) drive the eye-OFF surface, while the execution's end-of-run variable store drives the eye-ON surface. Logs are the subset of the execution's persisted logs whose `test_id` matches this test. Because the snapshot is authoritative and immutable, the historical canvas is stable across any subsequent edits to the source test - no drift, no banner, no reconciliation. If the source test has been deleted, the definitions resolve to empty and the panel renders the runtime snapshot only.

**Multi-Tab Independence**: Opening a historical canvas never interacts with the Workbench session for the current user, and never interacts with any live tracked execution. Multiple tabs can render multiple historical canvases concurrently with no shared state. The Workbench can be idle, mid-ephemeral-run, or mid-tracked-run without affecting what these tabs render.

**Deleted Source Test**: Deleting a test from the Tests list removes the live test entity but preserves every tracked execution that ran against it. The execution records stay in the Executions list with their stored name and tags, and the View action on each terminal test row still opens the Historical Test Canvas successfully: the snapshot on the execution record is sufficient to paint the graph, variables, and log slice without any reference to the live test. The only navigation that disables on deletion is the forward-looking one (e.g. "Open this test in the Tests editor"), not the historical view. Suite / regression executions that referenced the deleted test as a child behave identically - the child's View still resolves through the parent's per-test snapshot.

**Open in Workbench**: The Open in Workbench action opens the editable test. The view itself is a read-only window onto the snapshot and state pair the execution recorded.

**Error and Edge States**:

| Condition | Behavior |
|-----------|----------|
| Execution id not found | Popup / empty-state "Execution not found" with link back to the Executions list |
| Source test deleted | No special handling required - the canvas renders normally from the snapshot. No popup. Forward-looking navigation from this page (e.g. to the live test editor) is disabled, but the historical view itself is fully functional. |
| Browser refresh | Re-hydrate the canvas, Variable Manager, and Console from the same fetch used on initial mount; no subscription lifecycle because nothing streams here |

#### 15.6.4.2 Report Viewer

The Report Viewer is the in-app presentation of a generated execution report. It natively renders the canonical JSON report model (see 4.14 Reporting Endpoints) as a document-style page: a detail header over stacked Summary, Timeline, Errors, and Logs sections in a single scroll. The viewer and the exportable HTML report are two renderings of the same canonical model; the viewer follows the application design system while the HTML document is self-contained for sharing outside WAVE.

**Route**: `/reports/{execution_id}`. The viewer is keyed by execution; the latest JSON report version for that execution is resolved server-side. The reports list lives at `/reports` and is reached from the sidebar RESULTS section.

**Format Roles**: JSON is the primary format: opening the viewer for an execution without a JSON report generates it on demand, with determinate progress for suite and regression executions. HTML is produced only by the viewer's Export HTML action. Both formats are one-time per execution - when the artifact exists it is returned, never regenerated, and no regenerate affordance exists anywhere in the UI (regeneration is the artifact-deletion flow described in 4.14).

**Viewer Layout** (decisions RVIEW-1 through RVIEW-9):

| Section | Content |
|---------|---------|
| Header | Back to the reports list, report icon, execution name, entity-type chip, status dot+label; right-aligned actions: Download JSON, Export HTML |
| Summary | "Report generated" timestamp aside; 3x2 metadata grid (Environment / Triggered By / Started, Duration / Execution ID / Ended); count cards - suite/regression count tests (Tests, Passed, Failed, Skipped, Stopped, Excluded, Pending), test executions count blocks (Blocks, Passed, Failed, Skipped, Stopped, Not Run) |
| Timeline | Merged gantt: one row per test run (suite/regression) or per block (test) - status dot, name, time-positioned status-colored bar on a shared axis labeled with the domain start/end timestamps, right-aligned duration. Nested block paths render their container prefix muted. Suite/regression row names link to the run's Historical Test Canvas |
| Errors | One bordered card per failure: level badge (ERROR/FATAL), scope path, right-aligned timestamp, message in error color. Executions without failures show a success-tinted "No errors recorded" state |
| Logs | The merged chronological log stream in console visual vocabulary (time, level, source, message; per-level colors; FATAL row tint) with a level-threshold dropdown and a live visible-entry count |

**Generation Interstitials** (RVIEW-10): while the JSON report is being produced the viewer shows a centered progress card ("N of M runs processed"; indeterminate for single-run test executions). A failed generation shows the failure reason with Try Again and View Execution actions. Opening the viewer for an execution that is still running or paused shows the conflict message with View Execution and Back to Reports actions.

**Reports List** (RVIEW-11, RVIEW-12): execution-centric - one row per execution not currently running or paused, presented with the standard list-page chrome (search, status / type / user / date filters, column visibility, pagination). Columns: Name, Type, Status, Finished, Duration, Report (generated time and size, or "Not generated"), Actions. The row action opens the viewer: View when the JSON report exists, Generate when it does not; the viewer owns generation in both cases.

**Entry Points** (RVIEW-13):

| Surface | Affordance |
|---------|-----------|
| Reports list row | View / Generate action opens `/reports/{execution_id}` |
| Suite / regression Execution Detail header | Report button between Re-run and Export; disabled while the execution is running |
| Test execution Historical Canvas action bar | Report button before Open in Workbench |

**Error and Edge States**:

| Condition | Behavior |
|-----------|----------|
| Execution id not found | Empty-state "Execution not found" with link back to the reports list |
| Execution still running / paused | Conflict interstitial (above); the report becomes available once the execution finishes |
| Execution predates artifact directory allocation | Conflict interstitial carrying the backend message; no report can be produced |
| Generation job lost to a backend restart | The viewer re-requests the report; the persisted artifact is returned when it exists |
| Browser refresh | Re-resolves the report through the same on-demand flow; a completed report loads directly from its artifact |

##### REQ-3.7.2.8 Filter logs by level in report viewer

The viewer's Logs section provides the same level-threshold semantics as the Console (see 15.6.3): a level dropdown over the DEBUG < INFO < WARN < ERROR < FATAL hierarchy showing entries at or above the selected severity, defaulting to INFO, with a live "N of M entries" count of visible rows. The filter operates on the report's persisted log stream entirely client-side; filter state is local to the viewer page and does not affect the Console or other surfaces. The exportable HTML report carries its own embedded equivalent controls (see 4.14.2), so filtering is available both in-app and wherever the document is opened.

#### 15.6.4.3 Artifact Browser

The Artifact Browser is the top-level RESULTS surface for inspecting the files an execution leaves behind. It is execution-first: the landing page lists executions that produced artifacts, and opening one drills into that execution's outputs grouped by meaning. It reuses the application list-page chrome and the shared detail header, so it reads as a sibling of the Executions and Reports surfaces.

**Artifact zone vs persistence zone**: The browser shows only the artifact zone - the files written under each execution's artifact directory. The persistence zone (entity and execution definitions and records) is never shown here. Ephemeral Workbench session runs write to a separate transient area and are excluded from the global browser; they are surfaced inside the Workbench itself.

**Landing list** (ARTIF-1, ARTIF-2): execution-centric, one row per execution that has artifacts, with the standard list-page chrome (search, Type / Status filters, Columns selector, pagination, selection mode). Columns: Execution, Type, Status, Finished, Artifacts (count), Size, Actions. A row's chevron opens the drill; its kebab offers Download (.zip) and Delete artifacts. Selection mode adds a toolbar Mass action menu (Download (.zip), Delete artifacts) over the selected executions' artifacts. Delete removes artifacts only - the execution record itself is untouched.

**Drill view** (ARTIF-3 through ARTIF-12): a detail page under the shared detail header (back to Artifacts, entity icon, execution name, entity-type chip, status; right-aligned Download all (.zip)), a "Finished - N artifacts - total size" meta line, and a Show removed toggle. Artifacts group into four collapsible sections in fixed order:

| Section | Content |
|---------|---------|
| Failure summary | The single error.json causal summary - the error/fatal log entries that caused tests to fail, correlated to the failing block; carries a failure-count badge. Absent for clean runs |
| Reports | report.html and report.json for the execution, each with download / delete |
| Logs | Per-test logs (`logs/<suite>/<test>.log`) shown as a Regression > Suite > Test tree by default, with a Tree / Flat toggle |
| Files | Plugin-written artifacts (File I/O outputs) shown flat by default, with a Tree / Flat toggle |

The Logs tree carries entity icons matching the sidebar vocabulary (regression, suite, test); a test leaf's icon is status-tinted (passed green, failed red) so a failed test never reads as a failed log, and suite/regression nodes summarize "N failed - M tests". Every file row has copy path, download, and delete; suite and regression nodes additionally expose their aggregate size, a subtree download (.zip), and a subtree delete. Long names cap with an ellipsis and reveal the full name in a popover only when clipped.

**Soft-delete and history**: Deleting an artifact (a file, a subtree, or an execution's whole set) frees its bytes but leaves a permanent tombstone record (name, path, size, removed-at, removed-by). Removed rows are hidden by default; the Show removed toggle reveals them muted, as an audit trail. There is no restore - the record documents what was removed; it is not a recycle bin.

##### REQ-5.2.1.41 Artifact browser

The Artifact Browser lists every execution's artifacts and downloads them. Listing is execution-first - the landing list of executions, then the drill's four per-execution sections (Failure summary, Reports, Logs, Files). Download retrieves a single file, a suite/regression subtree as a .zip, or an execution's whole artifact set as a .zip; compressed artifacts are handled transparently on download. WAVE-written artifacts are single-instance (see REQ-6.1.4 and the artifact-persistence design), so there is no version history to compare; replacing an artifact is the delete-then-regenerate flow.

##### REQ-5.2.6.4.8 Bulk artifact operations

The browser supports bulk operations over artifacts. In the landing list, selecting several executions downloads their combined artifacts as a .zip or deletes their artifacts together. In the drill, a suite or regression node downloads or deletes its whole log subtree at once, and Download all packages an execution's complete artifact set. Bulk deletes follow the same soft-delete-with-tombstone model as a single delete and confirm before proceeding. Every delete in the browser confirms first, whatever its size: a single artifact, a whole log subtree, or a selection of executions. The confirmation names what is going and says that removed artifacts stay as an audit record and cannot be restored.

#### 15.6.4.4 Workbench Session Artifacts

The Workbench surfaces the live session's artifacts in place, so a test engineer inspecting a run never has to leave the canvas to reach the files it wrote. Ephemeral Workbench runs write into a transient per-session area that the global Artifact Browser deliberately excludes; this surface is that area's one and only viewer.

**Chip** (ARTIF-14): an "Artifacts" affordance sits on the right side of the canvas breadcrumb bar, beside Reset, sharing Reset's visibility rules (absent on historical and read-only canvases). It carries a count badge whenever the session holds at least one artifact, doubling as the ambient signal that the session owns files worth protecting.

**Session Artifacts sheet** (ARTIF-15, ARTIF-16): the chip opens a floating, draggable sheet in the Workbench's floating-surface family. Its header names the surface and summarizes count and total size, with refresh and close actions. Rows reuse the Artifact Browser drill's file-row vocabulary - file-type tile, monospaced name, folder subtitle for nested paths, size, and per-file copy path, download and delete - so the two surfaces read as one system. Copy path copies the row's artifact:// reference and confirms with a check icon for two seconds, matching the app's copy idiom (TDATA-7). A footer note states the lifecycle contract: session artifacts are temporary and are deleted when the session is reset, replaced, or closed. Before any run has written files, the sheet presents an empty state explaining that plugin-written files appear here. While a run is executing with the sheet open, the listing tracks new files as plugins write them.

**Lifecycle and deletion semantics**: The session area is scratch space. Deleting a session artifact confirms first and then removes the file permanently - no tombstone, no removal history - in contrast to the audited soft-delete of the permanent artifact zone. The confirmation is the same one every other file surface uses, because the file is gone for good either way. A session that tracks a stored execution resolves its artifacts to that execution's permanent directory instead; there the sheet is a read-and-download window, and per-file deletion is refused because those files are governed by the Artifact Browser's soft-delete model. Destroying a session (reset, replacement, close, or switch) destroys its artifacts with it; the confirmation behavior for those gestures is part of the destructive-action dialog design (REQ-5.2.1.23 in the feedback-patterns design).

##### REQ-5.2.1.41 Artifact browser (workbench session surface)

The live session surface lists the active Workbench session's artifact files with their sizes and paths, downloads any file on demand, and deletes individual files. Listing and retrieval always reflect the session's current directory contents, including files written moments ago by a still-running test.

#### 15.6.4.5 Test Data Sheet

The Workbench surfaces a test's own input files beside its artifacts, so the files a test reads and the files it writes are managed from the same bar without leaving the canvas. Test data has no page of its own: the sheet is its only surface.

**Chip** (TDATA-3): a "Test Data" chip with a folder icon sits on the canvas breadcrumb bar left of the Artifacts chip, the same size, with no count badge and the same visibility gating (absent on historical and read-only canvases). It opens the Test Data sheet.

**Sheet** (TDATA-4): a floating sheet of the Workbench's draggable-sheet family, 480 px wide. The header carries a folder icon, the title "Test Data", Upload and New folder as small text buttons, then Refresh and Close icon buttons. A filter row holds "Filter files…" and a "N files - size" summary. The tree starts directly under it with no section headers: folders collapse and show their aggregate size; file rows show the type tile, a monospaced name and the size. Upload and New folder act on the selected folder, or on the root when none is selected; files dropped on the tree upload to the root, and files dropped on a folder row upload into it.

**Rows and Actions** (TDATA-5, TDATA-7): every file row and folder row carries Copy path, Download and a kebab. Copy path copies the row's testdata:// reference, swaps to a check icon for two seconds with a screen-reader announcement and never toasts, matching the copy idiom used across the app; the same control appears before Download on every row of the Artifacts drill and the Session Artifacts sheet, where it copies the artifact:// reference. The kebab of a file and of a folder offers Rename and Delete. Delete asks a simple confirmation; an upload over an existing name asks Replace or Cancel; New folder and Rename share one Name field. A refusal because the test is running shows the server's message inline, at the row that asked, naming the test.

**Empty State, Footer and Unsaved Test** (TDATA-6): before any file is uploaded the sheet reads "No test data yet" over "Upload the files this test reads. Reference them as testdata:// paths where a block reads a file." with an "Upload files" button. The footer of the populated sheet reads "Reference a file as testdata://folder/name. Blocks read test data; they never change it." An unsaved new test has no folder to upload into, so the sheet reads "Save the test first to add files." with Upload and New folder disabled.

##### REQ-3.4.3.2 Read test data files from configured paths (workbench surface)

The Test Data sheet lists the open test's own files as a folder tree with sizes, uploads files and creates folders, renames and deletes entries, downloads a file or a folder as a zip, and copies any entry's testdata:// reference for pasting where a block reads a file. It always reflects the test's folder as it is on disk. Changes are refused while any execution of the test is in progress, and the refusal is shown on the row that asked for it, so the files a running test reads never change underneath it.

#### 15.6.4.6 Test Data Uploads

A file on its way into a test is state of the test, not of one browser tab, and changing a test's files is the holder's right alone (TDATA-13 through TDATA-18).

**Who may change the files** (TDATA-13): only the session holding the test can change its files. A session that is watching keeps Copy path and Download and loses upload, folder creation, rename and delete.

**One shared view of what is arriving** (TDATA-14): every session watching a test sees the same in-flight rows, with the same progress and the same time remaining, because the server works both out once and tells everyone the same figures. An arriving file's row sits in the tree under the folder it is going to, and a closed folder opens while a file lands in it, so where a file is going is never in doubt.

**Chip state** (TDATA-16): the Test Data chip carries a spinner while anything is arriving and a red exclamation mark when something did not land; it can carry both.

**A file that did not land** (TDATA-14, TDATA-17): it leaves a notice. The notice offers Retry where the tab still holds the file and Choose file where it does not. A cancellation is a disposable row; an interruption is kept until it is dismissed or the same file lands.

**Stopping an upload** (TDATA-15, TDATA-18): a cancellation is announced the moment it is asked for, not when the transfer finally stops, and the sending session stops pushing as soon as it hears. Taking the test over never kills a transfer that is already running, and only the holder and the session that started an upload can cancel it. Half-written bytes stop holding their name after five minutes.

#### 15.6.5 Debug Mode (Ephemeral Workbench)

Debug Mode is the Workbench-only interactive execution mode that lets a test engineer observe a running test at block granularity: pausing on breakpoints, pausing on errors, pausing at the first block, and continuing or skipping from a paused state. Debug Mode is strictly a Workbench capability - it is never offered on tracked executions (Entity List / Entity Detail runs), so the Executions list, Execution Detail view, Historical Test Canvas, and Workbench Tracked Execution Viewer never render Debug affordances.

##### REQ-3.2.1.2 Debug mode with breakpoint support

**Entry Points**: The Workbench canvas toolbar exposes a Debug split button alongside Run (per RUN-5/6). Primary click launches a debug run on the currently-opened scope; the dropdown chevron shows scope options matching the Run button pattern (Debug Scope, Debug All). A gear icon to the left of the Debug button opens the Debug Options popover (DEBUG-4/5). The canvas right-click context menu on a block contributes per-element entries: Debug Current, Run Current, Debug from Here, Run from Here. The canvas right-click context menu (on the empty canvas) contributes Clear Breakpoints plus Pause on Error and Pause on First Block toggles that mirror the Debug Options popover.

**Ephemeral-Only**: Debug Mode applies only to ephemeral Workbench execution. The tracked-execution path (RunDialog, POST /executions, Execution Detail) has no Debug affordance and no Debug Options. A tracked execution cannot be paused by a breakpoint, cannot honor a Pause-on-first-block option, and will not pause on an error - it either runs to its terminal status or is Stopped by the user. This separation keeps the execution-history surface deterministic and the ephemeral surface deeply introspectable.

**Debug Options Popover** (DEBUG-4 through DEBUG-8, RUN-10 through RUN-12):

| Option | Default | Effect |
|--------|---------|--------|
| Pause on first block | Off | Engine pauses BEFORE the first executable block is run. Applies only to Debug runs. |
| Pause on errors | Off | Engine pauses at the failing block the moment an exception is raised, BEFORE any try-catch handler in scope runs. One failure causes one pause: a container or Function instance that fails because of it does not pause again. Applies only to Debug runs. |

Both options persist per user per browser in localStorage. The popover is right-aligned below the gear, and the gear itself renders with the debug-blue accent when either option is active (DEBUG-4).

**Debug Request**: When a Debug run is initiated, the Workbench submits the same run request shape as a Run, with two additional fields:

| Field | Source | Purpose |
|-------|--------|---------|
| `breakpoints` | Frontend workflow store (set of node ids with breakpoints) | The list of blocks on which the engine pauses BEFORE execution |
| `debug_options` | Debug Options popover state | `pause_on_first_block`, `pause_on_errors` |

There is no separate "debug mode" flag on the wire. Run and Debug share the same endpoint; the distinction is carried by the payload itself: a regular Run sends neither `breakpoints` nor `debug_options`, and the engine runs to completion without any pause machinery; a Debug Run sends both (the breakpoint list may be empty if the user is using only the pause-on-first-block or pause-on-errors options), and the engine runs the pause-point evaluator against every block. Neither field is persisted on disk; both live only in the active Workbench session's in-memory state for the lifetime of the run.

##### REQ-3.2.2.1 Set breakpoints before/during execution

**Breakpoint Ownership**: Breakpoints are owned by the frontend. They are a per-open-test in-memory set on the Workbench workflow store (the existing `breakpoints: Set<string>` already in the store). The set persists for as long as the test is open in the Workbench - across panning, zooming, editing, running, and pausing. It clears when the user closes the test (navigates away to a different test, a different surface, or explicitly discards the draft). Breakpoints are not persisted to the test definition, to localStorage, or to the backend - they are an ephemeral debugging artifact that belongs to the person debugging, not to the test asset.

**Setting a Breakpoint (pre-run)**: The user opens the block's context menu or action row (CANVAS-143) and picks "Toggle Breakpoint". A red dot renders at the top-right corner of the block (CANVAS-82). Toggling again removes the breakpoint.

**Setting a Breakpoint (during a Debug run)**: The same toggle affordance is available while a Debug run is live (Running or Paused). When toggled during a live run, the Workbench sends a PATCH to the workbench session's breakpoint endpoint. The backend updates the in-engine breakpoint set and the next not-yet-executed block that hits a breakpoint will pause the engine. Blocks that have already executed earlier in the same run are not re-evaluated.

**Breakpoint Visual**: A small filled red disc at the top-right of the block, overlapping the block border. Visible in all canvas states (idle, running, paused, terminal). Paused blocks that also carry a breakpoint render both the amber paused border (CANVAS-81) and the red breakpoint dot.

##### REQ-3.2.2.2 Skip blocks during runtime

Runtime block skipping is an existing, pre-run capability: a user marks blocks as skipped from the canvas action row or context menu, and the engine treats them as bypassed when it encounters them. Debug Mode extends this with an in-pause Skip action:

**Skip-at-Pause**: While the engine is paused at a block X (breakpoint, pause-on-first-block, or pause-on-errors), the Paused toolbar exposes a Skip Block control. Clicking it marks X with runtime status `skipped` (the amber-striped border per CANVAS-72), advances the engine past the block, and pauses BEFORE the next block. Skip does NOT unwind any variable or side effect produced by prior blocks.

**Skipping a block that has a result**: Skip applies the same way to a paused block that already has a result from an error pause or from Repeat Step. The block is marked skipped and the flow continues on its normal path: no catch path runs for the earlier failure. Whatever the block or its repeats already changed (variable writes, requests sent, files changed) stays.

**Control-flow parents**: For a parent with a single post-control egress (the 'then' handle), Skip marks the parent skipped, cascades the skipped visual to every block on its inner body or branch paths, and pauses on the first block of the 'then' egress. For a parent whose egresses are mutually exclusive with no unambiguous sibling-after (If-Else, Assert-Split, Switch), Skip is rejected: the Paused Toolbar disables the control and the engineer must step into a branch and skip a leaf there instead.

##### REQ-3.2.2.3 Pause/resume execution

Debug Mode introduces three independent triggers that move the engine from Running to Paused:

| Trigger | When |
|---------|------|
| Breakpoint hit | Engine reaches a block whose id is in the live breakpoints set. It pauses BEFORE the block starts executing, before any of the block's plugin code runs and before any output bindings are resolved. |
| Pause on first block | If the `pause_on_first_block` option was set at Debug start, the engine pauses BEFORE the first executable (non-Start) block of the workflow, regardless of whether that block has a breakpoint. |
| Pause on errors | If the `pause_on_errors` option was set at Debug start, the engine pauses at any block whose execution raises an exception. The pause happens AFTER the exception is raised but BEFORE any try-catch handler in the enclosing scope runs. The failed block already has its result, so Continue and Step Over do not run it again: they follow failure routing (REQ-3.2.1.4 Commit semantics) and never pause again on that failure, including at an enclosing container or Function instance that fails because of it. |

**Paused-state Contract**: While paused, the engine holds its asyncio run task in a wait state on a resumption signal. No further blocks execute. Variable values last written by completed blocks remain in the in-memory variable store and are visible in the Variable Manager. Log entries continue to appear on the Console for any events that happened just before the pause. The WebSocket emits a `debug_breakpoint_hit` event carrying `node_id`, pause reason (`breakpoint` / `first_block` / `error`), and (for errors) the exception class and message.

**Paused Canvas Visual** (CANVAS-79 through CANVAS-88):

| Element | State while Paused |
|---------|-------------------|
| Breadcrumb status | `● Paused` in amber |
| Paused block | Amber border (3-4px), no selection halo; breakpoint dot continues to render if present |
| Other blocks | Idle/completed/skipped visuals per their run status |
| Palette | Visible and expanded but disabled (gray) |
| Variable Manager | Auto-expanded |
| Console | Emits `⏸ Paused at [Name] (breakpoint)` / `(pause on first)` / `(pause on error: <msg>)` in amber |
| Console height | 140px when Vars expanded |

**Paused Toolbar** (CANVAS-80): Step Over, Step Into, Repeat Step, Continue, Skip Block, Stop, Focus Current. The Stop button carries the tracked-execution stop-confirmation policy only when the Workbench is acting as a tracked-execution viewer; for ephemeral Debug runs Stop is unconfirmed (the rapid-iteration policy from 15.6.0.3 applies).

**Continue**: Releases the resumption signal; engine resumes execution from the paused block onward. Execution runs until the next trigger (breakpoint, pause on errors, or terminal status). A Debug run with no remaining triggers runs to the same terminal outcomes as a regular run: `passed`, `failed`, `stopped`, `skipped`.

**Selection during Pause** (CANVAS-83, 84): Paused and Selected are different states. The user can select a different block while paused; selection gives access to a limited set of actions appropriate to the debugging context (inspect config, toggle breakpoint, toggle skip). Structural edits (drag, connect, add, delete) are not available while an execution is live, matching the existing Workbench contract.

##### REQ-3.2.2.4 Cancel execution with cleanup

The Paused-state toolbar exposes the same Stop control as the Running-state toolbar, with identical confirmation policy (unconfirmed for ephemeral Workbench runs).

##### REQ-3.2.2.5 Real-time progress monitoring

Debug Mode reuses the real-time progress infrastructure described in 15.6.1 (live flow visualization, block highlighting, breadcrumb status). The only Debug-specific addition is the `debug_breakpoint_hit` event, which carries the node id, pause reason, and (on error pauses) the exception class plus message. The event drives the Paused canvas visual transitions enumerated above.

##### REQ-3.2.1.3 Step-by-step execution

The Paused Toolbar exposes Step Over and Step Into using standard debugger semantics.

**Step Over**: Runs the paused block to completion and pauses before its sibling-after. On a control-flow parent, the entire sub-graph under the parent runs atomically (all loop iterations, every try-catch path, the taken branch of a divergence) before the engine pauses on the first block past the parent's post-control egress. A paused block that already has a result - it failed and caused an error pause, or Repeat Step ran it - is not run again: Step Over proceeds from that result and pauses before the next block the run reaches. A passed result goes on to the sibling-after; a failed result follows failure routing, and the pause lands on the first block that routing runs (the first block of a surrounding catch path).

**Step Into**: On a parent with an inner body (loops, try-catch, parallel), pauses before the first body block. On a parent that only evaluates and routes (If-Else, Assert-Split, Switch), pauses on the first block of the selected egress. On a leaf, collapses to Step Over. On a control-flow parent whose body is empty at runtime (For-count=0, ForEach over empty, while-false-on-first), also collapses to Step Over. On a container or a Function instance, Step Into always enters and runs the body step by step, even when the block already has a result from Repeat Step or an error pause; the outcome of that stepped run replaces the earlier result. On a leaf that already has a result, Step Into behaves as Step Over and does not run the block again.

##### REQ-3.2.1.4 Block re-execution during pause

The Paused Toolbar's Repeat Step control re-runs the currently paused block in place: the engine executes the block's plugin code against the session's live variable state, commits output bindings, and re-pauses on the same block. The block's canvas badge updates to the re-run's terminal status (passed or failed) while the amber paused border (CANVAS-81) remains, so both the outcome and the paused state are visible. The user may repeat any number of times, or advance via Continue / Step Over / Step Into / Skip Block. A repeat may substitute one parameter value of the paused block for that single execution, which is how the Python editor's Run tries an unsaved script while a debug run is paused at that block (PYRUN-21); the stored test is untouched. With no debug run paused at the block, the editor's Run is a Run Current of that block alone, and it runs exactly what the block's Save would save: the configuration sheet's whole unsaved draft, every editor open from it, and this editor's script, for that one run only and never saved. While something other than this script would block that sheet's Save, Run is disabled and says so with the same note.

**One pause**: A repeat runs the block through without pausing inside it: breakpoints and pause on errors within a repeated container do not fire. A repeat that fails, during any kind of pause, stays in the same single pause: the block shows failed and the user may repeat again or advance.

**Commit semantics**: Each repeat replaces the block's result. On advance, Continue and Step Over do not run the block again: the engine uses the latest result and its signals to drive successor routing (try-catch selection, diverging-branch selection). Intermediate attempts are not aggregated. A latest result that passed goes on down the normal path; a latest result that failed follows failure routing - a surrounding try-catch runs its catch path, otherwise the flow ends as with any failed block - and does not pause again on that failure. A try block whose repeat passes does not trigger its catch on Continue; a try block whose repeat fails again does. A block paused on an error has a result from the start, so Continue and Step Over take the failure path unless a repeat during the pause passed. Step Into on a container or a Function instance is the one advance that runs the block again, step by step (REQ-3.2.1.3). Skip Block marks the block skipped whatever its result (REQ-3.2.2.2).

**Every sub-flow**: These rules hold wherever the paused block sits - the main flow, try-catch paths, parallel branches, loop bodies, Wait Until bodies, Switch cases, Function bodies and Remote Execution bodies - and in Debug Current and Debug from Here runs.

**Writes stick**: Variable writes, external HTTP calls, file I/O, and other plugin side effects produced by Repeat Step are permanent. The UI does not sandbox or roll back. Blocks that mutate external state should be re-run deliberately.

**Availability**: Repeat Step is active on the Paused Toolbar while the session is paused, regardless of the pause reason (breakpoint-before, first-block, step, or error). It is disabled on the Running-state toolbar. The button has no effect when there is no block on which to re-run (paused_node_id is null).

**Events and Console**: On click, the Console emits `⟳ Repeated '[Name]' - [status]` at INFO (passed), WARN (skipped), or ERROR (failed) level depending on the re-run's outcome. The WebSocket `debug_breakpoint_hit` event after the re-run carries the updated `reason`.

##### REQ-3.2.1.6 Runtime variable override during pause

While a Workbench debug session is paused or stopped - any state other than actively running - the Variable Manager lets the engineer change a variable's current runtime value so the change takes effect on the continued or re-run execution. This is the debugging complement to Repeat Step (REQ-3.2.1.4): where Repeat Step re-runs a block against the live variable state, a runtime override edits that live state directly, after which Continue, Step, or a partial re-run proceeds with the new value.

**Where it applies**: The override acts on the session's live variable space - the same realized values the Variable Manager displays and the engine reads. It is reached from the variable's window, opened from the row pencil, with the value switch on Current (VMGR-108); a Current value of a Function's own input, output or local lands in that Function's scope. It is offered only while the engine is not actively executing (paused, stopped, or before the first run); while a run is actively in flight the affordance is disabled, because a value in motion has no stable point for an override to land. A breakpoint provides that stable point.

**What changes**: A literal variable takes a new typed value. An expression variable's source is changed and re-armed, so it re-evaluates the next time the run references it; the variable never stops being an expression. Until a run evaluates the new source, the Variable Manager shows no result for the variable: the result of the source it replaced is dropped. The current-value edit never alters the saved definition - editing the definition (the Initial value) is the separate, persisted change.

**Lifetime**: An override behaves like any other runtime value. It persists across a Stop and across subsequent partial executions (run-from-here, single-block) that inherit the session's variable state. A full run discards it by re-seeding every variable from its definition. Reset clears it. An expression or template source set by a current-value edit is carried the same way, whether or not a run has evaluated it yet: it stays the variable's source for the ad-hoc runs that follow, and its result is evaluated again when a variable it reads is edited. A current-value edit of a variable makes the next ad-hoc run evaluate again every computed variable that reads it; in a paused run, the results that run already holds stay as they are until it ends.

**Pre-run staging**: Before any run exists, the same Current edit in the window stages a transient value that the next run - including a single-block or partial run - starts from, without modifying the saved definition, so an engineer can seed a value for an isolated block run. The variable's row in the Variable Manager shows the staged value - for an expression or a template, the staged source - because that is what the next run uses. For a staged value, the row's definition view still shows the saved definition. A value staged for a Function's own input, output or local is kept in that Function's scope.

**Values the live run does not hold yet**: A Current value of a variable added after the run started, or of another type or kind than the live run declared, is staged for the next run, as before a run, because the live run keeps its own declarations (VMGR-19); a variable that took the name of one renamed since the run started is such a new variable. An undo or redo of a definition carries its staged Current value along: a rename moves it, and a removal, a retype or another kind drops it. A Current value of a Function the live run has not started yet is held for that Function and taken when it starts in this run, also when an earlier run left a scope for it; an ad-hoc run started from the session carries it while the Function still has not started. An input keeps the value its Function Block gives (REQ-3.1.2.10), so a held input value counts for a run of that Function on its own. Both are checked against the variable's declaration, name and type, as a live edit is.

**In-flight resumption**: If a run is resumed or started while a variable's window holds unsaved changes, the engineer is prompted before the run proceeds, with the choice to cancel, drop the pending edit, or apply it first.

##### REQ-3.2.1.5 Ad-hoc block execution

The right-click context menu on a canvas block exposes a single-element execution affordance under two variants: **Run Current** (no pauses) and **Debug Current** (honors breakpoints and the Debug Options pause flags). Both execute the right-clicked element in isolation from the rest of the workflow, sharing the same ephemeral Workbench session machinery as a full debug run, and producing the same `block_status_changed` / `execution_completed` / `variables_updated` / `debug_breakpoint_hit` event surface so the canvas, Console, and Variable Manager update without any new event wiring.

**Sub-tree boundary**: A single-element run executes the chosen block's structurally self-contained sub-tree and stops at that sub-tree's after-egress. Three block shapes determine the boundary:

| Block shape | What runs | Stops at |
|------------|-----------|----------|
| Atomic (plugin, Variable Setter, Halt, single-egress Assert) | The block itself only | The block's terminal egress |
| Composite single-egress (For, While, ForEach, TryCatchFinally, Parallel, single-egress Assert with body) | The block + its full internal sub-tree (every loop iteration, try + optional catch + finally, every parallel branch, etc.) | The control's after-egress (the engine does not enter the downstream sibling on the canvas) |
| Branching (IfElse, SwitchCase, Assert-split) | The block + the chosen branch's full sub-tree | The leaf of each chosen branch (no traversal past the branch into whatever block would come after the control) |

The boundary is enumerated structurally, not by execution outcome - the engine knows the boundary before traversal begins because it is derivable from the workflow's adjacency and block-shape metadata.

**Eligibility**: The Run Current / Debug Current items are offered on every plugin block, every saved block, and every control EXCEPT three: Break, Continue, and Note. Note is non-executable; Break and Continue are flow-control signals whose semantics are only defined inside their parent loop, so single-element execution outside that context is rejected. The menu items render disabled (visible, grayed) on those three block types so the affordance is discoverable but not actionable. The same items are also disabled while the Workbench session is `running` or `paused`, while the block is in a `disabled` or `skipped` state, and on stale or transient render states - matching the existing destructive-action gating pattern on the same context menu.

**Inside a Function body**: Run Current and Debug Current are offered on a block inside an opened Function body at any depth, including the read-only inspect view of a Published Function. The run identifies the block by the path of Function instances from the Test canvas plus the block's id, and runs the block's sub-tree inside that Function instance.

**Variable bootstrap**: Every ad-hoc run - Run Current and Run from Here at any level, Run Scope inside a Function body, and their Debug variants - starts from what the Workbench session holds now: the Test's variable values, the block states and the open SSH sessions as the most recent run left them, and for a Function body the instance's inputs and outputs as its most recent run left them, with the values the session gave its local variables. The session holds a value for a Test variable once something in the session gave it one: a block wrote it, an expression or a template was evaluated, or a current-value edit set it. It holds a value for a Function's input, output or local variable once a block in that Function wrote it or a current-value edit set it in that Function's scope. An evaluated result is held only while what it was computed from stays as it was: when the engineer edits the variable's source, or a variable it reads - in the definition or with a current-value edit, directly or through another computed variable - the next ad-hoc run evaluates it again, and the Variable Manager drops the old result at the edit. A result nothing touched is not evaluated again, so a time or a random value stays the same from one block run to the next. A value a block wrote into the variable itself, and a current-value edit of its value, still win. Nothing is evaluated again inside one run: a result the run already holds stays as it is when a block of that run writes a variable it reads. A Test variable, or a Function's local variable, the session gave no value has none of its own and follows its definition as it stands when the ad-hoc run starts, so a value edited after the last run is used by the next ad-hoc run, and a variable with no value holds null until it is given one. The Variable Manager shows a live value only for a variable the session gave one, so the row of a variable that follows its definition shows that definition. A variable an earlier run of the session marked secret as it ran stays secret, with the value it carries, in the ad-hoc runs that follow. Starting an ad-hoc run resets nothing, and what it produces is where the next ad-hoc run starts. Reset is the way to start clean: after Reset, or before any run in the session, the Test's variables start from their definitions and a Function body's inputs and local variables start from their declared defaults, with the advisory note that prior-run state is unavailable. Because WAVE's Variable Manager validates every input-parameter and expression-field reference at design time, a runtime "undefined variable" cannot arise from a saved test. Variable values may be defaults or stale relative to what an upstream block would have produced in a real run; that is the user's call to make at the moment of triggering the run.

**Result surface**: A single-element run does not navigate or open a modal. The canvas pulses on the touched block (running glow, then completed/failed border state) via the same `block_status_changed` events; the Console panel receives any logs the plugin emits; the Variable Manager updates any variables the plugin writes; on terminal status, a toast surfaces "Block <name> ran successfully" or "Block <name> failed: <one-line error>". The session is ephemeral - no Execution record is created and no entry appears in Execution History.

**Concurrency**: Only one Workbench session is active at a time. While a session is running or paused, the Run Current / Debug Current menu items (and the Run from Here / Debug from Here items below) are disabled. The user must Stop or Continue to terminal before another single-element or partial-graph run can be initiated.

##### Partial-graph execution: Run from Here and Debug from Here

The right-click context menu on a canvas block also offers **Run from Here** (no pauses) and **Debug from Here** (honors breakpoints and Debug Options). These are the partial-graph variants: the engine starts traversal at the chosen block and continues forward through every reachable downstream block until the test would naturally terminate. Blocks upstream of the chosen start block never execute.

**Differences from single-element execution**: Where Run/Debug Current is bounded by the chosen block's own sub-tree and stops at its after-egress, Run/Debug from Here continues PAST the after-egress and follows every outgoing edge to the end of the workflow. Same eligibility rules (Break / Continue / Note disabled), same ephemeral session model, same state carried in from the session (see Variable bootstrap above), same WS event surface, same toast-on-terminal pattern.

**Inside a Function body**: Run/Debug from Here on a block inside an opened Function body, at any depth, starts at that block inside that Function instance and runs through the rest of that Function body; it does not continue into the flow the Function Block sits in.

**Pause semantics for Debug from Here**: Existing breakpoints on any reachable downstream block still trigger pauses normally. When the Debug Options popover's Pause-on-first-block flag is enabled, the engine pauses AT the chosen start block (the first block of THIS run, not the workflow's actual Start node). When Pause-on-errors is enabled, error pauses fire normally throughout the run. Run from Here ignores all pause flags by definition - it is the no-pauses variant.

**Skipped and disabled blocks encountered during traversal**: Existing canvas-state semantics carry over. A `disabled` block downstream of the chosen start is bypassed (no-op pass-through). A block already marked `skipped` from a prior session has no special meaning at the start of a new ephemeral session.

##### Canvas Debug Context Menu

Right-clicking on the empty canvas (not on a block) exposes three new debug-related items in addition to whatever items existed before:

| Item | Type | Behavior |
|------|------|----------|
| Clear Breakpoints | Action | Empties the workflow's breakpoint set. Disabled when no breakpoints are set. If a Workbench session is currently active, the change is also pushed to the running engine via the existing PATCH `/workbench/{session_id}/breakpoints` endpoint with an empty list, so already-armed breakpoints stop pausing the engine on subsequent blocks. |
| Pause on Error | Toggle | Mirrors the `pause_on_errors` flag from the Debug Options popover. Showing check-state in the menu makes the current value scannable without opening the popover. If a Workbench session is currently active, the change is also pushed via the existing PATCH `/workbench/{session_id}/debug-options` endpoint so the running engine picks up the new value for subsequent error events. |
| Pause on First Block | Toggle | Mirrors the `pause_on_first_block` flag from the Debug Options popover; same check-state behavior and same live-session push pattern as Pause on Error. Note that toggling this during a live run only affects future runs; the engine has already passed (or paused at) the first block by the time the toggle is reached during an active session. |

These items remain enabled at all times (even during a running or paused session) because they are pure preference toggles - the Clear Breakpoints item is the only one of the three that is conditionally disabled (when the breakpoint set is empty). They are appended to the bottom of the existing canvas context menu with a separator above so they group cleanly with each other and remain visually distinct from any pre-existing canvas menu items.
