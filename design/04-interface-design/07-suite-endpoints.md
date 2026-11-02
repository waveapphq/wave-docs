### 4.7 Suite Management Endpoints

#### 4.7.1 Suite CRUD Operations

##### REQ-5.2.4.4 Test suite management endpoints

Suite management endpoints provide lifecycle operations for test suites enabling creation, modification, version control, test composition, ordering, and validation. Suites are organizational containers that group related tests into coherent execution units, supporting variable scoping inheritance and execution sequencing within the test hierarchy of Regression → Suite → Test → Block → Plugin.

**Lifecycle Management**: The system supports full CRUD operations with versioning. Suite creation establishes metadata including name, description, test references with ordering, variable declarations, and associated metadata. Suites maintain ordered test lists preserving execution sequence. Retrieval supports individual access by identifier and bulk listing with filtering by search term. Deletion enforces dependency checking to prevent orphaned regression references, returning conflict when suite is referenced by regressions.

**Version Control**: Every suite maintains version history following semantic versioning enabling temporal tracking and rollback. Test composition modifications create new versions preserving historical snapshots. Version creation validates test reference compatibility and ordering consistency. Version operations include listing all versions, retrieving specific versions, adding new versions, updating version metadata, and deleting versions with dependency analysis. Published suite versions become immutable, ensuring stable references for regressions.

**Test Composition**: Suite-test association endpoints manage the relationship between suites and tests. Adding tests validates test existence before creating the association with optional ordering position. Removing tests preserves suite integrity by updating ordering indices for remaining tests. The test order is saved with the suite.

**Variable Management**: Suite-level variables provide configuration values accessed via namespace expressions (e.g., `suite.deploy_env`) in test workflows. Variable support includes declaration of typed variables, expression-based computed variables, and secret variables with masking indicators. Variable binding validation ensures type consistency at suite boundaries.

**Discovery and Filtering**: List operations support page-based pagination for consistent navigation. Filtering includes text search across names and descriptions. Responses include essential metadata enabling quick capability assessment including test counts, variable declarations, and version information.

**Validation**: Pre-save validation verifies structural integrity including test reference validation (existence verification for all referenced tests), ordering consistency (no gaps or duplicates in sequence), variable binding completeness, and naming uniqueness. Validation returns detailed errors and warnings, operating idempotently without side effects.

**Caching and Performance**: Tag, name and list responses carry a 60-second cache lifetime. A version's response carries 60 seconds for the latest version and one hour for a historical one.

**Error Handling**: Operations enforce consistency through validation and transactional semantics. Naming conflicts return conflict status, deletion with regression references returns conflict with references, missing entities return not found, malformed requests return bad request with validation details, test reference validation failures return detailed test identifiers, ordering validation failures return index analysis, and internal errors return server error with correlation identifiers. Optimistic concurrency control prevents conflicting modifications.

#### 4.7.2 Tags Support

##### REQ-9.5.3.2 Suite tags aggregation endpoint

Suites support categorization through tags enabling organization by test category, feature area, or any user-defined grouping.

**Tags Discovery**: A dedicated endpoint returns all unique tags across all suites, sorted alphabetically. This supports filter dropdowns in the UI enabling tag-based suite filtering. The endpoint returns an array of tag strings representing the complete set of tags in use across the suite collection.

**Caching**: Tags responses cache briefly with automatic invalidation when suite entities are created, updated, or deleted. The cache supports efficient repeated queries for filter dropdown population without re-scanning the entire suite collection.

**Response Format**: The endpoint returns a success indicator and the tags array. Empty collections return an empty array, not an error.

#### 4.7.3 Suite Names Endpoint (Lightweight)

##### REQ-9.5.3.5 Suite names for dropdown population

A dedicated lightweight endpoint supports suite selection UI components, specifically the Suites filter dropdown on the Tests list page.

**Purpose**: When users filter tests by suite, the UI only needs suite identifiers and display names - not full entity details like descriptions, test counts, status, or timestamps. The names endpoint provides this minimal data set efficiently, following the same pattern as `GET /environments/names`.

**Response Characteristics**: The endpoint returns all suites without pagination, sorted alphabetically by name. Each entry contains only the identifier and name fields. This contrasts with the full list endpoint which returns paginated results with complete entity details.

**No Pagination**: Unlike the standard list endpoint, the names endpoint returns all suites in a single response. Suite collections are expected to remain manageable, making full collection retrieval appropriate for dropdown population.

**Caching**: Responses cache briefly (60 seconds) with automatic invalidation on suite create, update, or delete operations.

**Use Cases**: The endpoint serves the Suites filter dropdown on the Tests list page.
