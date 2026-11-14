##### REQ-3.4.3.8 Set variables during execution

The setter plugin modifies the variables of the flow it runs in during workflow execution, allowing to update shared state, accumulate results, and pass data between workflow steps.

**Scope of the Flow It Runs In**: The setter writes the variables of the flow its block runs in. In a Test it writes test-level variables, visible to every Block in the Test for the whole run. Inside a Function it writes that Function's own variables and its declared outputs; the Function's declared inputs are read-only, and a row naming one is refused. There is no block-local variable scope: blocks are isolated execution units with explicit inputs and outputs, and a variable the setter writes always belongs to the Test or the Function its block runs in.

**Create or Update Semantics**: When the setter targets a variable name that already exists in its flow, it updates the variable's value in place. A setter row naming a variable that does not exist yet declares it as a variable of the flow the block runs in, the Test or the Function, when the block is saved, with the row's type, holding null until the block runs. A row's value is a data field and accepts null (see REQ-3.1.3.21).

**Namespace Targets Rejected**: The setter cannot modify namespace variables. Attempting to set a value on a reserved name (`system`, `suite`, `regression`) or a namespace-prefixed target is rejected with a clear error message. Namespace variables are read-only and resolved at runtime through namespace objects.

**Reserved Name Enforcement**: The identifiers `system`, `suite`, and `regression` cannot be used as variable names. Variable name validation rejects these reserved names at creation time, preventing ambiguity between test-local variables and namespace references.

**Atomic Variables for Concurrent Access**: All variables in the system are implemented with atomic access semantics: a read from a parallel branch never observes a half-written value. Atomicity is per variable access, not per block (REQ-3.2.3.4).
