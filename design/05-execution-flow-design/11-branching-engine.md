### 5.11 Branching Engine

##### REQ-3.5.0.8 Structure metadata for visual rendering
##### REQ-3.5.0.9 Branch definitions for container controls

#### 5.11.1 Overview

The branching engine extends execution graph traversal to support conditional edge routing. The standard model assumes one outgoing edge per node (linear BFS). The branching engine adds named edge handles that let a control direct execution down specific paths based on runtime results.

Two distinct shapes consume it. **Diverging** controls (Assert split mode, If/Else) hand the chosen branch to the main flow and never regain control; when nothing matches, that path simply ends. **Container** controls (Try/Catch, the loops, Switch/Case) execute their branch sub-flows as scoped paths that return, then continue through a dedicated `then` egress, so the flow always resumes at a single known point.

#### 5.11.2 Edge Handle Model

Every edge has an optional `sourceHandle` attribute identifying which output port it originates from. Controls with a single output use edges with no `sourceHandle`. Branching controls use named handles (e.g., Assert split mode: `pass` and `fail`).

Controls declare branching structure via hints: `{"type": "sequential"}` for single output, `{"type": "branching", "branches": ["pass", "fail"]}` for split. The execution result carries a `__branch` value indicating which handle to follow.

#### 5.11.3 Execution Graph Traversal

After a node executes, the engine inspects the result:

- **Sequential** (or no hint): follow all outgoing edges unconditionally
- **Branching**: follow only edges whose `sourceHandle` matches `__branch`

Edges without `sourceHandle` are always followed regardless of branching.

**Lanes not run**: Whenever a container leaves a lane unrun - a branch an If/Else or a split Assert did not take inside a sub-flow (any slot of a container: a loop body, a Try/Catch path, a Parallel lane, a Switch/Case case, a Wait Until body, a Remote Execution body) or a Function body, a Switch/Case case or default it did not run, a catch or finally path that did not run, a loop body that ran zero times (For, For Each, While), a Wait Until retry lane that never ran because the condition held at the first check, a Remote Execution body that never started - and whenever a failure, a Break, a Continue or a Halt cuts off the rest of a lane, every block of that lane paints skipped on the canvas, the lanes of containers inside it included. One shared step decides this for every container kind, on the Test's own flow and in a Function body, and records it the moment it is decided, so the live canvas, a reload and the run history agree. Skipping stops where the lane reaches a block a lane that ran also reaches, so a rejoin block keeps the status of the path that ran, and a lane that ran keeps the real status of each block. On the Test's own flow an If/Else and a split Assert paint nothing on the branch they did not take; a Switch/Case there paints the cases and default it did not run skipped.

#### 5.11.4 Assert Control Integration

- **Single mode**: returns sequential hint. Pass/fail recorded in output variable but does not affect traversal.
- **Split mode**: returns branching hint with `__branch: "pass"` or `"fail"`. Engine follows matching edges only.
- **Evaluation error**: bypasses branching entirely, halts execution like any plugin crash.

#### 5.11.5 If/Else Control Integration

- N-branch conditional with stable per-branch handle IDs (e.g., `branch_0`, `branch_1`, `branch_2`)
- Declares `allows_dynamic_branches: true` in structure hints - branch count determined by node configuration, not static metadata
- Evaluates conditions top-to-bottom, returns `__branch` matching the first passing branch's ID
- No-match with no else: returns `__branch: "__no_match__"` (sentinel that matches no edge, producing controlled halt)
- Branches converge naturally via multiple input edges on downstream nodes - no explicit merge node needed

#### 5.11.6 Switch/Case Control Integration

##### REQ-3.5.1.20 Switch/Case multi-branch decisions

Switch/Case is the container form of multi-way branching: several condition-guarded cases, an optional catch-all, and a guaranteed continuation.

- **Case branches** carry conditions evaluated by the same assertion evaluator If/Else uses, in author order, with stable per-case handle IDs. No case is structurally privileged - all are renameable, reorderable and removable, and reordering never re-points a wired edge because identity is the handle ID, not the position.
- **Null values** follow the check rule of REQ-3.1.3.25: each case's condition rows offer is null and not null like any condition, and a null goes to Default unless a case above catches it first, as a case using is null, not equals, does not contain, does not have key, is empty or length equals 0 does.
- **Break semantics** govern what happens after a matching case's sub-flow returns. Break set ends the Switch. Break cleared resumes evaluation at the next branch, which decides on its own condition, so several cases can run in one pass but none runs on another's condition.
- **Default** is optional, sits last and is the final stop of the branch walk: it runs when no case matched, and equally when a case above it falls through, because it carries no condition to refuse with. Having run, it ends the Switch - there is nothing after it to fall into.
- **`then` egress** is always present. Whether one case ran, several ran, the default ran, or nothing matched, execution continues through `then`. A Switch therefore never produces a controlled halt, which is what distinguishes it from If/Else.
- **Sub-flow execution** reuses the scoped-path machinery the container controls already share: each case's chain is traced and executed as a dead-end path that returns to the control, with arbitrary nesting of further controls inside it.
- **No outputs.** Which cases ran is observable through canvas paint and the run log; the control writes no value.

Disabled or skipped Switch nodes follow the container rule: every case and the default cascade to `skipped` and the flow resumes at `then`. Because that single fall-through path exists, a Switch paused at a breakpoint supports Skip, unlike the diverging controls, which must be stepped into.
