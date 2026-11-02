### 3.4 Storage Design

#### 3.4.1 Storage Strategy
The system uses a file-based storage approach:

- **Configuration Storage**: Application settings, connection profiles, and credentials stored in YAML format
- **Component Library**: Functions are the versioned library, and tests, suites and regressions keep a version history
- **Artifact Management**: Timestamped directories for execution artifacts including logs, captures, and reports
- **Temporary Storage**: Isolated space for runtime temporary files

##### REQ-6.1.1 YAML/JSON test definitions

Entities are stored as YAML at rest, and import and export take YAML or JSON. Every file the storage layer reads or writes is UTF-8, whatever the WAVE host's locale (REQ-4.6.9).

##### REQ-6.1.2 Versioned component files

Tests, suites and regressions each keep a list of their versions beside their current record. Functions are versioned as described in REQ-3.1.2.2.

##### REQ-6.1.3 Metadata in all entities

All components within the system include comprehensive metadata to support traceability, attribution, discovery, and lifecycle management throughout the component's existence.

**Identity and Versioning**: Every component includes fundamental identification metadata consisting of a unique identifier, human-readable name, and semantic version number. These three fields form the core identity of a component and enable unambiguous references throughout the system. The unique identifier follows a naming convention (typically kebab-case) and remains stable across versions of the same component.

**Descriptive Metadata**: Components include a description field providing human-readable explanation of purpose and behavior. Author attribution identifies the creator or maintainer of the component. Tags provide a flexible categorization mechanism for discovery and organization, allowing components to be grouped by features, use cases, or organizational structure without rigid hierarchical constraints.

**Temporal Metadata**: The system automatically tracks component lifecycle timestamps. Creation timestamp records when the component version was first saved to the system. Update timestamp reflects the most recent modification, enabling detection of staleness and supporting cache invalidation strategies. These timestamps use UTC timezone to ensure consistency across distributed teams and systems.

**Type-Specific Metadata**: Different component types include additional metadata relevant to their function. Plugins declare category classification (network, remote, database, system, kubernetes, helm, data) for organizational grouping.

**Metadata Preservation**: The storage layer guarantees metadata preservation across serialization formats (YAML/JSON) and storage operations (save/load/update). Datetime fields are serialized to ISO 8601 format maintaining timezone information. Optional metadata fields serialize as null rather than being omitted, providing explicit indication of absence versus default values. This ensures metadata integrity.

##### REQ-6.1.4 Artifact directory structure

The system creates dedicated directory structures for each test execution to store all artifacts generated during workflow execution, including logs, screenshots, data captures, and reports. All artifact directories live under a single configurable artifact root (see REQ-3.7.1.4).

**Timestamped Directory Naming**: Each execution receives a unique artifact directory combining UTC timestamp and execution identifier (format: `YYYYMMDD-HHMMSS-{execution_id}`). The timestamp prefix enables chronological sorting and browsing, while the execution ID ensures uniqueness and provides a stable reference for artifact retrieval even when executions occur in rapid succession. The directory name is recorded on the execution record at creation, giving every execution a direct, scan-free path to its artifacts. One execution record owns exactly one artifact directory: suite and regression executions share their single directory across all child test runs.

**Automatic Directory Creation**: The system creates the execution's artifact directory when the execution begins. Directory creation is idempotent (safe to call multiple times) and creates parent directories as needed. This eliminates manual directory management and ensures artifacts always have a consistent storage location regardless of execution environment.

**Workbench Session Artifact Area**: Interactive workbench runs (Test and Function workflows) that are not tracked executions resolve their artifacts into a temporary per-session directory inside a dedicated sessions area under the same artifact root. The session directory follows the workbench session's state semantics: it persists across consecutive runs of the same workflow, transfers to the successor session when run state is inherited, is dropped when the session is reset or closed, and is swept at backend startup since sessions do not survive a restart.

**Self-Describing Layout**: An artifact is anything inside the execution's artifact directory. The directory's contents are the complete source of artifact metadata - there is no external index, manifest, or database to keep consistent. This makes every execution directory independently relocatable.

**Per-Test Test Data Zone**: Beside the artifacts area, the storage base holds a test data zone with one folder per test, `testdata/<test id>/`. It is the home of the files a test reads through the testdata:// scheme: created on the test's first upload, removed with the test, copied when the test is cloned. Files inside are stored without write permission, so nothing but WAVE's own test data operations changes them. Artifact retention and cleanup never touch this zone: its contents are inputs owned by the test, not outputs owned by a run.

**Artifact Organization**: Within each execution directory, the system supports flexible artifact organization. Subdirectories can be created for different artifact types (logs, screenshots, data-captures, reports) based on workflow needs. This hierarchical organization facilitates efficient artifact discovery and enables different cleanup policies for different artifact types (e.g., retain reports longer than debug logs). Execution logs follow a fixed convention: on completion each test's captured log stream is written under `logs/<suite>/<test>`, so per-test execution traces are first-class, browsable artifacts alongside reports and plugin outputs.

**Artifact Retention and Cleanup**: The timestamped naming convention supports automated retention policies based on execution age. Organizations can implement cleanup scripts that delete artifact directories older than a specified threshold (e.g., 30 days for standard tests, 90 days for regression tests). The timestamp-based naming enables efficient chronological queries without parsing directory contents.

**Cross-Execution References**: Artifacts from one execution may reference artifacts from related executions (e.g., baseline comparison tests). The stable execution ID portion of the directory name provides a reliable reference mechanism. The storage layer ensures execution IDs are unique across the system lifecycle through UUID or similar generation strategies.

##### REQ-6.1.5 Configuration file hierarchy

WAVE has one configuration file per installation rather than a hierarchy of global and per-environment files. Built-in defaults are the base layer, the configuration file overrides them, and the environment variables that name a setting override the file; a setting the file does not hold takes its default, and a section that sets only some of its values keeps the defaults of the others.

**One file, found from WAVE itself**: The file lives in the config folder of WAVE's installation folder, or wherever an environment variable names it by absolute path (REQ-4.7.15). Nothing selects among several files by an environment name; a deployment that runs several WAVEs gives each its own file.

**Written by Settings**: The file is read at startup and written back when an admin saves a settings page, changing only the saved settings and keeping the file's comments and layout (REQ-4.7.16), so it always shows how this WAVE is set up.

**Example file**: WAVE ships an example that holds every setting at its default with a one-line comment; an installation creates its configuration file from it.

#### 3.4.2 Data Models

The system defines structured data models for all major entities representing the hierarchical composition of tests from executable plugins up through release validation suites.

##### REQ-6.2.1 Plugin schema with parameters

Plugins define executable components with strongly-typed parameter schemas that declare their interface and enable automated validation before execution.

**Plugin Identity and Classification**: Each plugin declares a unique identifier, descriptive name, semantic version, and category classification. Categories (network, remote, database, system, kubernetes, helm, data) facilitate plugin discovery and organizational grouping. The identity fields enable unambiguous plugin references across the system and support version pinning in block definitions.

**Parameter Schema Declaration**: Plugins declare their input parameters as structured schemas including parameter name, data type (string, integer, float, boolean, array, map, table), required/optional status, default values, and human-readable descriptions. Data types support primitives (string, integer, float, boolean), collections (array), maps (key-value dictionaries), and tables (two-dimensional matrices). This declarative approach enables the GUI to automatically generate parameter input forms and provides runtime validation before plugin execution, catching configuration errors early.

**Validation Rules**: Plugins can specify validation rules for parameters including pattern matching, value ranges, custom validation logic, and error messages. These rules are evaluated before execution, providing immediate feedback to users when parameters don't meet requirements. Validation occurs at both save-time (when configuring blocks) and runtime (before plugin execution), ensuring data integrity throughout the test lifecycle.

**Return Type Specification**: Plugins declare their return type to inform consumers what data structure will be produced. Return types can be primitives (string, integer, float, boolean), collections (arrays), maps (key-value structures), tables (two-dimensional matrices), or null for plugins with no return value. This metadata enables variable type checking and helps workflow authors understand plugin outputs when composing workflows.

**Parameter Flexibility**: The parameter system supports both required and optional parameters with default values. Optional parameters can be omitted from block configurations, falling back to plugin-defined defaults. This reduces configuration verbosity for common cases while maintaining flexibility for advanced use cases. Parameter descriptions guide users in understanding the purpose and expected format of each input.

##### REQ-6.2.2 Block schema with dependencies

##### REQ-6.2.3 Test schema

Tests represent composable sequences that orchestrate block execution in both sequential and parallel patterns, enabling complex end-to-end validation scenarios through structured composition. When a test executes, it creates an execution workflow - the runtime sequence of blocks and plugins that run from start to completion.

**Test Identity and Versioning**: Each test declares unique identification (id, name, version) following semantic versioning conventions. Tests are first-class versioned components that keep a list of their versions.

**Block Composition**: A test holds its workflow as a graph of Blocks and the connections between them.

**Variable Management**: Tests define test-scoped variables that provide configuration values and store intermediate results throughout execution. Each variable includes a name and a type, and a value where the author gave one. Variables can be defined statically with fixed values or computed dynamically using expressions that reference other test variables, namespace-prefixed variables (system, suite, regression), or block return values. Suite, regression, and system variables are accessed via explicit namespace expressions at runtime.

**Parallel Execution Branches**: Tests support parallel execution through branch definitions that specify which blocks execute concurrently and how their completion is synchronized. Each parallel branch holds the blocks participating in that branch, and the join mode releases the section when every branch has finished or as soon as the first one does. Parallel branches enable concurrent operations (e.g., simultaneous load generation and monitoring) and share the test's live variables (REQ-3.2.3.3).

##### REQ-6.2.4 Suite organization schema

Suites represent collections of related tests organized for specific testing objectives, enabling logical grouping of tests by feature area, subsystem, or validation scenario. When a suite executes, it creates an execution workflow - the runtime sequence of tests, blocks, and plugins that run from start to completion.

**Suite Identity and Purpose**: Each suite declares unique identification (id, name, version) following semantic versioning. Suites serve as organizational containers grouping related tests that validate a specific feature area, component, or functional requirement.

**Test Aggregation**: Suites aggregate tests through an ordered list of test identifiers. The ordered sequence of test references defines the default execution workflow when the suite runs sequentially.

**Suite-Level Variables**: Suites define suite-scoped variables that provide configuration values available to all tests in the suite. Suite variables are accessed explicitly via namespace references in test expressions (e.g., `suite.deploy_env`, `suite.timeout`) following the same pattern as profile references (`prof_alias.param`). Test authors explicitly reference suite variables by name, making dependencies visible and self-documenting. Suite variables resolve at execution time based on which suite is running the test. System variables are accessed via `system.var_name` and regression variables via `regression.var_name`.

**Reserved Variable Names**: The namespace identifiers `suite`, `regression`, and `system` are reserved keywords and cannot be used as test-level variable names. This prevents ambiguity between test-local variables and namespace references in expressions. Variable name validation rejects these reserved names at creation time with a clear error message.

**Organizational Metadata**: Suites include descriptive metadata (description, author, tags) supporting discovery and documentation. Tags enable flexible categorization by feature area, priority, test type, or team ownership without rigid hierarchical constraints. This metadata-driven organization supports dynamic test suite selection in CI/CD pipelines and facilitates maintenance responsibility tracking.

**Suite Composition Flexibility**: Suites can reference any number of tests in any order, enabling both comprehensive validation suites that test entire subsystems and focused suites that validate specific scenarios. Tests are independent entities that can be referenced by multiple suites (many-to-many relationship). This enables shared prep/teardown tests across suites and mini-suites that reference subsets of a larger suite's tests without duplication.

##### REQ-6.2.5 Regression collection schema

Regressions represent collections of test suites for comprehensive release validation.

**Regression Identity and Release Alignment**: Each regression declares unique identification (id, name, version) with semantic versioning that typically aligns with product release versions. Regressions serve as release gates, defining the complete validation criteria that must pass before a product version is considered release-ready.

**Suite Aggregation**: Regressions aggregate suites through an ordered list of suite identifiers with no version, and a run uses what each suite holds at that moment.

**Regression-Level Variables**: Regressions define regression-scoped variables providing environment configuration, infrastructure references, and validation thresholds available to all suites. These variables are accessed explicitly via namespace references in expressions (e.g., `regression.release_version`, `regression.deploy_target`) following the same explicit namespace pattern used for suite and system variables. Typical regression variables include deployment endpoints, credential profiles, timeout limits, and acceptance criteria that apply broadly across all validation.

**Release Validation Workflow**: Regressions execute as comprehensive validation workflows, running all referenced suites in order and aggregating results into pass/fail outcomes for the entire regression. Failed suite execution can optionally halt the regression (fail-fast) or continue executing remaining suites for comprehensive failure analysis. Result artifacts capture complete execution history enabling post-mortem analysis and compliance documentation.

#### 3.4.3 Version Management

##### REQ-6.3.1 Semantic versioning for components

Functions follow semantic versioning, and it is enforced for them. Plugins and controls declare a version, and tests, suites and regressions bump their version automatically.

##### REQ-6.3.2 Version pinning capability

##### REQ-6.3.3 Dependency resolution

Function dependency checks are described in REQ-3.1.2.3.

#### 3.4.4 API Entity Persistence

API entities (tests, environments, profiles, executions) use a caching storage pattern that combines file-based persistence with in-memory caching for performance.

##### REQ-6.1.6 API entity file storage

API entities persist to file-based YAML storage enabling data survival across server restarts while maintaining fast access through in-memory caching.

**Storage Directory Structure**: Each entity type has a dedicated storage directory under the configurable base path. Within each type directory, individual entities are stored in subdirectories identified by their unique ID. This structure enables efficient entity enumeration, individual entity access, and clean deletion operations.

```
{base}/
├── api_tests/{id}/test.yaml
├── api_environments/{id}/environment.yaml
├── api_profiles/{id}/profile.yaml
├── api_executions/{id}/execution.yaml
└── ...
```

**Caching Strategy**: The service layer maintains an in-memory cache of all entities for fast read operations. The cache is lazily populated on first access by scanning the storage directory and loading all entity files. An entity that is cached and still present on disk is served from the cache without re-reading its file. Write operations update both cache and disk to maintain consistency.

**Cache Lifecycle**: The cache starts empty when the service initializes. On first read operation, the system scans the storage directory, loads all entity files, and populates the cache. The storage directory is the record and the cache follows it: for tests, environments and profiles, every listing operation reconciles the cache against the directory, dropping cached entries whose record file is gone and loading record files the cache does not yet hold. This keeps listings true when files are added or removed by something other than the product, such as a branch checkout, a restored backup or a hand edit. Reading a single entity whose record file has disappeared reports not found and drops that cache entry. For testing scenarios, utilities exist to clear the cache without affecting disk storage (simulating restart) or to clear both cache and disk storage (resetting to empty state).

**Execution Listing Exception**: Executions are the one entity type whose listing is not reconciled against the directory. Execution records accumulate without bound, scanning that directory costs far more than the listing itself, and a cached execution record is deliberately ahead of its file while the run is in progress. An execution directory removed outside the product stays listed until something reads that execution, which reports not found and drops it, after which the listing is correct.

**Persistence Operations**: Entity creation writes a new YAML file and updates the cache. Entity updates modify the existing YAML file and update the cache entry. Entity deletion removes the YAML file (and its parent directory) and removes the cache entry. All operations through the product keep cache and disk synchronized, and changes made directly to the storage directory are picked up by the reconcile described above.

**Error Recovery**: If an entity file becomes corrupted or unreadable, the system logs a warning and excludes that entity from the cache. This prevents one corrupted file from blocking access to all entities. Administrative tools can identify and repair or remove corrupted entity files.

##### REQ-6.1.7 Profile persistence

Connection profiles require persistent storage to preserve configuration across server restarts and enable consistent profile references from tests and workflows.

**Profile Storage Format**: Each profile is stored as a single YAML file containing all profile data including identity fields (id, name, type), configuration values, environment-specific overrides, tags, and timestamps. The YAML format provides human-readable storage enabling manual inspection and editing when needed.

**Secret Field Storage**: Profile configurations may contain secret fields (passwords, API keys, private keys, tokens). Their values are sealed at rest rather than stored as plaintext in the file (REQ-4.5.1).

**Environment Values Structure**: Profiles support environment-specific configuration through an environment values map. Each environment can have its own set of configuration values that override the profile's default configuration when executing in that environment. This structure persists as nested maps within the profile YAML file.

**Usage Tracking**: Profile usage counts (number of tests/workflows referencing the profile) are computed dynamically at retrieval time rather than stored. This avoids stale counts when referencing entities change and ensures accurate usage information without cross-entity update coordination.

##### REQ-6.1.8 Execution persistence

Execution records require persistent storage to preserve state, logs and variables across server restarts, enabling history viewing and recovery from unexpected termination.

**Multi-File Storage Structure**: Unlike single-file entities, executions use a multi-file structure separating core state from logs and variables. This enables independent access to each data category without loading the entire execution state.

```
{base}/api_executions/{execution_id}/
├── execution.yaml          # Core state: id, entity_type, entity_id, status, timestamps
├── composition.yaml        # Suite and test tree, written once
├── runs.yaml               # One entry per dispatched test
├── status.yaml             # One entry per test state transition
├── logs.yaml               # Regression and suite log entries, and a reference to each test's log
├── logs/<run_id>.yaml      # Per-test log entries
├── snapshots/<run_id>.yaml # Per-test terminal snapshot
└── variables.yaml          # End-of-run variables (secrets masked), test executions only
```

**Caching Strategy**: Core execution metadata is cached for fast status queries and list operations. Log data loads on-demand only when requested, avoiding memory overhead for large execution histories.

**Orphan Recovery**: On server startup, the storage layer scans for executions in running or paused states. These orphaned executions are marked stopped with recovery logged, ensuring accurate final states rather than perpetually running entries. Orphan recovery never auto-resumes.

**Persistence Operations**: State transitions persist immediately after validation, ensuring state machine consistency survives restarts. Deletion removes all associated files and is only permitted for terminal states (passed, failed, stopped).

#### 3.4.5 Artifact Generation & Storage

WAVE-generated outputs (failure captures, reports, exports) are produced through a single artifact store that writes into the owning execution's artifact directory under a plain logical name. Each artifact is single-instance per execution: regenerating an output is an explicit delete-then-write, never a new version. The store provides serialization validation and optional compression. Plugin-written files share the same directories through the artifact:// path scheme but are plain filesystem writes - WAVE imposes no structure or compression on plugin output.

##### REQ-3.7.1.1 JSON artifact generation

The artifact store generates JSON artifacts from structured execution data. Content is serialized and validated in memory before any I/O occurs, catching non-serializable values and circular references before a partial file can reach disk. Text and binary artifacts are supported through the same write surface for non-JSON outputs.

The framework's failure capture is a JSON artifact: a failed execution produces a single structured error summary that aggregates the error and fatal log entries that caused the failure, each correlated to the originating block and scope. This gives post-mortem analysis a machine-readable causal failure summary alongside whatever files the workflow itself produced.

##### REQ-3.7.1.3 Artifact compression

Each WAVE-generated artifact write can select a compression mode: none, gzip, or zip. Compression is applied after complete artifact generation, and compressed artifacts carry the corresponding file extension so the compression state is visible in the directory listing. Compression failures fall back to uncompressed storage rather than losing data. Plugin output is never compressed by the framework.

##### REQ-3.7.1.4 Store artifacts in configurable temp folder

All artifact directories live under a single configurable artifact root. The location is resolved from an environment override when present, defaulting to an artifacts area inside the storage base path. The root contains the per-execution timestamped directories and the sessions area holding temporary per-session directories for workbench runs.

Because each execution's directory is isolated, self-describing, and relocatable, the artifact root can be placed on dedicated or temporary storage with large capacity, and a complete execution (its append-only record plus its artifact directory) can be bundled, moved to external storage, and later reviewed as proof of a run.

##### REQ-4.7.9 Test artifacts directory configuration

Files live in two zones under the storage base. The artifacts zone holds a run's outputs: its root is the artifacts area inside the storage base, or the existing environment override for the artifact root when one is set, and it contains the per-execution timestamped directories and the workbench sessions area. The test data zone holds each test's own input files, one folder per test in a dedicated area directly under the same storage base; a test's folder is created on its first upload and removed with the test, and the zone takes no override of its own.

Both roots are launch-time facts of the storage base. The UI shows them as they are - the Artifact Browser and the Test Data sheet address files inside them by relative path - and never edits them. There is no configuration key and no environment variable for test data beyond the storage base setting that already places everything else.

#### 3.4.6 Data Folder and Application Settings

WAVE keeps two kinds of state apart. The data folder holds shared work, laid out so that several WAVE instances can share it as a repository of its own. The config file holds this WAVE's own settings: what an installation fixes before WAVE starts and what an admin changes in Settings while WAVE runs. Browser preferences stay in each browser.

##### REQ-4.7.8 Application settings configuration

**Installation settings**: The config file holds what an installation fixes before WAVE starts: the data folder, the browser origins the API accepts, the secret encryption key, the server-side folder recipe secrets may be read from, and whether error responses carry detail. Values resolve from built-in defaults, then the config file, then the environment variables that name a setting, each overriding the one before it (13.2). Settings shows the ones an admin needs to see read-only and lock-marked, because WAVE cannot change what it was started with (WSRV-2).

**Settings changed in WAVE**: The same file holds what an admin changes in Settings while WAVE runs (REQ-4.7.17): the log level, secret masking, the SSH lock and default host key action, the public certificate authorities source, the proxy, the MCP server and the periodic backup schedule. A value the file does not hold takes its built-in default, and a section that sets only some of its values keeps the defaults of the others. Settings writes these values back into the file when the admin saves a page (REQ-4.7.16), so they survive a restart and the file stays the one place that shows how this WAVE is set up. They describe one WAVE and are never part of the shared work in the data folder.

##### REQ-4.7.13 Data folder set at installation

The data folder holds the shared work of WAVE: the entities and their versions, test data, the trusted SSH hosts list and the Additional Certificates, together with the run history and artifacts WAVE writes. This WAVE's own settings are not in it; they live in the config file (REQ-4.7.17). Its location is a required setting of the config file, written there when WAVE is installed as a step of the installation instructions. A relative value starts at the folder that holds the config file, never at the folder WAVE was started from, so starting WAVE from another folder can never create an empty data folder and present an empty WAVE. The environment-variable override of the location stays for deployments that place the data folder at launch, and it takes an absolute path only.

When the location is not set, WAVE refuses to start and its message names the setting to add. At startup WAVE writes one log line naming the absolute path of the data folder in use. Settings > WAVE Server shows the same absolute path in the Backend Server card as a lock-marked, read-only and copyable Data folder row, because the data folder is fixed when WAVE starts and is not changed from within it (WSRV-10).

##### REQ-4.7.15 One config file per WAVE

WAVE reads its settings from one config file. By default it is the file in the config folder of the folder WAVE is installed in; WAVE finds that folder from where its own code lives, never from the folder it was started from, so starting WAVE from another folder always reads the same file. An environment variable names another file instead, by absolute path, for deployments that keep their configuration elsewhere and for test stacks that each need a file of their own; a relative path is refused because it would depend on the folder WAVE was started from. There is one file per WAVE: no environment name selects among several.

WAVE refuses to start when the file is missing, cannot be read, does not hold a mapping of settings, or holds a setting it cannot accept, and its message names the file and the setting. For a file that does not parse, the message names the line and what the parser found there, in one line, and never quotes a value from the file: a value may be a secret. At startup WAVE writes one log line naming the absolute path of the file in use, before the line naming the data folder, and Settings > WAVE Server shows the same path in the Backend Server card as a lock-marked, read-only, copyable Config file row beside the Data folder row (WSRV-11). Environment variables still override the settings they name, the log level among them (REQ-4.7.18).

WAVE ships an example file that holds every setting at its default with a one-line comment, and an installation creates its config file from it (13.1).

##### REQ-4.7.16 Settings saved into the config file

Every Settings page that holds settings - General, SSH Hosts, Certificates, Proxy, MCP Server and the periodic backup schedule - has Cancel and Save. A change on the page changes only the page: Save and Cancel are disabled and faded until something changed, Save writes the page's changes in one step, Cancel restores the saved values, and both are disabled again afterwards. Leaving a page that holds unsaved changes - through the Settings section list, the app's navigation or the browser's back - asks first: stay, discard the changes and leave, or save and leave once the save succeeds, staying when it fails; a save that needs the page's own confirmation, such as restarting the MCP endpoint, asks that confirmation too. Shared work edited on the same page, such as the trusted hosts or the Additional Certificates, is written to the data folder by the same Save; when that write fails, the settings already written to the config file are put back, so a Save is all or nothing. If putting them back is refused as well, the page still shows the first failure and the server log records that the config file kept the new values.

**What a Save keeps**: A Save keeps only the settings the page shows for the choices being saved. A setting the page hides goes back to its default on Save, and a password stored for it is removed: the Custom proxy's address, port, login, address list and local-addresses switch while the proxy is off or follows the WAVE host's settings, and the schedule's fields while periodic backup is off. Turning the feature on again therefore starts from its defaults. A value typed into a field that is then hidden is neither checked nor kept, so it never blocks Save. What the config file itself holds for a hidden setting, typed into the file by hand, is not read either, and the next Save of that page replaces it with the default. A record WAVE writes itself, such as the result of the last scheduled backup, is not a setting and stays. A field that is shown faded or disabled is still shown, and its value is kept.

Save changes only the settings the page saved. WAVE reads the file again immediately before writing, so an edit made by hand since WAVE started is kept, and the file keeps its comments, the order of its settings, its layout and its line ends. The new file replaces the old one in one step and keeps the old file's permissions, so a crash never leaves half a file; a config file that is a link is written through to the file it points at.

When the config file cannot be written - it is read-only for WAVE, its folder cannot take the new file, or a hand edit has left it unreadable - Save is refused with a message that names the file and the reason, the file is left as it was, and nothing changes in the running WAVE. The running WAVE holds exactly what the file holds for these settings and takes a saved value only after the file has it.

A Save never writes a setting the next start would refuse. Every saved value is checked against the same rules the start applies, together with the settings already in force; a value that fails is refused with the rule's own words, and nothing is written.

##### REQ-4.7.17 Shared work and this WAVE's own settings

The data folder holds shared work: tests, suites, regressions, Functions, profiles, environments, system variables, test data, the trusted SSH hosts list and the Additional Certificates. It is laid out so that several WAVE instances can share it as a repository, so nothing in it depends on one WAVE's preferences or on one WAVE host's network.

The config file holds this WAVE's own settings: the log level (REQ-4.7.18), the four secret masking controls (REQ-3.1.3.17), the SSH lock and default host key action (REQ-4.5.14), the choice of public certificate authorities (REQ-4.5.16), the proxy (REQ-4.7.14), the MCP server (REQ-5.2.7.1) and the periodic backup schedule with the result of its last run (REQ-4.7.20), beside the installation settings (REQ-4.7.8). Every new stored setting follows the same rule: shared work goes to the data folder; a preference of this WAVE, or anything that depends on the WAVE host's network, goes to the config file.

Browser preferences - the time zone (REQ-4.7.19), the theme, the sidebar, page sizes, columns, console display and workbench choices - stay in each browser and never reach the server.

##### REQ-4.7.18 Log level from the config file

The log level is a setting of the config file, one of DEBUG, INFO, WARN, ERROR and FATAL. An environment variable overrides it when WAVE starts, for a deployment that wants another level without editing the file; a value WAVE does not recognize in either place stops startup with a message naming the accepted levels. Settings > General shows the level in force. Save applies the new level to every backend logger at once and writes it to the config file; when the environment variable is set, it still wins at the next start.
