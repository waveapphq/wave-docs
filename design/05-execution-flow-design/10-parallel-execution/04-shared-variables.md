### 5.10 Parallel Execution

#### 5.10.4 Shared Variables Across Branches

##### REQ-3.2.3.3 Branches share the test's live variables

**One set of variables**: A parallel section does not copy variables at the fork or merge them at the join. Branches read and write the same variables the rest of the test uses, with the same names, the same types and the same scoping rules. Nothing about a variable changes because the block touching it happens to sit inside a branch.

**Writes are visible to siblings**: A value written by one branch is visible to any read a sibling performs afterwards. Ordering is defined at block boundaries: a read that begins after a write completed observes that write. This is the property cross-branch coordination rests on - a Wait Until in Watch mode inside one branch is satisfied by what another branch writes, a consumer branch reads what a producer branch produced, and a probe branch observes state a load branch is generating.

**Writes survive the join**: Because there is no branch-local copy, what a branch wrote is simply the variable's value after the section. Blocks past the join read it exactly as they would read a write from anywhere else, and the variable panel shows the live value during the run rather than a per-branch view that only resolves at the end.

**Concurrent writes are the author's business**: Two branches writing the same variable is last-write-wins. The engine does not arbitrate which write is more correct, and does not silently merge them. Where the order matters, it is expressed in the test - by writing to different variables, or by sequencing the branches against each other with Wait Until.

**Work that is not shared state**: Concurrency guarantees cover WAVE's own variables. They do not extend to the systems a test drives: two branches writing the same file, calling the same endpoint or mutating the same database row race in that system, on that system's terms. Coordination there is expressed in the test, again with Wait Until, and the design does not pretend otherwise.

##### REQ-3.2.3.4 Atomic variable access under concurrent branches

**Guarantee**: A read never observes a partially written value. A write to a variable completes before any read of that variable returns, and concurrent reads of the same variable do not block each other. This holds for compound values - a table, a map, a list - as much as for scalars, so a branch reading a table never sees it half-replaced by a sibling's write.

**Reader/writer semantics**: Reads of a variable proceed concurrently; a write to that variable excludes readers for the duration of the write. Access is per variable, so activity on one variable never delays work on another.

**An explicit mechanism, not an ambient one**: The guarantee is provided by an explicit access layer that every variable read and write in the execution path goes through, not by a property of the runtime that happens to hold today.

**Scope of the guarantee**: Atomicity is per variable access, not per block. A block that reads a variable, computes, and writes it back is not atomic as a whole - two branches doing that concurrently can lose one of the updates. Where a read-modify-write must not interleave, the test expresses it, and the same Wait Until coordination applies.
