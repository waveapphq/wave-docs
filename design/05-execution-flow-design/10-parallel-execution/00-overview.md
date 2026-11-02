### 5.10 Parallel Execution

Parallel execution extends the sequential model with concurrent work inside a single test. A parallel section forks the flow into named branches that run alongside each other and rejoins at a single exit, which is the section's synchronization point. This serves what a sequential walk cannot express: driving a producer and a consumer at the same time, holding load while probing behavior under it, racing redundant approaches, and satisfying a condition one part of the flow waits on while another part brings it about.

The system comprises branch execution for running several sub-flows concurrently, synchronization for the join and what releases it, shared variable access for the state branches read and write together, and termination control for failure, time limits and debugging.

The governing principle is that a branch is an ordinary sub-flow. Everything a block does outside a parallel section it does the same way inside one, including nesting further parallel sections, Functions, loops and error handling. Branches are concurrent, not isolated: they share the test's variables and see each other's writes, which is what makes cross-branch coordination possible.
