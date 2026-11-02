#### 3.7.10 Loop Controls

##### REQ-3.5.1.15 For loop with counter variable

The For loop control provides count-based and range-based iteration within visual workflows, executing a body chain of nodes repeatedly with a counter variable that increments on each iteration.

**Control Purpose and Use Cases**: Repeating a sequence of operations a deterministic number of times. Stress/consistency testing (repeat HTTP requests), data generation (create N records), numeric range traversal (ports 8080-8085), and parameterized test variations.

**Iteration Modes**: Two modes selected via a configuration radio:

- **Count mode**: Iteration count provided. Sequence: 0 (inclusive) to count (exclusive) with configurable step.
- **Range mode**: Explicit start and end. Sequence: start (inclusive) to end (exclusive) with configurable step.

Both modes share step (default 1) and timeout parameters. All iteration values are integers. The count is a data field: a null count runs zero times, the same "holds nothing" rule checks use (see REQ-3.1.3.25). A loop that runs zero times paints every block of its body skipped (Branching Engine, Lanes not run). In range mode the start, end and step are settings, not data fields, so a null start, end or step still fails the block and names it (REQ-3.1.3.21).

**Parameter Schema**:

| Parameter | Type | Required | Default | Visibility | Group | Description |
|-----------|------|----------|---------|-----------|-------|-------------|
| `mode` | string enum (count, range) | yes | count | always | - | Iteration mode |
| `count` | integer | yes (count) | - | mode = count | - | Number of iterations |
| `start` | integer | yes (range) | - | mode = range | - | Starting value (inclusive) |
| `end` | integer | yes (range) | - | mode = range | - | Ending value (exclusive) |
| `step` | integer | yes | 1 | always | Advanced | Increment per iteration |
| `timeout` | float | no | - | always | Advanced | Max execution time in seconds |

**Output Variables**:

| Output | Type | Description |
|--------|------|-------------|
| `index` | integer | Current iteration value |

##### REQ-3.5.1.16 For Each loop over collections

The For Each control iterates over collection elements (arrays, maps, tables), executing a body chain for each element with a typed item envelope and iteration index.

**Control Purpose and Use Cases**: Iterating over dynamic collections - server lists, API response arrays, configuration maps, test data tables. The test engineer selects a collection variable and the control iterates its elements, exposing each as a typed envelope to body nodes.

**Supported Collection Types**: Three iterable types, matching WAVE's variable type system:

- **Array**: Iterates over elements sequentially. Item envelope contains each element with its inferred type.
- **Map**: Iterates over root-level keys. Item envelope contains each key as a string.
- **Table**: Iterates over rows. Item envelope contains each row as a map of column values.

Empty collections produce zero iterations; the body paints skipped and execution continues via the then-path. A null collection also runs zero times, and a null item inside a list reaches the body as null, never as text (see REQ-3.1.3.25). After zero runs the item variable keeps the value it had, wherever the loop sits - in the main flow or inside a loop, Switch, Try/Catch, Parallel or Function.

**Collection Input**: The collection parameter uses a variable-reference combobox filtered to iterable variable types (array, map, table). The combobox shows available variables with type badges. An expression toggle switches to a full expression editor for computed collections (e.g., sorted lists, filtered subsets, API response fields).

**Typed Item Envelope**: Each iteration's item is wrapped in a self-describing map with `type` (WAVE type string) and `value` (the actual element). This provides a consistent output structure regardless of collection content. The item output type is `map`.

**Parameter Schema**:

| Parameter | Type | Required | Default | Visibility | Group | Description |
|-----------|------|----------|---------|-----------|-------|-------------|
| `collection` | variable ref (array, map, table) | yes | - | always | - | Collection variable to iterate over |
| `timeout` | float | no | - | always | Advanced | Max execution time in seconds |

**Output Variables**:

| Output | Type | Description |
|--------|------|-------------|
| `item` | map | Current item envelope: {"type": "...", "value": ...} |
| `index` | integer | Current iteration index (0-based) |

##### REQ-3.5.1.17 While loop with condition

The While loop control provides condition-based iteration within visual workflows, repeating a body chain of nodes while a boolean condition remains true. It supports two evaluation modes - pre-check and post-check - covering both standard while-loop and do-while-loop semantics.

**Control Purpose and Use Cases**: Repeating operations until a dynamic condition is met. Polling/waiting patterns (wait for a service to become healthy), convergence checks (retry until error rate drops below threshold), queue draining (process items until queue is empty), and conditional retry loops (re-attempt until success or limit reached).

**Evaluation Modes**: Two modes selected via a configuration dropdown:

- **While-Do** (pre-check, default): Condition evaluated before each iteration including the first. If false on the first check, body executes zero times, its blocks paint skipped, and flow proceeds to the then-path. Standard while-loop semantics matching every programming language's `while`.
- **Do-While** (post-check): Body executes first, then the condition is checked. The body always runs at least once. Useful when the first iteration produces the state that the condition evaluates - for example, making an HTTP request before checking the response status.

**Condition System**: The condition uses the same assertion evaluator as If/Else branching. Conditions are an ordered list of assertion items interleaved with AND/OR operators:

- **Literal assertions**: Variable comparison with operator and reference value (e.g., pod_status not_equals "Running").
- **Expression assertions**: Freeform Python expression evaluating to boolean.
- **Groups**: Nested assertion lists for complex precedence.
- **Operators**: AND (higher precedence) and OR connecting assertion items.

The config modal renders the same visual condition builder used by If/Else, giving test engineers a familiar interface for constructing boolean logic without writing raw expressions. A condition row on a null value follows the check rule of REQ-3.1.3.25.

**Safety Guards**: Two optional limits prevent runaway loops:

- **Max iterations**: When the iteration count reaches this limit and the condition is still true, the loop terminates and the session fails. This is a hard failure - hitting the limit means the expected condition was never satisfied.
- **Timeout**: When execution time exceeds this limit, the loop terminates and the session fails. Same failure semantics as max iterations.

Both guards are optional with no default values, placed in a collapsible Advanced configuration group. Test engineers set these explicitly when building polling or retry workflows.

**Parameter Schema**:

| Parameter | Type | Required | Default | Visibility | Group | Description |
|-----------|------|----------|---------|-----------|-------|-------------|
| `mode` | string enum (while-do, do-while) | yes | while-do | always | - | Evaluation mode: pre-check or post-check |
| `conditions` | array (assertion items) | yes | - | always | - | Condition to evaluate (same format as If/Else) |
| `max_iterations` | integer | no | - | always | Advanced | Maximum iteration count (session fails if reached) |
| `timeout` | float | no | - | always | Advanced | Max execution time in seconds (session fails if reached) |

**Output Variables**:

| Output | Type | Description |
|--------|------|-------------|
| `index` | integer | Current iteration counter (0-based, updates each iteration) |

---

##### Common Loop Behavior

The following applies to all loop controls (For, For Each, While).

**Path Architecture**: All loops follow the scoped dead-end chain pattern from try/catch. Two named output handles:

- **Body** (cyan): Dead-end chain executed on each iteration. Engine traces and re-executes the chain per iteration.
- **Then** (gray): Egress handle. Only exit point. Engine follows this after the loop completes normally.

**Scope Isolation**: Same rules as try/catch path scopes. Body-to-outer, outer-to-body, and cross-scope edges rejected. Same-scope edges (branching within the body) allowed. Rejected edges show the red indicator. Scope tracking reuses path color tag infrastructure.

**Branching Inside a Body**: A branching control in the body (If/Else, split Assert or Switch/Case) routes exactly as it does at test level. The branches not taken paint skipped, but a block that the taken branch also reaches - the point where branches rejoin a shared continuation - runs as part of the taken path. Skipping stops where the two paths merge, so a body that forks and rejoins executes its continuation on every iteration regardless of which branch fired.

**Canvas Rendering**: Two bottom handles: body (cyan, left) and then (gray, right). Body-chain nodes inherit cyan left border. Each loop type has a distinct Lucide icon: For (Repeat), For Each (IterationCw), While (RefreshCw). All share the cyan loop category color.

**Error Handling**: Body node failure aborts the loop immediately. Remaining nodes, remaining iterations, and then-path are skipped. Session fails. For resilience, wrap body nodes in try/catch.

##### REQ-3.5.1.2 Break current block execution
##### REQ-3.5.1.3 Continue to next iteration

**Break and Continue** are loop modifier controls - separate sequential nodes placed inside loop body chains. They carry no condition logic and no expression evaluation. They are pure signal controls: when execution reaches a Break or Continue node, the signal fires unconditionally.

- **Break**: Exits the target loop immediately. Remaining body nodes and remaining iterations are skipped. Execution continues via the target loop's then-path.
- **Continue**: Skips the remaining body nodes in the current iteration of the target loop. The target loop advances to its next iteration (or completes if no iterations remain).

**Through a Try/Catch**: A Break or Continue inside a Try/Catch path (try, catch or finally, at any depth inside it) reaches its target loop the same way. The rest of that path does not run, the catch path does not run because of it, and every finally path the signal crosses runs once, innermost first, before the target loop acts. A loop between the Try/Catch and the target does not consume the signal. If a finally path fails while carrying the signal, the failure wins: the Try/Catch fails, and the loop holding it fails as for any body failure (Try/Catch Control, REQ-3.2.2.10).

**Target Loop Selection**: Break and Continue have a single required parameter: `target_loop`, which stores the node ID of the loop to control. This enables multi-level break/continue in nested loop scenarios.

The config modal renders this as a dropdown populated by traversing the workflow graph upstream from the break/continue node through body-handle edges, collecting enclosing loop nodes. Only ancestor loops reachable via body handles are included - no southbound, eastbound, or parallel-branch loops appear. The dropdown label format is "[Type]: [Name]" (e.g., "For Each: Iterate Servers") or "[Type]: No Title" for unnamed loops. The innermost enclosing loop is pre-selected by default.

When placed outside any loop body, the dropdown is empty with "No loops detected" text. Since `target_loop` is required, workflow validation fails, preventing execution of invalid workflows.

**Parameter Schema**:

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `target_loop` | string (loop node ID) | yes | Node ID of the enclosing loop to target |

**Output Variables**: None.

**Multi-Level Signal Propagation**: When a break or continue targets a loop that is not the immediately enclosing loop, the signal propagates upward through intermediate loops. Each intermediate loop exits its iteration cycle and, instead of queueing its then-path successors, stores the signal and target in the session for the next enclosing loop to consume. This continues until the target loop is reached, which then processes the signal normally (break: queue then-path; continue: advance iteration).

**Backend Validation**: The execution engine validates that `target_loop` references a real loop node ID that is an upstream ancestor of the break/continue node. Invalid references produce a validation error.

**Canvas Rendering**: Standard sequential control nodes with cyan loop category color. Break uses the LogOut icon; Continue uses the SkipForward icon. Single input and output handles. No special rendering treatment beyond the standard control node appearance.

##### REQ-3.5.1.19 Optional rate limiting in loops

**Rate Limiting**: Rate limiting between loop iterations is achieved by placing a Delay control inside the loop body chain. The Delay control's configurable duration provides per-iteration throttling without requiring a dedicated rate limit parameter on the loop itself.

**Iteration Logging**: Loop iteration events are broadcast at DEBUG level to avoid console spam in production workflows. Test engineers who need per-iteration visibility place Log nodes inside the body chain.
