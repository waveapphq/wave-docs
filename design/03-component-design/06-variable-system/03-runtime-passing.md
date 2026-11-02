### 3.6.3 Runtime Variable Passing

#### REQ-3.1.3.3 Runtime variable passing between elements

The runtime variable passing mechanism enables elements (plugins and controls) executing within the same test workflow to exchange data through shared test-scoped variables.

**Shared Test Scope**: All elements in a test share the test's variable namespace. When an element assigns an output to a test-scoped variable, subsequent elements can reference it. This eliminates explicit message passing - the variable system is the communication channel.

**Sequential Data Flow**: In sequential execution, earlier elements produce variables that later elements consume. Element N sees all variables assigned by elements 1 through N-1. This creates a pipeline pattern where each element transforms or augments data, storing results for the next stage.

**Data Availability Guarantees**: Sequential execution provides deterministic availability by execution order. Parallel branches share the test's live variables: a value one branch writes is visible to any read a sibling performs afterwards, and stays the variable's value after the join (see REQ-3.2.3.3). A variable holds its start value until a block writes it; one declared with no value, or created by WAVE, starts as null (see REQ-3.1.3.20).

**Variable Naming Conventions**: Producer elements document the variables they create. Consumer elements declare dependencies on specific variable names through their parameter configuration. Descriptive names clarify data provenance without tracing execution paths.
