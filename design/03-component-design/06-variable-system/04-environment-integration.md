### 3.6.4 Environment Variable Integration

#### REQ-3.1.3.4 Environment variable integration

The variable system does not implicitly integrate with OS environment variables. Environment variables are not part of the variable storage levels and are not automatically available to test workflows.

**Explicit Access Only**: Tests that need to read environment variables do so through the Python plugin, which has full access to the Python standard library including `os.environ`. This keeps environment access explicit and traceable in the workflow rather than creating hidden dependencies through implicit variable resolution.

**No Implicit Fallback**: Variable references resolve against test-level variables (by name) or against namespace objects for system, suite, and regression levels (by namespace prefix). If a variable is not found in the applicable scope, the reference is unresolved - it does not fall back to OS environment variables.
