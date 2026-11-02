### 5.10 Parallel Execution

#### 5.10.1 Parallel Branch Execution

##### REQ-3.2.3.1 Execute parallel branches within single test

**Fork**: A parallel section declares an ordered list of named branches. Each branch owns one lane out of the section; the sub-flow wired to that lane is the branch body. When the flow reaches the section, every branch body starts, and they proceed alongside each other until the join releases.

**A branch is an ordinary sub-flow**: Within a branch, blocks run in sequence exactly as they do on the main flow. Every element type is admissible - plugins, Functions, controls, further parallel sections - and every mechanism behaves identically: variable reads and writes, output bindings, profile bindings, error handling through Try/Catch, loop break and continue targeting a loop inside the branch, and Halt with its declared scope.

**Concurrency model**: Branch progress interleaves at block boundaries. The engine advances whichever branches are ready while others wait on their own work, which is what makes concurrent I/O - requests, queries, remote commands, waits - overlap in wall-clock time. A branch never observes a sibling mid-block: an effect a sibling has becomes visible at block boundaries, which is the granularity every cross-branch guarantee in this section is stated at.

**Branch identity**: Each branch carries a name, chosen by the author and defaulted when left blank. The name identifies the branch on the block's lane, in the run log and in reports, so a section with several lanes stays readable long after it was authored. Branch order is the authored order and is stable: it determines lane order on the block and the order branches appear wherever the section is reported.

**Section boundaries**: Branch bodies are scoped to the section. A branch body dead-ends when its last block completes; the flow does not leave a branch directly. The only continuation out of a parallel section is its exit lane, which is the join (5.10.2). A block outside the section is therefore never reachable from inside a branch except through the join.

##### REQ-3.2.3.5 Nested parallel sections at any depth

**Nesting**: A parallel section may appear inside a branch of another parallel section, inside a loop body, inside a Try/Catch path, inside a Wait Until retry lane, and inside a Function body at any depth. The inner section forks and joins within the branch that contains it: its join must release before that branch can continue, and the containing section's join accounts only for its own direct branches.

**Composition with Functions**: A Function Block inside a branch runs its own workflow to completion inside that branch, including any parallel section the Function itself contains. Function input and output parameters behave as they do anywhere else; the Function boundary is unaffected by running concurrently with sibling branches.

**Depth and bookkeeping**: Every nested section maintains its own branch set, join state and time limits. Signals that address an enclosing construct - a loop break, a Halt at test scope - traverse outward through the sections that contain them, terminating each section they pass through under the rules in 5.10.3.
