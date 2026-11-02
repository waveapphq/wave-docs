### 15.7 Plugin Development Interface

#### 15.7.0 Plugin Configuration Pattern

##### REQ-3.4.1.4 HTTP/HTTPS client operations

Plugin configuration follows a schema-driven pattern. Each plugin defines a parameter schema in the frontend registry, and the element configuration modal (§15.4.1, REQ-3.1.1.5) renders the appropriate input controls automatically. No plugin-specific UI components are needed - the generic parameter table handles all parameter types using WAVE's established type system.

**Plugin Configuration Architecture**: Plugin configuration uses three coordinated registries:

| Registry | Content | Purpose |
|----------|---------|---------|
| Plugin definitions | ID, name, category, icon, color | Palette rendering and node display |
| Parameter schemas | Parameter list with types, defaults, validation | Config modal form generation |
| Plugin metadata | Version history, changelog, documentation | Version selector and info dialogs |

**Schema-to-Form Mapping**: The parameter schema drives the config modal form. Each parameter's type determines which input control renders (see §15.15.5 for the complete type-to-component mapping). Parameter types align with WAVE's canonical type system used in the variable manager and profile schemas - including complex types like `map` that render using the same editor modals (MapEditorModal) used throughout the application. Required parameters display validation indicators and prevent saving when empty. Default values pre-populate fields on first open.

**Plugin Parameter Flow**: Configuration values flow from the frontend config modal to backend plugin execution. The frontend stores parameter values and an `expressionFields` array in the workflow node's configuration state. During execution, the backend extracts these values, evaluates expression-flagged parameters as Python expressions with safe builtins, validates result types against declared parameter types, and passes the resolved values to the plugin's execute method.

| Stage | Location | Action |
|-------|----------|--------|
| Configuration | Config modal | User enters parameter values (literal or expression) |
| Persistence | Workflow store | Values and expressionFields stored in node config state |
| Save | Test API | Config serialized with workflow structure |
| Execution | Backend service | Expression parameters evaluated, result types validated |
| Validation | Plugin validate method | Type checking and constraint verification |
| Execution | Plugin execute method | Receives resolved, validated parameters |

**HTTP Plugin Configuration**: The HTTP plugin demonstrates the full configuration pattern with multiple parameter types and conditional visibility:

| Parameter | Type | Required | Behavior |
|-----------|------|----------|----------|
| URL | String | Yes | Text input with expression toggle for dynamic URLs |
| Method | Enum | Yes | Dropdown: GET, POST, PUT, DELETE, PATCH, HEAD, OPTIONS (default: GET) |
| Headers | Map | No | Key-value pair editor for request headers (string values only) |
| Body | String | No | Text area, visible when method supports a body (POST, PUT, PATCH, DELETE) |
| Timeout | Integer | No | Request timeout in seconds (default: 30) |

**HTTP Outputs**: The plugin produces three output variables that can be bound to workflow variables for use by downstream elements:

| Output | Type | Content |
|--------|------|---------|
| status_code | Integer | HTTP response status code |
| body | String | Response body as text |
| headers | Map | Response headers as string key-value pairs |

**Unit Alignment**: Parameter units in the frontend schema must match backend expectations exactly. There is no conversion layer between the config modal and backend plugin execution - values pass through as-is. For example, timeout values are specified in seconds (matching the backend), not milliseconds.
