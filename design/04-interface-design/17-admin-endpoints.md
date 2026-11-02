### 4.17 Admin API Endpoints

The Admin API provides controlled state manipulation capabilities for testing infrastructure and production debugging scenarios. These endpoints enable direct data store state changes, cache management, and execution control that would otherwise require file system access or service restarts.

#### 4.17.1 Admin Mode Security Model

##### REQ-5.2.6.1 Admin mode toggle (startup flag to allow, API endpoint to activate/deactivate)

The admin mode toggle implements a two-layer security model that prevents accidental or unauthorized use of dangerous state manipulation operations. This design ensures admin capabilities are explicitly opted-in at deployment time and explicitly activated at runtime, providing defense in depth against misuse.

**Two-Layer Security Architecture**

The security model separates the ability to use admin features from their actual activation:

- Layer 1 (Deployment Gate): A startup configuration flag determines whether admin mode CAN ever be enabled for this deployment. This gate is controlled by operators during deployment and cannot be changed at runtime. When disabled, all admin endpoints return HTTP 404 Not Found as if the API does not exist, preventing capability enumeration by attackers.

- Layer 2 (Runtime Activation): When the deployment gate allows admin mode, runtime API endpoints control whether admin mode IS currently active. This enables activation for specific operations with explicit enable/disable lifecycle.

**Deployment Gate Configuration**

The deployment gate uses environment variable configuration following WAVE's standard configuration patterns. The admin allowed flag defaults to disabled, requiring explicit opt-in for any deployment where admin capabilities are needed. This prevents accidental exposure of admin endpoints in production deployments where the environment variable is not set.

Configuration follows the existing WAVE naming convention with the WAVE_ prefix. The value accepts standard boolean representations (true/false, 1/0, yes/no) with case-insensitive parsing. Invalid values are treated as disabled to fail safely.

**Security State Matrix**

The system operates in one of three security states based on the combination of deployment gate and runtime activation:

- State 1 (Fully Disabled): When the deployment gate disallows admin mode, all admin endpoints return HTTP 404 Not Found as if the API does not exist. This conceals the existence of admin capabilities from potential attackers. This is the default state for production deployments.

- State 2 (Allowed but Inactive): When the deployment gate allows admin mode but admin is not currently activated, only the activation endpoint and status endpoint are accessible. State manipulation endpoints return HTTP 403 Forbidden until admin mode is explicitly enabled.

- State 3 (Fully Active): When both the deployment gate allows admin mode and admin has been activated at runtime, all admin endpoints are accessible. This state enables full state manipulation capabilities.

**Runtime Activation Endpoints**

The activation endpoint enables admin mode for the current server instance. It requires a reason parameter documenting why admin mode is being enabled, supporting audit trail requirements. The reason is logged and included in the admin status response. Activation is idempotent; calling enable when already enabled updates the reason and refreshes the activation timestamp.

The deactivation endpoint disables admin mode, returning the system to the allowed-but-inactive state. All subsequent state manipulation requests are rejected until admin mode is re-enabled. Deactivation is also idempotent; calling disable when already disabled has no effect.

**Activation Lifecycle**

Admin mode activation is instance-scoped, meaning each server instance maintains its own activation state.

Activation persists for the lifetime of the server process. Server restart returns the system to the allowed-but-inactive state, requiring explicit re-activation. This ensures admin access is not accidentally preserved across deployments or restarts.

**Request and Response Patterns**

The enable endpoint accepts a JSON request body containing the reason for activation. The response confirms activation with the timestamp when admin mode was enabled and the provided reason. The response also indicates the full set of admin capabilities now available.

The disable endpoint requires no request body. The response confirms deactivation with the timestamp when admin mode was disabled and the duration for which admin mode was active.

Both endpoints return HTTP 404 Not Found when the deployment gate disallows admin mode, concealing the existence of admin capabilities. Server-side logging captures the blocked attempt for operator visibility, allowing debugging without exposing information to clients.

**Error Handling**

Missing or empty reason on enable requests returns HTTP 400 Bad Request with a clear error message. The reason requirement ensures audit trail completeness and prevents accidental activation without documentation.

Requests to state manipulation endpoints when admin is allowed but not enabled return HTTP 403 Forbidden with an error message indicating admin mode must be enabled first. This creates a clear distinction: HTTP 404 means admin is not available in this deployment, while HTTP 403 means admin is available but not currently activated.

**Integration with Application Lifecycle**

Admin mode state integrates with the application configuration system but is stored separately from persistent configuration. The deployment gate is read from configuration at startup and cached. Runtime activation state is held in memory only.

The admin activation state is exposed through the admin status endpoint for monitoring and operational visibility. Health check endpoints do not require admin mode and remain accessible regardless of admin state.

##### REQ-5.2.6.2 Admin mode status endpoint (enabled state, activation time, reason)

The admin mode status endpoint provides visibility into current admin mode state for operators, monitoring systems, and automation tooling. This endpoint serves as the primary mechanism for querying whether admin capabilities are available and active.

**Status Information Model**: The response includes four fields: deployment gate status indicating whether admin mode can be enabled (static per server lifetime), activation status indicating whether admin mode is currently active (dynamic), activation timestamp in ISO 8601 UTC format for audit trail support, and activation reason documenting why admin mode was enabled. When inactive, timestamp and reason are null; when active, all fields are populated.

**Access Control**: The status endpoint follows the admin security model. When the deployment gate disallows admin mode, returns HTTP 404 Not Found concealing admin capabilities. When allowed, the endpoint is always accessible regardless of activation state, enabling operators to check availability before attempting operations.

**Operational Use Cases**: Supports pre-operation verification before state manipulation, monitoring and alerting for unexpected or prolonged admin sessions, audit and compliance through timestamp and reason accountability, and automation integration for CI/CD pipelines and testing frameworks to verify admin setup.

#### 4.17.2 State Management Operations

##### REQ-5.2.6.3 State reset endpoint (clear all entities and executions)

The state reset endpoint clears all test data from the system, enabling clean-slate scenarios for E2E testing and development workflows. The operation maintains symmetry with bulk upsert capabilities: anything that can be injected can be cleared.

**Scope**: The reset clears all test data categories: entity definitions (tests, suites, regressions) and the execution state held in memory (records, variables and runtime state). System infrastructure is preserved: plugins, configuration, and credential profiles remain untouched.

**Runtime History**: The activity history the assistant integration accumulates in memory is clearable through the same endpoint, but only when named explicitly, so that routine test-data resets leave diagnostic history intact. Live assistant connections are never dropped by a reset: they are governed by the integration's own connect and disconnect lifecycle.

**Selective Reset**: An optional scope parameter accepts a list of categories to clear (e.g., only executions). When omitted, all test data categories are cleared. Environments, credential profiles, assistant history, network settings (the host key policy, the public certificate authorities choice and the additional certificates) and the trusted SSH hosts list are cleared only when named. Selective reset enables testing error isolation scenarios but may leave WAVE in an inconsistent state (e.g., executions referencing deleted entities). This is intentional for testing edge cases and error handling paths.

**Safety**: Beyond requiring active admin mode, the endpoint requires an explicit confirmation parameter set to true in the request body. Requests without confirmation return HTTP 400 Bad Request. This prevents accidental invocation through automation or operator error.

**Execution**: Each data category clears independently with failures isolated per category. The response includes item counts cleared per category and an ISO 8601 UTC timestamp. Partial success scenarios report what was cleared before any failure. All related caches (including dashboard cache) are invalidated after clearing.

**Error Handling**: Returns HTTP 404 when deployment gate disallows admin, HTTP 403 when admin not enabled, HTTP 400 for missing confirmation, and HTTP 500 for storage errors with partial results in the response body.

**Audit**: Reset operations are logged with timestamp, admin session details, and counts per category cleared.

#### 4.17.3 Bulk Entity Operations

##### REQ-5.2.6.4 Bulk entity operations (overview)

The bulk entity endpoints enable controlled data manipulation for E2E testing scenarios. Unlike the user-facing import feature which provides file-based import with conflict resolution UI, these admin endpoints are programmatic, immediate, and support the full range of CRUD operations.

**Operations**: Add (create, fails if exists), Modify (partial update, fails if not exists), Replace (upsert), Delete (fails if not exists). Mixed operations within a single request are supported.

**Common Behavior**: All bulk endpoints share consistent patterns for referential integrity validation, cache invalidation, error handling (404/403/400/500), and audit logging. Entity-specific details are documented in each sub-requirement below.

**Implementation Strategy**: Each entity type has a dedicated endpoint.

##### REQ-5.2.6.4.1 Bulk execution operations

Manages execution records - the primary mechanism for seeding dashboard test data. Required fields: identifier, entity type (test/suite/regression), source entity reference, and status. Supports optional progress percentage, environment, and timestamps for timeline visualizations.

##### REQ-5.2.6.4.2 Bulk test operations

Manages test definitions - the atomic execution units containing validation logic and block sequences. Required fields: identifier, name, and version. Supports optional description and tags.

##### REQ-5.2.6.4.3 Bulk suite operations

Manages test suites - containers that group related tests with shared configuration and execution order. Required fields: identifier, name, and version. Supports optional test references.

##### REQ-5.2.6.4.4 Bulk regression operations

Manages regressions - top-level containers grouping suites for comprehensive validation cycles. Required fields: identifier, name, and version. Supports optional suite references.

##### REQ-5.2.6.4.5 Bulk environment operations

Manages environment definitions - named configuration contexts that categorize connection profiles for deployment targets (Production, Staging, Development, QA). Required fields: identifier and name (unique). Supports optional description.

##### REQ-5.2.6.4.6 Bulk profile operations

Manages connection profiles - configuration containers storing credentials and parameters for external systems. Required fields: identifier and name. Supports type-specific connection parameters (database, ssh, http, etc.).
