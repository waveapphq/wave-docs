#### 3.7.12 Parallel Control

The Parallel control runs several parts of a test at the same time and brings them back
together at one point. It is the control for work that is genuinely concurrent: a producer
and a consumer, load held while behavior is probed under it, redundant approaches raced
against each other, and any pairing where one part of the flow waits for something another
part brings about.

Canvas authoring of branches and their merge is specified under REQ-3.1.1.3 in
the Visual Workflow Builder design; the execution semantics are
specified under REQ-3.2.3.1-9 in the Parallel Execution design.

##### Structure

**Lanes**: Parallel is a container control. The node carries one lane per branch plus an
always-present exit lane, in the same shape as Try/Catch, Switch and Wait Until. Branch lanes
hold the concurrent sub-flows; the exit lane is the join and the only continuation out of the
control, so a parallel section never dead-ends the flow.

| Lane | Role |
|------|------|
| branch | One per branch, in authored order. Holds a sub-flow that runs alongside its siblings and dead-ends at its own end |
| then | Always present. The join - the flow leaves the control only here |

Lanes carry no text on the canvas; each lane's name is its hover tooltip, the same convention
the other container controls use. Branch lanes take a color of their own so a fan-out is
distinguishable at a glance from a try path, a case lane or a loop body; the exit keeps the
container-exit neutral.

##### Authoring

**Branch creation**: Branches are authored as an ordered list on the control's configuration.
Each branch has a name, used on its lane tooltip, in the run log and in reports, and an
optional time limit of its own. Branches can be added, renamed, reordered and removed;
reordering moves the lanes with them so the canvas keeps matching the list. A parallel
section holds at least two branches - one branch is a sequence, which the canvas already
expresses without a control.

**Merging**: The join releases according to the control's join mode - when every branch has
finished, or as soon as the first one does - and the exit lane then runs. What each branch
computed is already in the test's variables, because branches share them live rather than
working on copies (5.10.4); there is no merge step and no aggregated result object to unpack.
A branch that needs to report something names a variable and writes it, exactly as a block
outside the section would.

**Configuration**: The control carries an optional section-wide time limit, the branch list
with its per-branch names and optional limits, and the join mode. It has no output variables:
everything a branch produces reaches later blocks through the variables it wrote.

##### Runtime behavior

**Failure and interruption**: A failed branch fails the control, stops its siblings at their
next block boundary and skips the exit, which stops the run - the same shape as a failing
sub-flow in the other containers. Try/Catch inside a branch is how a failure is tolerated.
Halt inside a branch ends the section and then carries its declared scope and status outward.
A disabled Parallel skips its branch bodies and passes the flow through the exit lane.

**Debugging**: Breakpoints work inside branch bodies. The first branch to hit one becomes the
paused block and the section parks; Step drives that branch while its siblings hold, and
Continue releases the section (5.10.3).

**Authoring surface**: The control is expressible in recipes with the same vocabulary the
canvas uses - named branches each holding their own steps, the join mode, and the time limits
- so a compiled test and a hand-built one are indistinguishable.
