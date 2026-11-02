### 5.10 Parallel Execution

#### 5.10.3 Branch Termination Control

##### REQ-3.2.3.8 Branch failure ends the section and stops running siblings

**Failure ends the section**: When a branch fails, the parallel section fails. Its still-running siblings are stopped, the exit lane is skipped, and the run stops there. This mirrors how a failing sub-flow behaves in the other container controls: a container does not continue past work that did not succeed.

**Tolerating a failure**: A failure that should not end the section is handled the way failures are handled everywhere else in the product - with Try/Catch inside the branch. The catch path absorbs the error, the branch continues or completes normally, and the section never sees a failed branch. There is no separate error strategy on the parallel section, because that would be a second, parallel-only way to express something the product already expresses.

**Stopping a branch**: A branch that is stopped - because a sibling failed, because the join already released under first-to-finish, or because a time limit expired - is canceled at once, even in the middle of the block it is executing, so the section never waits on a long block. That block's work may be left unfinished: a request is cut off, and what it already did on the remote side stays done. The branch's stopped block is reported as stopped, not failed; blocks the branch had not reached are marked skipped.

**Run Stop**: A Stop of the whole run cancels every lane at once, so a block running inside any branch is stopped the same way it would be outside a parallel section, and every process that block started is ended with it (SHRUN-15).

**Assert failure inside a branch**: An Assert or Wait Until that fails inside a branch behaves exactly as it does elsewhere. Without Fail Current Scope it is informational and the branch continues. With Fail Current Scope a single-mode Assert or a Wait Until fails its block, which fails the branch and ends the section like any block failure; a split-mode Assert marks the enclosing Test or Function body failed and the branch continues down its fail branch without stopping the section. Only a block failure, a Halt, or an expired time limit stops a branch or the section.

**Halt inside a branch**: A Halt raised inside a branch ends the section immediately: siblings are stopped the same way and the exit lane is skipped. The Halt then continues outward with the scope and status the author declared, so a Halt at test scope ends the test with that status, and a Halt at suite or regression scope propagates past the test as it would from anywhere else.

##### REQ-3.2.3.9 Debug pause parks the whole parallel section

**One paused block**: Debugging keeps a single cursor. The first branch to reach a pause trigger - a breakpoint, or a step that lands inside it - becomes the paused block, and the section parks: every sibling stops advancing at its next block boundary and holds there rather than running to completion while the user inspects state.

**Stepping**: Step advances the paused branch alone; parked siblings stay parked, so stepping through a branch is as predictable as stepping through the main flow. Continue releases the whole section, and every parked branch resumes. If a sibling was itself holding at a pause trigger, it becomes the paused block next, so several breakpoints inside one section surface one after another rather than competing.

**State while parked**: Variables, the canvas paint and the console show the state the section actually reached - branches that finished carry their outcome, the paused block carries the paused treatment, and parked branches show the last block they completed. Because parked branches are stopped at a block boundary, the variable state a user inspects is a consistent one.
