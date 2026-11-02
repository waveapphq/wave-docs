#### 3.7.8 Assert Control

##### REQ-3.5.1.11 Assertion block with success/failure branches

The assert control evaluates a set of typed assertions against runtime variable values and routes execution based on the result. It operates purely on variables already in scope - upstream plugins (JSON, XML, HTTP, etc.) extract and store values; the assert control validates them.

##### REQ-3.3.1.1 Exact string matching
##### REQ-3.3.1.2 Regular expression patterns
##### REQ-3.3.1.4 Count-based validations (equal, less, greater)

**Assertion Row Types**: Each assertion row is either literal (variable + type + operator + value) or expression (a single boolean Python expression). Multiple rows are combined using AND/OR logic with optional grouping for precedence. The overall result is a single `passed` boolean.

| Type | Operators |
|------|-----------|
| integer/float | equals, not equals, less than, greater than, less or equal, greater or equal, is null, not null |
| string | equals, not equals, contains, does not contain, starts with, ends with, matches regex, is null, not null |
| boolean | is true, is false, is null, not null |
| array | contains, does not contain, is empty, is not empty, length equals, is null, not null |
| map | has key, does not have key, is empty, is not empty, is null, not null |
| table | is null, not null, is empty, is not empty |

**Null values**: A literal row treats a null value as holding nothing: equals, every ordering comparison (less, greater, less or equal, greater or equal), contains, starts with, ends with, matches, has key, is true and is false answer false; not equals, does not contain and does not have key answer true; is empty and length equals 0 answer true; is not empty and any other length answer false. "is null" and "not null" tell null from empty, take no compare-to value, and are written `is_null` and `is_not_null` in a recipe. A table is empty when it has no rows. A failed assertion on a null value shows `actual: null`. The rule is the same in every block that checks a value (see REQ-3.1.3.25).

**Typed values**: `length equals` compares the item count as a number, and `contains` and `does not contain` on an array compare typed values, so the integer `3` and the text `'3'` are different items.

Expression rows accept any Python boolean expression referencing in-scope variables, with AST-based validation at design time.

##### REQ-3.3.1.5 Configurable failure behavior (stop/continue)

**Operating Modes**: The control has two modes; split-output mode is the default:

- **Single-output mode**: One outgoing handle. Pass = green/INFO log, execution continues. Fail = amber/WARN log with failed assertion details, then continues (default); with `fail_current_scope` enabled the failure is a block failure instead - red/ERROR log with failed assertion details, and the flow stops at the Assert.
- **Split-output mode**: Two handles (pass/fail). Pass = follow pass edges. Fail = follow fail edges. With `fail_current_scope` enabled the failure also marks the current scope failed; the flow still follows the fail edges, which is where cleanup belongs.

In both modes, a Python evaluation error (exception during assertion evaluation) halts execution like any plugin crash. Evaluation errors are distinct from assertion failures: a failure is a deliberate `false` result; an evaluation error is a programming mistake.

**`fail_current_scope` Toggle**: Controls whether an assertion failure is informational or counts, and how it counts depends on the mode.

- **Single mode - a flow gate**: the failure is a block failure, exactly like any failed block. The Assert paints failed and the flow stops there. A surrounding Try/Catch catches it through its catch path; otherwise the failure ends the enclosing Function, whose Function Block reports failed, or the root Test, which ends failed.
- **Split mode - a scope mark**: the failure marks the current scope failed and the flow continues down the fail branch. The current scope is the innermost enclosing root Test or Function body. Loop bodies, branches, Switch cases, Parallel lanes, Wait Until bodies, Remote Execution bodies and Try/Catch paths are not scopes: a mark made inside any of them lands on the Test or Function body around them. The mark records an expected failure the Assert already handled, so no Try/Catch runs its catch path for it and no Try/Catch clears it.

| Mode | `fail_current_scope` | Assertions | Block Color | Log Level | Scope Result | Execution |
|------|-------------|-----------|------------|-----------|-------------|-----------|
| single or split | false (default) or true | pass | green | INFO | unchanged | continues (pass edges in split mode) |
| single | false (default) | fail | amber | WARN | unchanged | continues |
| single | true | fail | red | ERROR | block failure | stops at the Assert |
| split | false (default) | fail | amber | WARN | unchanged | continues down the fail edges |
| split | true | fail | red | ERROR | scope marked failed | continues down the fail edges |

A mark is observed when its scope ends: a marked root Test ends with terminal status `failed`; a marked Function body makes its Function Block report failed, which is a failed block in the flow the Block sits in - that flow stops at the Function Block unless it sits in a Try/Catch, which catches it. Stopping a run on purpose, with a chosen status and scope, is the Halt control's job.

##### REQ-3.5.1.12 Success branch logs and continues

**Pass Semantics**: All assertions true - block turns green, INFO log, follows pass handle (split) or continues (single).

##### REQ-3.5.1.13 Failure branch logs with a configurable outcome

**Fail Semantics**: Combined result is false. Behavior depends on `fail_current_scope` and the mode:

- **`fail_current_scope=false` (default)**: Block turns amber (warning), WARN log with per-assertion detail. Scope result is unaffected - the assertion is informational. Single mode continues; split mode follows the fail handle.
- **`fail_current_scope=true`, single mode**: Block turns red (failed), ERROR log with per-assertion detail, and the Assert fails as a block: the flow stops at the Assert and the failure travels outward the way any block failure does.
- **`fail_current_scope=true`, split mode**: Block turns red (failed), ERROR log with per-assertion detail, the current scope is marked failed, and the flow follows the fail handle to the end of the scope.

WARN (not ERROR) is the default because assertion failure is a controlled outcome. ERROR is reserved for `fail_current_scope=true`, where the user explicitly wants the failure to count.

**Failure Summary**: The failure log opens with a count of how many of the assertions failed out of how many the author declared, followed by one detail line per failed assertion. AND-connected assertions short-circuit at the first failure, so fewer assertions may be evaluated than declared; the summary always counts what the author wrote, so the reader can locate the block's conditions from the message alone. When a single-mode failure fails the block, the same summary is the failure's message, so a catch path reads it from the caught error.

##### REQ-3.3.3.5 Response body validation (JSON/XML)
##### REQ-3.3.3.7 Validate JSON event fields and structure

**Composition with Data Plugins**: Assert contains no JSON/XML extraction. Validation is achieved by composing assert with upstream data plugins that store extracted values as variables.

**Parameter Schema**:

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `mode` | string enum | yes | `"single"` | `"single"` or `"split"` |
| `assertions` | list | yes | - | Ordered list of assertion rows with AND/OR grouping |
| `fail_current_scope` | boolean | no | `false` | When true, a failed assertion in single mode fails the block and stops the flow; in split mode the flow continues down the fail branch and the Test or Function it runs in ends failed |

**Output Variable**: Single `passed` boolean reflecting the aggregate assertion result.

**Canvas Visuals**: Single mode has one bottom handle. Split mode has two bottom handles: left = pass (green checkmark), right = fail (red X). Before execution, pass edges are green and fail edges are red (structural coloring). After execution, only the taken branch retains its color; the untaken branch is dimmed.

Block border color on completion:
- Pass: green (success)
- Fail with `fail_current_scope=false`: amber (warning) - informational failure
- Fail with `fail_current_scope=true`: red (failed) - in single mode a block failure that stops the flow; in split mode a scope mark (the Test ends failed, a Function Block reports failed)
