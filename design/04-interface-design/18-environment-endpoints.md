### 4.18 Environment Endpoints

#### 4.18.1 Environment Entity Model

##### REQ-4.7.10 Environment-specific configurations

Environments represent named configuration contexts that categorize deployment targets and enable organized management of connection profiles. Each environment (Production, Staging, Development, QA) serves as a logical grouping that profiles can reference, allowing users to select an execution context when running tests.

**Entity Identity**: Each environment has a unique identifier (UUID), a human-readable name that must be unique across all environments, and an optional description field for documentation purposes. The name serves as the primary display identifier in UI components such as environment selectors.

**Profile Association**: Environments serve as organizational containers for connection profiles. A profile holds `environment_values` keyed by environment, so each environment can have values in multiple profiles. This association enables environment-scoped profile discovery and filtering.

**Usage Tracking**: The system maintains a count of associated profiles for each environment.

#### 4.18.2 Environment Storage Pattern

##### REQ-6.1.1 YAML/JSON test definitions (Environment Application)

Environments follow the file-based storage pattern established for other WAVE entities, using YAML serialization for human readability and consistency with the broader storage architecture.

**Directory Structure**: Environments are stored under the data directory following the established entity storage pattern. Each environment has a dedicated subdirectory named by its identifier containing an `environment.yaml` file (e.g., `api_environments/{uuid}/environment.yaml`). This structure maintains consistency with other API entities (tests, suites, regressions).

**File Format**: Environment files contain the complete entity state serialized as YAML including identifier, name, description, creation timestamp, and update timestamp. Timestamps use ISO 8601 format in UTC timezone for consistency with other entities.

**Storage Operations**: The storage layer provides standard CRUD operations for environments following the established patterns in the file storage module. Create operations generate new identifiers and persist the entity. Read operations load and deserialize from the file system. Update operations modify entity state and update the modification timestamp. Delete operations remove the file and the environment's values from every profile.

**Cache Integration**: Environment data integrates with the application cache layer for performance optimization. Cache invalidation occurs on create, update, and delete operations.

#### 4.18.3 Environment CRUD Endpoints

Lifecycle operations for environments enable creation, retrieval, modification, and deletion through a consistent REST API following patterns established by other entity endpoints.

**Create Environment**: Accepts name (required, must be unique) and description (optional). Returns the created environment with generated identifier and timestamps. Returns HTTP 400 for validation failures including duplicate names.

**List Environments**: Returns all environments with pagination support. Supports sorting by name or usage. Returns environment summaries including identifier, name, description, and profile count.

**Get Environment**: Retrieves a single environment by identifier. Returns HTTP 404 if not found. Response includes full entity details and associated profile count.

**Update Environment**: Modifies environment name and/or description. Name changes validate uniqueness. Returns HTTP 400 for validation failures, HTTP 404 if not found.

**Delete Environment**: Removes an environment by identifier and removes that environment's values from every profile; callers confirm first. Returns HTTP 404 if not found.

#### 4.18.5 Tags Support

##### REQ-9.5.3.4 Environment tags aggregation endpoint

Environments support categorization through tags enabling organization by deployment stage, region, or any user-defined grouping (e.g., "production", "staging", "development", "us-east", "eu-west").

**Tags Discovery**: A dedicated endpoint returns all unique tags across all environments, sorted alphabetically. This supports filter dropdowns in the UI enabling tag-based environment filtering. The endpoint returns an array of tag strings representing the complete set of tags in use across the environment collection.

**Caching**: Tags responses cache briefly with automatic invalidation when environment entities are created, updated, or deleted. The cache supports efficient repeated queries for filter dropdown population without re-scanning the entire environment collection.

**Response Format**: The endpoint returns a success indicator and the tags array. Empty collections return an empty array, not an error.

#### 4.18.6 Environment Names Endpoint (Lightweight)

##### REQ-4.7.10 Environment selection for execution context

A dedicated lightweight endpoint supports environment selection UI components across the application (workbench, test/suite/regression execution dialogs, profile management).

**Purpose**: When users select an environment for test execution, the UI only needs environment identifiers and display names - not full entity details like descriptions, usage counts, or timestamps. The names endpoint provides this minimal data set efficiently.

**Response Characteristics**: The endpoint returns all environments without pagination, sorted alphabetically by name. Each entry contains only the identifier, name, and tags fields. This contrasts with the full list endpoint which returns paginated results with complete entity details.

**No Pagination**: Unlike the standard list endpoint, the names endpoint returns all environments in a single response. Environment collections are expected to remain small (typically under 100 entries), making full collection retrieval appropriate for dropdown population.

**Tags Inclusion**: Tags are included in the response, so a selector can show them without another API call.

**Caching**: Responses cache briefly (60 seconds) with automatic invalidation on environment create, update, or delete operations. This balances responsiveness to changes with efficient repeated queries for selector population.

**Use Cases**: The endpoint serves components that need environment selection including the workbench environment selector, test/suite/regression run dialogs, and profile management forms. These components call the names endpoint once on mount rather than the full list endpoint with pagination parameters.
