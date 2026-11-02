#### 3.7.4 Delay Control

##### REQ-3.5.4.1 Configurable delay blocks

The delay control pauses workflow execution for a specified duration, enabling timing-sensitive scenarios such as waiting for asynchronous operations, simulating user pacing, or allowing external systems to stabilize.

**Duration Parameter**: The control accepts a single required parameter specifying pause duration in milliseconds. The backend validates the value as a positive integer greater than zero.

**Execution Behavior**: The control suspends execution using async sleep, yielding to the event loop without consuming CPU. Execution logs duration at start and actual elapsed time at completion. If the workflow is canceled or times out during the delay, the control terminates immediately and propagates cancellation.

**Variable Integration**: The duration parameter supports expression evaluation, enabling dynamic delays based on runtime variables (e.g., increasing delay on retry iterations).

**Expression Validation**: The duration parameter accepts two forms of input:

| Input Form | Example | Validation |
|------------|---------|------------|
| Literal integer | `5000` | Must be positive integer (ms) |
| Expression (toggle enabled) | `delay_ms`, `base * 2` | AST-validated at design time, result type-checked at execution |

When the expression toggle is enabled, the value is a pure Python expression evaluated at runtime. The result must be a positive integer representing milliseconds. Float results are rejected. The validation method performs AST-based syntax checking at design time and defers value validation to execution time when variables are resolved.
