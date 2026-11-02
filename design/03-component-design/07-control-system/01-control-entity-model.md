#### 3.7.1 Control Entity Model

Controls represent a distinct entity type that modifies execution flow rather than performing computational logic. This fundamental distinction from plugins and blocks drives the control system architecture: controls determine what executes, when, how many times, and under what conditions.

##### REQ-3.5.0.1 Controls as distinct entity type separate from plugins and blocks

Controls occupy a unique position in WAVE's component hierarchy. Plugins serve as atomic executors (SSH, HTTP, SQL), Functions serve as reusable compositions, and controls serve as flow modifiers that alter the execution path.

**Execution Semantics**: Plugins follow input-process-output. Controls modify the execution graph topology at runtime. A conditional creates branching paths, a loop causes repeated execution. This semantic difference requires controls to be modeled as a separate entity type.

**Fixed Control Set**: Controls are a closed set of built-in constructs: conditional branching (if/else, switch, assert), iteration (for, for-each, while, break, continue), flow modification (parallel, try-catch, halt), and timing (delay, wait-until). Both frontend and backend share hardcoded definitions, eliminating discovery API complexity while ensuring type safety.

##### REQ-3.5.0.2 Control data model with id, version, name, description, category

**Identity**: Each control has a unique identifier from a fixed enumeration (if-else, for-loop, delay, assert). Enumerated identifiers provide compile-time validation.

**Version**: Controls support semantic versioning enabling tests to pin specific versions. Existing tests continue using pinned versions while new tests adopt updates.

**Descriptive Metadata**: Human-readable name and description support UI presentation. Name provides concise labels for palette and nodes ("If/Else Conditional"). Description explains purpose and behavior.

**Category**: Controls organize into categories (conditional, loop, flow, timing) for UI grouping. Category derives from control identity rather than stored data, eliminating synchronization concerns.

##### REQ-3.5.0.3 Control parameter schema definition (inputs for configuration)

**Typed Configuration**: Each control defines its own strongly-typed configuration structure rather than generic parameter lists. If/else defines branch conditions, delay defines duration, for-loop defines iterator bindings. Per-control typing provides compile-time checking and IDE autocompletion.

**Expression Parameters**: Many configurations include expressions evaluated at runtime. Conditions, bounds, and timing values may reference workflow variables. Expression parameters integrate with WAVE's variable system for dynamic control behavior.

**Validation**: Each control type defines validation rules. Required parameters must be present, type constraints satisfied, semantic invariants hold. Validation occurs at save time, preventing malformed configurations from persisting.

##### REQ-3.5.0.4 Control metadata for core controls (id, name, category, iconName)

**Core Control Inventory**:

| Category | Controls |
|----------|----------|
| Conditional | If/Else, Switch/Case, Assert |
| Loop | For, For-Each, While, Break, Continue |
| Flow | Parallel, Try-Catch-Finally, Halt, Variable Setter |
| Timing | Delay, Wait-Until |

**Metadata Structure**: Each control defines: identifier, display name, category, and icon name (Lucide library). This enables palette rendering and node display without runtime API calls.
