### 4.8 Regression Suite Endpoints

Regression suite endpoints provide lifecycle operations for regressions - the top-level organizational containers in the WAVE hierarchy (Regression - Suite - Test - Block - Plugin). Regressions group related suites into coherent execution campaigns with variable scoping.

#### 4.8.1 Lifecycle Management

##### REQ-5.2.4.5 Regression suite operations endpoints

**CRUD Operations**: The system supports full CRUD operations with versioning. Regression creation establishes metadata including name, description, suite references with ordering, variable declarations, and tags. Retrieval supports individual access by identifier and bulk listing with filtering by search term. Deletion enforces dependency checking, returning conflict when regression contains active execution references.

**Version Control**: Every regression maintains version history following semantic versioning enabling temporal tracking and rollback. Suite composition modifications create new versions preserving historical snapshots. Version creation validates suite reference compatibility and ordering consistency. Version operations include listing, retrieving, adding, updating metadata, and deleting with dependency analysis. Published regression versions become immutable, ensuring stable references.

**Suite Composition**: A regression holds an ordered list of suites. Membership and order are saved through the regression update, and a dedicated endpoint lists the regression's suites.

**Discovery and Filtering**: List operations support page-based pagination. Filtering includes text search across names and descriptions. Responses include essential metadata including suite counts, variable declarations, and version information.

#### 4.8.2 Variable Management & Collision Validation

Regression-level variables provide campaign-wide configuration values accessed via namespace expressions (e.g., `regression.release_version`) in test workflows. Variable support includes typed variables, expression-based computed variables, and secret variables with masking indicators.

**Collision Detection**: When regression variables are created or modified, the system validates that same-named variables across all referenced suites maintain consistent types. Collisions are detected before persistence - validation runs against the proposed state, not the saved state. This follows the same validation pattern established for suites.

**Validation Entry Points**: Three paths trigger collision validation: (1) regression create, (2) regression update - the primary UI path for all modifications including variable edits and suite membership changes, and (3) regression import. All three validate before persisting to avoid rollback scenarios.

**Error Response**: Collision errors return structured details including the conflicting variable name, the type declared in the regression, the conflicting type in the suite, and the suite identifier. This enables the frontend to display actionable error messages with resolution options.

**Pre-merged Variables in Test Detail**: The test detail endpoint returns regression variables - pre-merged variable metadata from all regressions containing the test's parent suites. This eliminates the need for multiple frontend requests to resolve regression-level variable context, following the same optimization applied to suite variables.

#### 4.8.3 Import/Export

##### REQ-5.2.1.37 Regression import and export operations

Regression import/export enables portability across WAVE instances, following the same architectural pattern as suite import/export.

**Export**: Regression export serializes the definition including metadata, suite references (with ordering and version pinning), variable declarations, and tags. Export supports YAML and JSON formats with optional secret masking. Suite references are exported as identifiers - the actual suite definitions are not bundled (suites must exist independently on the target instance). Single regression export produces a flat file; batch export produces a ZIP archive.

**Import**: Regression import validates the incoming definition before persisting. Validation includes name uniqueness checking, suite reference resolution (all referenced suites must exist), variable collision validation (see 4.8.2), and structural integrity checks. When naming conflicts arise, the import system supports conflict resolution strategies: skip, update (overwrite), or create as new. Import operates as a two-phase process - a dry-run validation endpoint returns importability status and conflicts, followed by the actual import endpoint that accepts conflict resolution decisions.

#### 4.8.4 Tags Support

##### REQ-9.5.3.3 Regression tags aggregation endpoint

Regressions support categorization through tags enabling organization by release cycle, environment target, or any user-defined grouping. A dedicated endpoint returns all unique tags across all regressions, sorted alphabetically, supporting filter dropdowns in the UI. Empty collections return an empty array, not an error.

#### 4.8.5 Cross-Cutting Concerns

**Validation Strategy**: Pre-save validation verifies structural integrity including suite reference existence, ordering consistency (no gaps or duplicates), variable collision detection (see 4.8.2), and naming uniqueness. Validation returns detailed errors and warnings, operating idempotently without side effects.

**Caching and Performance**: Tag, list and single regression responses carry a 60-second cache lifetime. A version's response carries 60 seconds for the latest version and one hour for a historical one.

**Error Handling**: Operations enforce consistency through validation and transactional semantics. Naming conflicts return conflict status. Deletion with active execution references returns conflict with references. Missing entities return not found. Malformed requests return bad request with validation details. Suite reference and ordering validation failures return detailed diagnostics. Internal errors return server error with correlation identifiers. Optimistic concurrency control prevents conflicting modifications.
