#### 3.7.11 Wait Until Control

The Wait Until control holds execution at a point in the workflow until a stated condition
becomes true, or until a time limit expires. It is the control for the "the system is not
ready yet" problem: a deployment that needs a moment to come up, a queue that needs to
drain, an endpoint that answers only once a dependency has settled.

##### REQ-3.5.4.3 Wait until condition with timeout

**Condition**: The condition is an assertion set, authored and evaluated with the same
assertion machinery the Assert, If/Else, Switch and While controls use. It carries literal
comparisons, expression rows, and AND/OR groups, and it is evaluated against the variable
context visible at the wait's position in the workflow. Wait Until introduces no condition
form of its own. A check on a null value answers as the Assert control's null rule says, so
`status equals 'ready'` on a null `status` answers false and the wait keeps waiting, as it
does for a field that is not there yet (see REQ-3.1.3.25).

**Structure**: Wait Until is a container control with an always-present egress, so a wait
never dead-ends the flow. A mode setting selects its shape:

| Mode | Lanes | Meaning |
|------|-------|---------|
| Retry | a retry sub-flow lane plus the egress | The retry sub-flow runs on every round; its steps are what make the condition eventually true. When the condition holds at the first check the retry lane never runs and its blocks paint skipped |
| Watch | the egress alone | The wait only re-evaluates the condition; nothing inside the wait changes it |

Retry is the mode that stands on its own: because the workflow is otherwise idle for the
duration of the wait, a condition over the test's own variables cannot change unless
something re-reads the system under test. The retry sub-flow is where that read lives -
a request, a query, a command whose captured values the condition then tests. Watch is the
shape for a condition another concurrently executing flow will satisfy.

**Time limit**: The timeout is required and bounds the entire wait, including a retry
sub-flow still in progress when the deadline arrives. Blocks left running inside the retry
sub-flow at that moment are reported as stopped by the wait, the same way a loop reports
the blocks its own time limit cut short.

**Outcome when the limit is reached**: A per-control setting decides what running out of
time weighs, the same in Retry and Watch mode. When it is disabled the timeout is
informational: the block paints amber, the run log warns, and execution continues through
the egress. When it is enabled the timeout is a block failure, exactly like any failed
block: the block paints failed and the flow stops at the wait. A surrounding Try/Catch
catches it through its catch path, which is the way to react to a wait that never landed;
otherwise it ends the enclosing Function, whose Function Block reports failed, or the root
Test, which ends failed. In Retry mode the blocks still running in the retry sub-flow at the
deadline are reported stopped first, then the wait fails. A block that fails inside the retry
sub-flow fails the wait at once, like a failed block inside any container; a Try/Catch inside the
sub-flow is how a wait keeps polling past a round that failed. An optional message names what
the wait was for and is what the run log prints when it gives up.

**Values produced**: The control reports whether the condition came true and how long the
wait took, both bindable to variables. Which round satisfied the condition is visible in
the run log rather than exposed as a value.

**Disabled behavior**: A disabled Wait Until skips its retry sub-flow and continues through
its egress, identical to a disabled Try/Catch, loop or Switch, which is what allows
skip-while-paused to operate on it.

##### REQ-3.5.4.4 Poll-based condition checking

**Round order**: A round evaluates the condition first and leaves immediately if it already
holds, so a wait whose condition is already satisfied costs nothing and runs no retry
sub-flow. Otherwise the retry sub-flow runs, the check interval elapses, and the condition
is evaluated again. Rounds repeat until the condition holds or the time limit is reached.

**Check interval**: The interval spaces consecutive checks and defaults to one second. It
is the recovery time given to the system being waited on, so it is measured after the retry
sub-flow completes rather than between the start of one round and the next. The wait yields
while the interval elapses and consumes no processing while idle, and it stops waiting the
moment the run is stopped.

**Observability**: Every round is reported to the run log, and the block carries the ordinary
running badge for the whole wait, so a long wait reads as alive rather than hung.

**Authoring parity**: The recipe format expresses a Wait Until as a container step carrying
the condition, the time limit, the optional interval and message, and - in Retry mode - the
retry sub-flow, so a compiled wait and a hand-built one are indistinguishable and the
round trip back to a recipe is lossless.
