#### 3.7.1 Control Entity Model

Controls represent a distinct entity type that modifies execution flow rather than performing computational logic. This fundamental distinction from plugins and blocks drives the control system architecture: controls determine what executes, when, how many times, and under what conditions.

##### REQ-3.5.0.1 Controls as distinct entity type separate from plugins and blocks

Controls occupy a unique position in WAVE's component hierarchy. Plugins serve as atomic executors (SSH, HTTP, SQL), Functions serve as reusable compositions, and controls serve as flow modifiers that alter the execution path.

**Execution Semantics**: Plugins follow input-process-output. Controls modify the execution graph topology at runtime. A conditional creates branching paths, a loop causes repeated execution. This semantic difference requires controls to be modeled as a separate entity type.

**Fixed Control Set**: Controls are a closed set of built-in constructs: conditional branching (if/else, switch, assert), iteration (for, for-each, while, break, continue), flow modification (parallel, try-catch, halt), timing (delay, wait-until), and misc (note). Both frontend and backend share hardcoded definitions, eliminating discovery API complexity while ensuring type safety.

##### REQ-3.5.0.2 Control data model with id, version, name, description, category

**Identity**: Each control has a unique identifier from a fixed enumeration (if-else, for, delay, assert). Enumerated identifiers provide compile-time validation.

**Version**: Controls support semantic versioning enabling tests to pin specific versions. Existing tests continue using pinned versions while new tests adopt updates.

**Descriptive Metadata**: Human-readable name and description support UI presentation. Name provides concise labels for palette and nodes ("If/Else Conditional"). Description explains purpose and behavior.

**Category**: Controls organize into categories (conditional, loop, flow, timing, misc) for UI grouping. Category derives from control identity rather than stored data, eliminating synchronization concerns.

##### REQ-3.5.0.3 Control parameter schema definition (inputs for configuration)

**Declared Fields**: Each control declares its settings as a list of fields, the same kind of list a plugin declares: name, type, required, default, allowed values, bounds, whether a whole null is accepted, and whether the field is evaluated at run and so takes an expression. Delay declares a duration; For declares a mode, a count and a step. Beyond the plain kinds a control field can hold structure: a condition list (literal comparisons and expression rows joined by AND / OR, in groups), the If/Else branch list and the Switch case list (each entry with a label and a condition list), the Parallel lane list (each lane with a name and an optional time limit), the Variable Setter's map of variable rows, and a loop target. A Note declares its text and color the same way. A control's fields are kept, edited, saved, checked and run by the same shared Block code as a plugin's settings and a Function Block's inputs (REQ-3.1.1.5): one stored place, one set of expression, template and element-type marks, and one Save that stores only what the user set.

**Expression Parameters**: Counts, bounds, time limits, messages, compare values and Setter values may be expressions or, for text, templates, and reference workflow variables. The shared resolver evaluates a control's fields once, before the control runs, the entries of a Setter row's list or map value included. Conditions keep their own evaluator, because a While, a Wait Until and a Switch check them again each time they reach them and groups nest deeper than one level.

**Validation**: One save check covers every control field with the rules every Block field gets - required, type, allowed values, bounds, unknown fields, undeclared variables and read-only expressions - at any depth of a condition list, a branch list or a lane list; no place is marked both expression and template, and a setting with a default that is given null is refused. It adds each control's structure rules: an If/Else starts with its if branch and keeps its else last, a Switch holds at least one case and keeps its single default last, every if, else-if and case branch carries a condition, branch and lane identities are unique, a Parallel holds at least two lanes and each of its time limits is at least 1, a For count is not negative and its step is not 0, a While's max iterations is at least 1, each Break and Continue aims at a loop whose body holds it, and a condition row names a variable and, when its operator takes one, a compare value. The save refuses what the run would refuse, a written value with the message the run gives for that value, so a malformed configuration never persists; an expression whose result breaks one of these rules fails at run with the run's own line.

##### REQ-3.5.0.4 Control metadata for core controls (id, name, category, iconName)

**Core Control Inventory**:

| Category | Controls |
|----------|----------|
| Conditional | If/Else, Switch/Case, Assert |
| Loop | For, For-Each, While, Break, Continue |
| Flow | Parallel, Try-Catch-Finally, Halt, Variable Setter |
| Timing | Delay, Wait-Until |
| Misc | Note |

**Metadata Structure**: Each control defines: identifier, display name, category, and icon name (Lucide library). This enables palette rendering and node display without runtime API calls.
