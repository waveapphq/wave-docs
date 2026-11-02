### 5.10 Parallel Execution

#### 5.10.2 Synchronization and Convergence

##### REQ-3.2.3.2 Synchronization point for branch convergence

**The join is the exit**: Every parallel section carries one always-present exit lane. It is the join: concurrent execution ends there and sequential execution resumes. The exit is the only continuation out of the section, so a section can never dead-end the flow and a reader can always see where the concurrent region stops.

**Release**: The join releases when the section's join mode is satisfied (REQ-3.2.3.6) or when a time limit expires (REQ-3.2.3.7). Until then the flow does not advance past the section, no matter how the branches are progressing.

**Branch states at the join**: A branch reaches the join having completed, failed, or been stopped. The section's own outcome follows from those: it passes when no branch failed and no time limit expired, and fails otherwise. A branch stopped because the join had already released is the join mode working, not a failure - under first-to-finish every loser is stopped by design. A failed section skips its exit lane, so the flow stops there rather than continuing past a section that did not do its work (5.10.3).

**Convergence and the disabled case**: A disabled parallel section runs nothing. Its branch bodies cascade to skipped and the flow continues through the exit, which is how every container control behaves when disabled - the exit is the pass-through.

##### REQ-3.2.3.6 Configurable join mode (wait-for-all, first-to-finish)

**Wait for all**: The default. The join releases once every branch has reached a terminal state. This is the mode for work that must all happen before the test moves on.

**First to finish**: The join releases as soon as the first branch reaches a terminal state. The remaining branches are stopped as 5.10.3 says. This is the mode for racing redundant approaches, and for a probe that only needs to observe something once.

**No condition mode**: There is no join mode that evaluates a condition over branch statistics. A condition-gated wait is expressed inside a branch with Wait Until in Watch mode, whose condition another branch satisfies - the same mechanism, authored where the condition is meaningful, using the assertion machinery the rest of the product already uses.

##### REQ-3.2.3.7 Optional time limit per branch and per section

**Two independent limits**: A branch may carry its own time limit, and the section may carry one for the whole concurrent region. Both are optional and independent; either may be set without the other. A branch with no limit is bounded only by the section limit, if there is one, and a section with no limit is bounded only by whatever limits its branches carry.

**Branch limit**: A branch that exceeds its own limit stops at its next block boundary and ends as failed. It is a failed branch in every respect, so it ends the section under the failure rule (REQ-3.2.3.8).

**Section limit**: When the section limit expires, every branch still running stops at its next block boundary and the section ends as failed. Branches that had already finished keep the outcome they reached.

**Reporting**: A limit that expires is reported at the block it stopped and at the section, naming which limit expired, so a run that ran out of time is distinguishable from one that failed on its own terms.
