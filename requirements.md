# WAVE Engineering Requirements Specification

## Document Information
- **Version**: 1.0.0-draft1
- **Date**: 2025-10-10
- **Status**: Draft
- **Author**: WAVE maintainers

## Revision History

| Version | Date | Author | Description |
|---------|------|--------|-------------|
| 1.0.0-draft1 | 2025-10-10 | WAVE maintainers | Initial requirements specification |

---

## 1. Introduction

### 1.1 Purpose
This document specifies the engineering requirements for WAVE (Workflow Automation and Validation Engine), a comprehensive automation and testing framework for distributed systems, with strong support for Kubernetes environments and event processing pipelines.

### 1.2 Scope
The system provides dual-interface (CLI/GUI) test orchestration with visual workflow design, interactive debugging, and comprehensive validation across multiple system layers.

### 1.3 Definitions
- **Plugin**: Core executor providing specific functionality (SSH, K8s API, HTTP)
- **Function**: Parameterized, reusable, isolated subgraph component built from plugins, controls, and child functions; configured per-instance through the Function Block's input fields and output bindings; available as either a Test-local definition or an immutable shared-library entry.
- **Test**: Composable workflow of blocks with sequential/parallel execution
- **Suite**: Collection of related tests
- **Regression**: Version-controlled collection of test suites for release validation

## 2. System Overview

### 2.1 Architecture Components
- Backend API (Python/FastAPI)
- Web Frontend (React/React Flow)
- CLI Interface
- Plugin System
- Storage Layer (File-based, DB-ready)

### 2.2 Deployment Context
- Runs on Linux host with K8s API and ingress access
- No in-cluster modifications
- SSH access to nodes for packet capture

## 3. Functional Requirements

### 3.1 Test Composition

#### 3.1.1 Visual Workflow Builder
- REQ-3.1.1.1: Drag-and-drop interface for block arrangement
- REQ-3.1.1.2: Sequential flow connections between blocks
- REQ-3.1.1.3: Parallel branch creation and merging
- REQ-3.1.1.4: Nested block composition (blocks containing blocks)
- REQ-3.1.1.5: Block parameter configuration forms (every Block - control, plugin, Function - keeps, edits, checks and runs its fields one way; a free-entry field nobody set shows its default in gray in literal and expression mode, switching modes copies nothing in, Save stores only what the user set, and the run fills the rest from the current default; a choice list or true / false holds a value from the moment the Block is created; a reset icon inside the field takes it back to not set)
- REQ-3.1.1.6: Expandable/collapsible blocks for granularity control
- REQ-3.1.1.7: Expanded blocks display detailed internal flow
- REQ-3.1.1.8: Collapsed blocks show simplified flow view
- REQ-3.1.1.9: Expand/collapse all blocks globally
- REQ-3.1.1.10: Expand/collapse one level from current view
- REQ-3.1.1.11: Maintain flow visibility at all expansion levels
- REQ-3.1.1.12: Workbench loading experience with centered canvas initialization
  - REQ-3.1.1.12.1: Loading overlay during test fetch and canvas preparation
  - REQ-3.1.1.12.2: Canvas centered on Start node for blank/draft workflows
  - REQ-3.1.1.12.3: Smooth transition from loading to ready state
  - REQ-3.1.1.12.4: No visual flicker of intermediate canvas states

#### 3.1.2 Function Management
- REQ-3.1.2.1: Create custom Functions with input parameters
- REQ-3.1.2.2: Version control for Functions with metadata
- REQ-3.1.2.3: Function dependency specification
- REQ-3.1.2.4: Function validation before save
- REQ-3.1.2.5: Import and export Functions as files
- REQ-3.1.2.6: Optional log level override per Function instance
- REQ-3.1.2.7: Run-log warning for an unused input or an unwritten output (every run of a Function Block writes a WARN to the run console log for each input no block in it uses and each output no block writes; it never blocks saving, running or publishing)
- REQ-3.1.2.8: Function output kind (an output is Literal, Expression, or Template for text, like a variable; an expression output is evaluated once, at its first read or when the Function returns, and counts as written)
- REQ-3.1.2.9: Secret input and output parameters (a secret input acts as a block's secret field: any value passed in is masked inside the Function and the variable passed in is never made secret; a secret output seeds Secret ticked in the Function Block's outputs table, and its default is sealed at rest and masked everywhere)
- REQ-3.1.2.10: Inputs with no default and the value of a run on its own (a Required input may have a default, and a Required true/false input must have one, as an enum input must; a Function Block never falls back to it when it runs and must give the value, so the default is used only when the Function runs on its own, with no Function Block running it; the one exception is a Required choice-list or true/false input, which a Function Block starts on the input's default, written when the field appears; a secret input has no default; running a Function alone uses the Current value set before the run, also right after Reset, and a Required input with no value stops the run and names it)
- REQ-3.1.2.11: Enum inputs (an enum input's type is String, Integer or Float and its allowed values are of that type, offered as a choice in the Function Block, and checked on Save for a picked value and when the run starts for a variable or expression; an enum input is always Required, never secret and never null, and its default is required and is one of the allowed values)

#### 3.1.3 Variable System
- REQ-3.1.3.1: Variable scoping with namespace-based access (system, regression, suite, test)
- REQ-3.1.3.2: Self-contained test variables with namespace access
- REQ-3.1.3.3: Runtime variable passing between blocks
- REQ-3.1.3.4: Environment variable integration
- REQ-3.1.3.5: Secret variable masking in logs
- REQ-3.1.3.6: Return values from plugins (scalar, array, hash)
- REQ-3.1.3.7: Return values from blocks (scalar, array, hash)
- REQ-3.1.3.8: Return values from tests (scalar, array, hash)
- REQ-3.1.3.9: Return values from suites (scalar, array, hash)
- REQ-3.1.3.10: Store return values as variables for reuse
- REQ-3.1.3.11: Variable watching UI panel
- REQ-3.1.3.12: Auto-refresh watched variables on change
- REQ-3.1.3.13: Computed variables with expressions and runtime evaluation
- REQ-3.1.3.14: Exact numeric precision for integer and float variables (arbitrary precision, no IEEE-754 rounding)
- REQ-3.1.3.15: Template variables (a string variable whose `{{ }}` pieces are filled when it is read)
- REQ-3.1.3.16: Per-block protocol-field masking control (a plugin declares the exact fields it knows are credential-bearing; each is masked by default and individually revealable per block)
- REQ-3.1.3.17: System-wide secret reveal controls (four controls, each off by default, decide where secrets may be shown across the deployment: run output, a test's variable values in the workbench, the global, regression and suite variable editors, and plugin credential input fields; the run-output control is the default every block's run-output masking inherits; revealing never touches exports or encryption at rest; kept in this WAVE's config file and changed by saving Settings > General)
- REQ-3.1.3.18: Declared credential input fields (a plugin declares which of its input fields carry credentials; a literal typed into one is sealed at rest and write-only, an expression follows the referenced variable's secrecy, and each field shows a protection-status indicator - declaration-driven, with no user-facing secret toggle on a plugin's input fields; a Function input its author marks secret acts as such a field, REQ-3.1.2.9)
- REQ-3.1.3.19: Declared element types for array and map values (each array item and map value carries the type its author declared - string, integer, float, boolean, array or map - stored beside the value, kept by every surface that edits, stores, transfers or reads the value back, and deciding the element's run-time type; a type is never taken from how the text looks)
- REQ-3.1.3.20: Null is the only empty value (every variable holds a value; null is a value of every type, distinct from empty and zero, and keeps the declared type)
- REQ-3.1.3.21: Block settings and Function inputs refuse null (a setting or input given null fails and names it, unless it is a data field that accepts null or a Function input that is optional with no default; a null item, map value or table cell reaches the Block, which uses it its own way or fails naming the field and the entry)
- REQ-3.1.3.22: Null in text, scripts and SQL (a null `{{ }}` piece fails and names the piece, a null reaching a script as text fails before the script starts, and SQL sends empty text as empty text and null as NULL)
- REQ-3.1.3.23: One null across languages (Python None, SQL NULL and JSON null become WAVE's null and back, and null prints as `null`)
- REQ-3.1.3.24: Set to null by hand (a "Set to null" choice in the one window for adding and editing a variable, Function local variable, Function input or Function output, for the Initial value (an input's default) and the Current value; hidden while Kind is Expression or Template; off on a Required or enum input; forced on a secret input)
- REQ-3.1.3.25: Checks and loops on null (a check treats null as holding nothing, "is null" and "not null" are offered, and loops run zero times on null)

### 3.2 Test Execution

#### 3.2.1 Execution Modes
- REQ-3.2.1.1: Normal execution (run to completion)
- REQ-3.2.1.2: Debug mode with breakpoint support
- REQ-3.2.1.3: Step-by-step execution (step-over, step-into)
- REQ-3.2.1.4: Block re-execution during pause
- REQ-3.2.1.5: Ad-hoc block execution
- REQ-3.2.1.6: Runtime variable override during pause/stop (Workbench debug)

#### 3.2.2 Execution Control
- REQ-3.2.2.1: Set breakpoints before/during execution
- REQ-3.2.2.2: Skip blocks during runtime
- REQ-3.2.2.3: Pause/resume execution
- REQ-3.2.2.4: Cancel execution with cleanup
- REQ-3.2.2.5: Real-time progress monitoring
- REQ-3.2.2.15: Live flow visualization during execution
- REQ-3.2.2.16: Highlight currently executing blocks
- REQ-3.2.2.17: Real-time dashboard with metrics
- REQ-3.2.2.18: Live variable values display
- REQ-3.2.2.6: Try-catch error handling per block
- REQ-3.2.2.7: Configurable retry logic (count, delay, backoff)
- REQ-3.2.2.8: Retry on specific error conditions
- REQ-3.2.2.9: Continue execution after caught errors
- REQ-3.2.2.10: Finally/finalize blocks for guaranteed cleanup
- REQ-3.2.2.11: Finalize blocks execute on normal termination
- REQ-3.2.2.12: Finalize blocks execute on abnormal termination
- REQ-3.2.2.13: Test-level finalize blocks
- REQ-3.2.2.14: Block-level finalize blocks

#### 3.2.3 Parallel Execution
- REQ-3.2.3.1: Execute parallel branches within single test
- REQ-3.2.3.2: Synchronization point for branch convergence
- REQ-3.2.3.3: Branches share the test's live variables
- REQ-3.2.3.4: Atomic variable access under concurrent branches
- REQ-3.2.3.5: Nested parallel sections at any depth
- REQ-3.2.3.6: Configurable join mode (wait-for-all, first-to-finish)
- REQ-3.2.3.7: Optional time limit per branch and per section
- REQ-3.2.3.8: Branch failure ends the section and stops running siblings
- REQ-3.2.3.9: Debug pause parks the whole parallel section

### 3.3 Validation Capabilities

#### 3.3.1 Assertion Types
- REQ-3.3.1.1: Exact string matching
- REQ-3.3.1.2: Regular expression patterns
- REQ-3.3.1.3: JSON path expressions
- REQ-3.3.1.4: Count-based validations (equal, less, greater)
- REQ-3.3.1.5: Configurable failure behavior (stop/continue)

#### 3.3.2 Log Validation
- REQ-3.3.2.1: Collect logs from pod containers
- REQ-3.3.2.2: Collect logs from OpenSearch with query filtering
- REQ-3.3.2.3: Filter logs by time range
- REQ-3.3.2.4: Multi-container log aggregation
- REQ-3.3.2.5: OpenSearch index pattern selection (wildcards, date math)
- REQ-3.3.2.6: OpenSearch field-based filtering and queries

#### 3.3.3 Data Validation
- REQ-3.3.3.1: Retrieve JSON events from OpenSearch index patterns
- REQ-3.3.3.2: OpenSearch statistics validation
- REQ-3.3.3.3: SQL database state checks
- REQ-3.3.3.4: LDAP entry validation
- REQ-3.3.3.5: Response body validation (JSON/XML)
- REQ-3.3.3.6: Query OpenSearch documents by ID or search criteria
- REQ-3.3.3.7: Validate JSON event fields and structure

#### 3.3.4 Network Validation
- REQ-3.3.4.1: Packet capture via SSH chain
- REQ-3.3.4.2: TLS verification in captures
- REQ-3.3.4.3: Port connectivity tests (nc -vz)
- REQ-3.3.4.4: Ping tests for reachability
- REQ-3.3.4.5: Packet filtering and analysis

#### 3.3.5 Kubernetes Validation
- REQ-3.3.5.1: Pod status monitoring
- REQ-3.3.5.2: Container restart detection
- REQ-3.3.5.3: Resource state verification
- REQ-3.3.5.4: Event monitoring
- REQ-3.3.5.5: Namespace state comparison

### 3.4 Plugin System

#### 3.4.1 Core Plugins
- REQ-3.4.1.1: Shell script execution
- REQ-3.4.1.2: Python script execution
- REQ-3.4.1.3: SSH remote execution
- REQ-3.4.1.4: HTTP/HTTPS client operations
- REQ-3.4.1.5: Kubernetes API operations
- REQ-3.4.1.6: SQL database operations
- REQ-3.4.1.7: LDAP operations with TLS
- REQ-3.4.1.8: SFTP file transfers
- REQ-3.4.1.9: Network tools (ping, nc)
- REQ-3.4.1.10: File I/O operations (read/write/copy/move)
- REQ-3.4.1.11: XML processing with XPath support
- REQ-3.4.1.12: CSV parser plugin (read and parse to table)
- REQ-3.4.1.13: Excel parser plugin (XLSX/XLS support)
- REQ-3.4.1.14: Variable setter control
- REQ-3.4.1.15: OpenSearch operations (query, index, bulk)
- REQ-3.4.1.16: JSON parser plugin (read, parse, query with JSONPath)
- REQ-3.4.1.17: Log plugin (write messages to execution log with configurable level)
- REQ-3.4.1.18: Shell command execution
- REQ-3.4.1.19: The SQL block's drivers never read configuration from the WAVE host. An Oracle TNS alias, a directory (LDAP) address and a configuration provider address are refused (Host, Port and Service name, or a full connect descriptor, are used instead); an Oracle address or protocol given in Driver options must agree with the TLS row, and one that disagrees is refused before anything connects; Oracle sessions use the database's own time zone; every database that accepts a program name receives "WAVE"; SQLite runs on WAVE's own bundled SQLite, the same version on every WAVE host and remote host.

#### 3.4.2 Plugin Development
- REQ-3.4.2.1: Plugin creation interface
- REQ-3.4.2.2: Plugin parameter definition
- REQ-3.4.2.3: Plugin validation logic
- REQ-3.4.2.4: Plugin versioning
- REQ-3.4.2.5: Plugin documentation
- REQ-3.4.2.6: Plugin log level filtering
- REQ-3.4.2.7: Plugin runtime UI with console for logs
- REQ-3.4.2.8: Interactive plugin testing interface
- REQ-3.4.2.9: Optional log level override per plugin instance
- REQ-3.4.2.10: Every plugin declares how it relates to the Proxy setting: it uses the proxy, it does not use it, or it makes no web requests. A plugin that states nothing makes no web requests. Settings > Network > Proxy lists the blocks by this declaration.

#### 3.4.3 Plugin Features & Capabilities
- REQ-3.4.3.1: Store files in test artifacts directory
- REQ-3.4.3.2: Read test data files from configured paths
- REQ-3.4.3.3: Store parsed data as table variables
- REQ-3.4.3.4: Multiple endpoints for connectivity plugins
- REQ-3.4.3.5: Round-robin failover for endpoints
- REQ-3.4.3.6: Blacklist unreachable endpoints temporarily
- REQ-3.4.3.7: Configurable blacklist duration (default 60s)
- REQ-3.4.3.8: Set variables with configurable scope (block/test)
- REQ-3.4.3.9: The HTTP block's proxy choice is WAVE setting (the default: follows the Proxy setting) or Off (a direct connection). On a remote host the block uses that host's own proxy settings and the choice is not offered.
- REQ-3.4.3.10: The HTTP block's Auto content type for a file body or a multipart file part comes from WAVE's own file-type table, the same on every WAVE host; an explicit content type overrides it.
- REQ-3.4.3.11: A Shell Script, Shell Exec or Python block whose Working Directory is empty starts in the temp folder Python reports on the machine the block runs on.
- REQ-3.4.3.12: Every relative path a block takes starts at the home folder of the account the block runs as, or at the block's Working Directory where its design says so.

#### 3.4.4 Plugin UI Integration

##### 3.4.4.1 Plugin Visual Metadata (Built-in Plugins)
- REQ-3.4.4.1: Plugin definitions with icon name field for UI rendering
- REQ-3.4.4.2: Icon names from predefined set (Lucide icon library) via centralized icon registry

##### 3.4.4.2 Custom Plugin Discovery
- REQ-3.4.4.3: Custom plugin registration at application startup
- REQ-3.4.4.4: Custom plugins available via discovery API

### 3.5 Default Blocks

#### 3.5.0 Control System Foundation

##### 3.5.0.1 Control Entity Model
- REQ-3.5.0.1: Controls as distinct entity type separate from plugins and blocks
- REQ-3.5.0.2: Control data model with id, version, name, description, category
- REQ-3.5.0.3: Control parameter schema definition (inputs for configuration)
- REQ-3.5.0.4: Control metadata for core controls (id, name, category, iconName)

##### 3.5.0.2 Control Versioning
- REQ-3.5.0.5: Independent semantic versioning per control type
- REQ-3.5.0.6: Test/Block models support control references (controlId + controlVersion)
- REQ-3.5.0.7: Multiple control versions available simultaneously

##### 3.5.0.3 Control Structure Metadata
- REQ-3.5.0.8: Structure metadata for visual rendering (branching, looping, sequential)
- REQ-3.5.0.9: Branch definitions for container controls (if/else branches, parallel branches)

##### 3.5.0.4 Control Registration
- REQ-3.5.0.10: Control base class with metadata declaration
- REQ-3.5.0.11: Control registry for runtime version lookup
- REQ-3.5.0.12: Control registration at application startup

#### 3.5.1 Control Flow Blocks
- REQ-3.5.1.1: Conditional execution block (if/else)
- REQ-3.5.1.2: Break current block execution on condition
- REQ-3.5.1.3: Continue to next iteration on condition
- REQ-3.5.1.4: Halt block (stop test/suite/regression execution)
- REQ-3.5.1.5: Custom condition evaluation (expression-based)
- REQ-3.5.1.6: Visual conditional branching (if/elseif/else)
- REQ-3.5.1.7: Multiple elseif branches support
- REQ-3.5.1.8: Display branches as parallel flows in UI
- REQ-3.5.1.9: Highlight executed branch during runtime
- REQ-3.5.1.10: Merge point after conditional branches
- REQ-3.5.1.11: Assertion block with success/failure branches
- REQ-3.5.1.12: Success branch logs and continues
- REQ-3.5.1.13: Failure branch logs with a configurable outcome: warn and continue, or fail the current scope (a single-output assertion stops the flow; a split assertion continues on the failure branch)
- REQ-3.5.1.14: Halt scope configuration (test/suite/regression level)
- REQ-3.5.1.15: For loop with counter variable
- REQ-3.5.1.16: ForEach loop over collections
- REQ-3.5.1.17: While loop with condition
- REQ-3.5.1.18: Loop break and continue support
- REQ-3.5.1.19: Optional rate limiting in loops
- REQ-3.5.1.20: Switch/Case multi-branch decisions

#### 3.5.2 Kubernetes Management
- REQ-3.5.2.1: ConfigMap CRUD operations
- REQ-3.5.2.2: Secret CRUD operations
- REQ-3.5.2.3: Deployment management
- REQ-3.5.2.4: Namespace backup/restore
- REQ-3.5.2.5: Resource state comparison

#### 3.5.3 Helm Operations
- REQ-3.5.3.1: Chart installation
- REQ-3.5.3.2: Chart testing
- REQ-3.5.3.3: Chart upgrade
- REQ-3.5.3.4: Status monitoring
- REQ-3.5.3.5: Rollback capability
- REQ-3.5.3.6: Helm runs with a clean environment: the session's cluster settings, WAVE's own per-call folders and, when it is on, the proxy from Settings; nothing else from the WAVE host. Registry logins stored on the WAVE host are ignored; a private chart is pulled with the block's own Username and Password, OCI registries included.

#### 3.5.4 Timing and Synchronization
- REQ-3.5.4.1: Configurable delay blocks
- REQ-3.5.4.2: Optional jitter for delays
- REQ-3.5.4.3: Wait until condition with timeout
- REQ-3.5.4.4: Poll-based condition checking

#### 3.5.5 Data Management
- REQ-3.5.5.1: Data table support (CSV, JSON)
- REQ-3.5.5.2: Loop over data rows
- REQ-3.5.5.3: External file data sources
- REQ-3.5.5.4: Data-driven test iteration


### 3.6 Editor Capabilities

#### 3.6.1 Logstash Pipeline Editor
- REQ-3.6.1.1: Basic text editor with monospace font
- REQ-3.6.1.2: Syntax validation on save (via Logstash config test)
- REQ-3.6.1.3: Display validation errors below editor

#### 3.6.2 Configuration Editors
- REQ-3.6.2.1: YAML syntax validation
- REQ-3.6.2.2: JSON syntax validation
- REQ-3.6.2.3: JSON Schema validation for JSON files
- REQ-3.6.2.4: XML editor with XSD validation support
- REQ-3.6.2.5: XSD loading and referencing in XML editor
- REQ-3.6.2.6: Format conversion (JSON↔YAML)
- REQ-3.6.2.7: Diff view for comparing versions
- REQ-3.6.2.8: JSON formatter/beautifier (indent, sort keys)
- REQ-3.6.2.9: YAML formatter (proper indentation, alignment)
- REQ-3.6.2.10: XML formatter (indent, attribute formatting)

### 3.7 Reporting

#### 3.7.1 Artifacts
- REQ-3.7.1.1: JSON artifact generation
- REQ-3.7.1.3: Artifact compression
- REQ-3.7.1.4: Store artifacts in configurable temp folder

#### 3.7.2 Reports
- REQ-3.7.2.1: HTML report generation
- REQ-3.7.2.2: Pass/fail summary
- REQ-3.7.2.3: Execution timeline
- REQ-3.7.2.4: Error details with context
- REQ-3.7.2.5: Custom logging from tests/blocks
- REQ-3.7.2.6: Log levels (DEBUG, INFO, WARN, ERROR, FATAL)
- REQ-3.7.2.7: Merge custom and framework logs in reports
- REQ-3.7.2.8: Filter logs by level in report viewer
- REQ-3.7.2.9: Log level inheritance (test→suite→regression)

#### 3.7.3 User Attribution
- REQ-3.7.3.1: Extract user from git config
- REQ-3.7.3.2: Prompt for name if not available
- REQ-3.7.3.3: Store user in artifacts
- REQ-3.7.3.4: Display user in reports

## 4. Non-Functional Requirements

### 4.1 Performance
- REQ-4.1.1: UI response time < 200ms for interactions
- REQ-4.1.2: Support parallel operations (no artificial limit)
- REQ-4.1.3: Handle test suites with 1000+ tests
- REQ-4.1.4: Real-time updates via WebSocket
- REQ-4.1.5: Artifact loading < 5 seconds

### 4.2 Usability
- REQ-4.2.1: No-code test creation for basic users
- REQ-4.2.2: Code access for power users
- REQ-4.2.3: Intuitive drag-drop interface
- REQ-4.2.4: Contextual help system
- REQ-4.2.5: Keyboard shortcuts for common operations

### 4.3 Reliability
- REQ-4.3.1: Graceful error handling
- REQ-4.3.2: Test execution recovery
- REQ-4.3.3: Artifact persistence
- REQ-4.3.4: Connection retry logic
- REQ-4.3.5: Transaction support for state changes

### 4.4 Quality Assurance
- REQ-4.4.1: Unit tests for all components
- REQ-4.4.2: 100% code coverage requirement
- REQ-4.4.3: Coverage reports in HTML and JSON formats
- REQ-4.4.4: Integration tests for critical paths
- REQ-4.4.5: End-to-end tests for workflows
- REQ-4.4.6: Code quality metrics (linting, complexity)

### 4.5 Security
- REQ-4.5.1: Credential encryption at rest
- REQ-4.5.2: TLS for API communications
- REQ-4.5.3: Secret masking in logs
- REQ-4.5.4: Secure SSH key management
- REQ-4.5.5: K8s RBAC compliance
- REQ-4.5.6: Load credentials from K8s secrets
- REQ-4.5.7: Custom credential processing (decode/decrypt)
- REQ-4.5.8: Support custom binaries/scripts for decryption
- REQ-4.5.9: Credential loading at plugin/block level
- REQ-4.5.10: Credential loading at profile level
- REQ-4.5.11: Eager credential evaluation at test start
- REQ-4.5.12: Fail fast on credential loading errors
- REQ-4.5.13: Credential caching for efficiency
- REQ-4.5.14: WAVE keeps its own list of trusted SSH hosts in its data folder, and that list is the only source of host keys for every SSH connection. In Settings an admin views the list, adds hosts by reading their keys from the host or by pasting key lines, removes hosts, locks the list so no block can add to it, and sets the host key action stamped into newly placed SSH blocks, Disabled by default; these edits take effect only when the admin saves the page. An SSH block with no host key action uses Disabled. The list is shared work and is part of backup and restore; the lock and the default host key action are this WAVE's own settings, kept in its config file.
- REQ-4.5.15: An SSH connection authenticates only with the host, port, username, password, private key and passphrase its block names, and reaches jump hosts only through WAVE's session chain. The WAVE host account's SSH config file, default keys and SSH agent are never used.
- REQ-4.5.16: With certificate verification on, a block's CA certificate, when set, is the only certificate the block trusts. When it is blank, the block trusts the Additional Certificates in Settings plus the public authorities chosen there: WAVE's built-in list (the default) or the WAVE host's store. Any trusted certificate, root or intermediate, completes a chain. Environment variables of the WAVE host never change trust. The run log names the source used. In Settings an admin chooses the public authorities and adds or removes Additional Certificates; these edits take effect only when the admin saves the page. The choice of public authorities is this WAVE's own setting, kept in its config file; the Additional Certificates are shared work, kept in the data folder. A PostgreSQL connection with TLS off is a plain connection.
- REQ-4.5.17: Every secret WAVE stores is locked with one secret key that belongs to the WAVE instance, set in its config file or the WAVE_SECRET_KEY environment variable. WAVE ships no key: each installation makes its own with one documented command, and WAVE refuses to start without a key, with a message naming that command. The server never generates the key on its own and never writes it into the data folder, an export or a backup. In Settings an admin sees the key ID, a one-way fingerprint that tells keys apart without revealing them, and where the key comes from, and downloads the key as a key file with a short note on keeping and using it.
- REQ-4.5.18: In Settings an admin imports a key file to switch WAVE to that key, after a step that offers to export the current key and can be skipped only by confirming a copy is saved somewhere safe. The switch re-locks every stored secret, in the data folder and in the config file, with the imported key as one step that completes fully or not at all, even across a crash; a stored value the current key cannot open becomes null, shown before the admin confirms, so the block that uses it fails before sending anything and the secret shows as owed. The switch is refused while the key comes from the WAVE_SECRET_KEY environment variable, while a test runs and while a backup or restore is in progress.
- REQ-4.5.19: Every import (tests, suites, regressions, Functions, profiles, system variables, environments) and every backup restore treats secret values by one rule: a value without WAVE's locked-value marker is plain and is locked with the current key, except the masked placeholder a masked export leaves, which is treated like a value WAVE cannot open and becomes null, owed until a person enters the real value, though it never replaces a real secret already stored at the same place; a value locked with the current key is kept as it is; values locked with another key are opened with that key, provided by the person and checked against the archive's key ID when there is one, and locked with the current key, or else become null; either way the variable stays secret. WAVE never stores a secret value it cannot open: any save that would store one, from any client, is refused with the secret's name.
- REQ-4.5.20: A block Save that would turn an existing plain variable into a secret, through a Secret row in its outputs table or in a Variable Setter, first asks the person to confirm and names every such variable, one or many; canceling returns to the block unsaved.

### 4.6 Compatibility
- REQ-4.6.1: Linux host support
- REQ-4.6.2: Modern browser support (Chrome, Firefox, Edge)
- REQ-4.6.3: Kubernetes 1.19+ compatibility
- REQ-4.6.4: Python 3.12+ requirement (latest stable)
- REQ-4.6.5: Node.js 22 LTS requirement
- REQ-4.6.6: React 19 for frontend
- REQ-4.6.7: Git 2.0+ for attribution
- REQ-4.6.8: A test's outcome never depends on hidden state of the WAVE host: its account files, environment variables or system configuration. Anything WAVE still takes from the WAVE host is turned on by an admin in Settings, shown there, and named in the run log when it is used.
- REQ-4.6.9: WAVE reads and writes all text as UTF-8, storage included, whatever the WAVE host's locale; the build fails when code opens a text file without an explicit encoding. WAVE does not start on a host whose file-name encoding is not UTF-8, and its message names the setting that turns it on. Text WAVE sends over an SSH session is never altered: a script or command that cannot be written as UTF-8 fails its block before anything is sent.

### 4.7 Configuration
- REQ-4.7.1: Template library for editors (XML, JSON, YAML)
- REQ-4.7.2: Load and customize templates in editors
- REQ-4.7.3: Connection profiles (database, SSH, HTTP)
- REQ-4.7.4: Profile management UI
- REQ-4.7.5: Profile fields based on plugin requirements
- REQ-4.7.6: Plugin support for static inputs or profiles
- REQ-4.7.7: Kubernetes cluster profiles with namespace
- REQ-4.7.8: Application settings configuration
- REQ-4.7.9: Test artifacts directory configuration
- REQ-4.7.10: Environment-specific configurations
- REQ-4.7.11: Credential storage in profiles
- REQ-4.7.12: Profile reuse across tests
- REQ-4.7.13: The data folder is set in the config file (`storage.base_path`) when WAVE is installed; a relative value starts at the config file's folder. WAVE refuses to start when it is not set, names its absolute path in the startup log, and shows it read-only in Settings > WAVE Server.
- REQ-4.7.14: Settings > Network > Proxy turns on one proxy for the web requests (HTTP and HTTPS) WAVE makes from the WAVE host: either the WAVE host's own proxy variables, or a custom proxy given as an address and a port with an optional username and a sealed password. With the custom proxy an address list decides which requests use it, either every address except the listed ones or only the listed ones, each entry an exact address or a regular expression; local addresses connect directly unless a switch sends them through the proxy too. A second page, Proxy Blocks, lists which blocks use the proxy, which do not and which make no web requests. Other connections, and blocks running on a remote host, never use it. The run log says when a request went through the proxy. It is off by default.
- REQ-4.7.15: WAVE reads its settings from one config file: config/config.yaml in the folder WAVE is installed in, found from where WAVE's code lives and never from the folder WAVE was started from, or the file the WAVE_CONFIG_FILE environment variable names by its absolute path. WAVE refuses to start when that file is missing, cannot be read or holds an invalid setting, and its message names the file. The startup log names the file in use, and Settings > WAVE Server shows its path read-only. Environment variables still override the settings they name.
- REQ-4.7.16: Every Settings page that holds settings has Cancel and Save, both disabled until something changed, and a change applies only when the admin saves the page; Cancel restores the saved values. Save keeps only what the page shows: a setting hidden by another choice on the page, such as the fields of a feature that is switched off, goes back to its default, and a password stored for it is removed. Leaving a page with unsaved changes asks whether to stay, discard the changes or save them first. Save writes this WAVE's own settings into the config file: it changes only the saved settings, keeps the file's comments and layout, and keeps any edit made to the file by hand since WAVE started. When the config file cannot be written, Save is refused with the reason and nothing changes in the running WAVE.
- REQ-4.7.17: Shared work lives in the data folder: tests, suites, regressions, Functions, profiles, environments, system variables, test data, the trusted SSH hosts list and the Additional Certificates. This WAVE's own settings live in its config file: the log level, secret masking, the SSH lock and default host key action, the choice of public certificate authorities, the proxy, the MCP server and the periodic backup schedule, beside the data folder location, browser origins, secret key, recipe secrets folder and error detail. Browser preferences, such as the time zone, the theme and table layouts, stay in the browser.
- REQ-4.7.18: The log level is set in the config file, and the WAVE_LOG_LEVEL environment variable overrides it when WAVE starts. Settings > General shows the level in force. Save applies the new level at once and writes it to the config file; when WAVE_LOG_LEVEL is set, it still wins at the next start.
- REQ-4.7.19: The Time Zone in Settings > General is a preference of the browser: it changes only how that browser shows times, and the page says so. The WAVE server is not affected and records and sends all its times in UTC.
- REQ-4.7.20: The periodic backup schedule lives in the config file. Its save folder defaults to wave-backups in the home folder of the account WAVE runs under; a save folder inside the data folder is refused. Turning the schedule off and saving returns its other settings to their defaults.

## 5. Interface Requirements

### 5.1 External Systems
- REQ-5.1.1: Kubernetes API (via kubectl or client library)
- REQ-5.1.2: OpenSearch REST API
- REQ-5.1.3: LDAP protocol with StartTLS
- REQ-5.1.4: SQL databases via drivers
- REQ-5.1.5: SSH protocol for node access
- REQ-5.1.6: HTTP/HTTPS for service APIs

### 5.2 User Interfaces

#### 5.2.1 Web GUI

##### Foundation
- REQ-5.2.1.1: React 19 application with Vite build system
- REQ-5.2.1.2: Client-side routing with navigation history
- REQ-5.2.1.3: REST API client layer with error handling
- REQ-5.2.1.4: WebSocket client for real-time updates
- REQ-5.2.1.5: Zustand state management stores
- REQ-5.2.1.6: Reusable UI component library

##### Visual Design & Theming
- REQ-5.2.1.7: Modern design system with consistent styling
- REQ-5.2.1.8: Light and dark theme support with system preference detection
- REQ-5.2.1.9: Professional color palette with semantic colors
- REQ-5.2.1.10: Typography system with readable fonts
- REQ-5.2.1.11: Icon library integration
- REQ-5.2.1.12: Smooth animations and micro-interactions

##### Performance & Responsiveness
- REQ-5.2.1.13: Initial load time < 2 seconds
- REQ-5.2.1.14: Route transitions < 100ms
- REQ-5.2.1.15: Optimistic UI updates for snappy feel
- REQ-5.2.1.16: Client-side caching for frequently accessed data
- REQ-5.2.1.17: Responsive layout (desktop, tablet breakpoints)
- REQ-5.2.1.18: Virtualized lists for large datasets

##### UX Polish
- REQ-5.2.1.19: Toast notifications for user feedback
- REQ-5.2.1.20: Loading skeletons and spinners
- REQ-5.2.1.21: Empty state illustrations and guidance
- REQ-5.2.1.22: Error boundaries with recovery options
- REQ-5.2.1.23: Confirmation dialogs for destructive actions
- REQ-5.2.1.24: Undo/redo support for critical operations

##### Navigation & Discovery
- REQ-5.2.1.25: Breadcrumb navigation
- REQ-5.2.1.26: Global search with fuzzy matching
- REQ-5.2.1.27: Command palette (Cmd+K) for power users
- REQ-5.2.1.28: Recent items and favorites
- REQ-5.2.1.29: Sidebar navigation with collapsible sections
- REQ-5.2.1.50: Layout choices remembered for the browser session (a section, group or panel the user opens, closes, shows, hides or resizes comes back the same on return to the page and after a page load in the same tab; a new tab starts from the defaults)

##### Accessibility
- REQ-5.2.1.30: WCAG 2.1 AA compliance
- REQ-5.2.1.31: Full keyboard navigation
- REQ-5.2.1.32: Screen reader compatibility (ARIA labels)
- REQ-5.2.1.33: Focus management and visible focus indicators
- REQ-5.2.1.34: Sufficient color contrast ratios

##### Entity Management Screens
- REQ-5.2.1.35: Test management screen (list, create, edit, delete, duplicate tests)
- REQ-5.2.1.36: Suite management screen (list, create, edit, organize tests within suites)
- REQ-5.2.1.37: Regression management screen (list, create, edit, version control)
- REQ-5.2.1.38: Function library browser (search, filter, preview, import Functions)
- REQ-5.2.1.39: Plugin discovery screen (browse available plugins, filter by category, view documentation)
- REQ-5.2.1.40: Execution history screen (list executions, filter by status/date, compare runs, re-run)
- REQ-5.2.1.41: Artifact browser (list artifacts, preview content, download)
- REQ-5.2.1.42: Settings and preferences screen (user preferences, system configuration)
- REQ-5.2.1.43: Status Dashboard screen (system overview with entity counts, recent activity feed, running executions list, quick action shortcuts)

##### Frontend Plugin Integration
- REQ-5.2.1.44: Frontend plugin store fetching catalog from backend API
- REQ-5.2.1.46: Centralized icon registry mapping iconName to React components
- REQ-5.2.1.47: Plugin palette renders from API data (no hardcoded lists)

##### Frontend Control Integration (Hardcoded)
- REQ-5.2.1.48: Control palette renders from hardcoded control definitions (controls are fixed, not extensible)
- REQ-5.2.1.49: Control categories for palette grouping: conditional, loop, flow, timing

#### 5.2.2 CLI
- REQ-5.2.2.1: Command-line interface with structured commands

#### 5.2.3 REST API Foundation
- REQ-5.2.3.1: FastAPI application instance with ASGI server support
- REQ-5.2.3.2: Application configuration management (settings, environment variables)
- REQ-5.2.3.3: CORS middleware for cross-origin requests
- REQ-5.2.3.4: Global error handling middleware with standardized error responses
- REQ-5.2.3.5: Request/response logging middleware for audit trail
- REQ-5.2.3.6: OpenAPI/Swagger documentation auto-generation

#### 5.2.4 REST API Endpoints
- REQ-5.2.4.1: Plugin management and discovery endpoints
- REQ-5.2.4.2: Reusable block management endpoints
- REQ-5.2.4.3: Test workflow operations endpoints
- REQ-5.2.4.4: Test suite management endpoints
- REQ-5.2.4.5: Regression suite operations endpoints
- REQ-5.2.4.6: Execution control and monitoring endpoints
- REQ-5.2.4.7: Variable management endpoints
- REQ-5.2.4.8: Connection profiles endpoints
- REQ-5.2.4.9: Import and export operations endpoints
  - REQ-5.2.4.9.1: Every archive WAVE writes of its entities (a batch export of two or more entities, a test exported with its test data files, a backup) carries a details file with the WAVE version, the creation time, what it holds, its secrets handling and, when it holds locked values, the key ID of the key that locked them. A single-file export carries no details file; importing one tries the current key.
  - REQ-5.2.4.9.2: A backup holds what the admin selects, each a choice of its own: tests with their test data files, suites, regressions, Functions, environments, system variables, profiles, the trusted SSH hosts list, the Additional Certificates, run history, reports, artifacts and the config file. Restore applies the selected parts in dependency order; a report or artifact whose run is neither on this WAVE nor restored with it is skipped and named.
  - REQ-5.2.4.9.3: A backup can be password protected: the whole archive is encrypted with AES-256 and opens in WAVE or a zip tool such as 7-Zip with the password, while its file names stay visible. Settings shows the password's strength as it is typed, without ever blocking a password. A scheduled backup's password is stored in the config file, locked with the secret key. Restoring a protected backup asks for its password.
  - REQ-5.2.4.9.4: The config file is backed up and restored only as its own choices; restoring it is off by default. The backed-up config file never holds the secret key. Restoring it applies the preferences it holds (log level, secret masking, SSH and certificate settings, the MCP login and the backup schedule) and keeps this WAVE's paths, addresses, ports, MCP on/off state, proxy and secret key.
  - REQ-5.2.4.9.5: A restored item comes back as itself: it keeps its ID, name, description, tags, version and version history, and a Function keeps every version, so every link between restored items holds. An item this WAVE already has, by ID or by name, is a conflict, and the admin makes one choice for all conflicts: skip keeps the item here, overwrite gives the item here the backup's content, rename brings the backup's item in beside it, under a new name when its own is taken. Every restored item points at what its target became.
  - REQ-5.2.4.9.6: A restore never stops at an item it cannot restore: the item is left out and named with the reason, and everything else is restored. Before anything is written, the preview names every conflict and every item that cannot be restored, with the reason, for the parts and the conflict choice the admin picked; the result names every skipped item and every item that could not be restored.
- REQ-5.2.4.10: Resource discovery and metadata endpoints
- REQ-5.2.4.11: System health and status endpoints
- REQ-5.2.4.12: Configuration and artifacts endpoints
- REQ-5.2.4.13: Reporting endpoints

#### 5.2.5 WebSocket
- REQ-5.2.5.1: Real-time updates via WebSocket protocol
- REQ-5.2.5.2: Debug execution control endpoints
- REQ-5.2.5.3: Advanced API features (batch, clone, history, dependencies)

#### 5.2.6 Admin API
- REQ-5.2.6.1: Admin mode toggle (startup flag to allow, API endpoint to activate/deactivate)
- REQ-5.2.6.2: Admin mode status endpoint (enabled state, activation time, reason)
- REQ-5.2.6.3: State reset endpoint (clear all entities and executions)
- REQ-5.2.6.4: Bulk entity operations (add/modify/replace/delete with atomicity, cache invalidation, audit logging)
  - REQ-5.2.6.4.1: Bulk execution operations (identifier, entity type, source reference, status, progress, timestamps)
  - REQ-5.2.6.4.2: Bulk test operations (identifier, name, version, description, tags, block references)
  - REQ-5.2.6.4.3: Bulk suite operations (identifier, name, version, test references)
  - REQ-5.2.6.4.4: Bulk regression operations (identifier, name, version, suite references, scheduling)
  - REQ-5.2.6.4.5: Bulk environment operations
  - REQ-5.2.6.4.6: Bulk profile operations
  - REQ-5.2.6.4.7: Bulk block operations
  - REQ-5.2.6.4.8: Bulk artifact operations
  - REQ-5.2.6.4.9: Bulk report operations
  - REQ-5.2.6.4.10: Bulk configuration operations
  - REQ-5.2.6.4.11: Bulk user operations (future: multi-user authentication)
  - REQ-5.2.6.4.12: Bulk plugin operations (custom plugins)

#### 5.2.7 MCP Server
- REQ-5.2.7.1: Embedded MCP server, off by default, enabled from Settings > MCP Server persistently or until-restart, with configurable port, bind scope (local-only or network), and optional HTTP Basic authentication (username plus sealed, write-only password, with a strength score shown while a new password is typed that never blocks saving); every change on the page, enablement included, is staged and written on Save, and a Save that restarts a running endpoint or widens the bind scope to the network is confirmed first; the settings live in the config file
- REQ-5.2.7.2: Assistant acts as an ordinary client (same services, validation, and masking as the web UI; no privileged path; executions attributed to the assistant client)
- REQ-5.2.7.3: Recipe-based test authoring (structural control flow, catalog-driven block settings, inline local functions, published Function references by name and version; no geometry or wiring authored)
- REQ-5.2.7.4: Recipe compilation to canvas tests with generated wiring and deterministic automatic layout, stability-preserving on amendments
- REQ-5.2.7.5: Two-stage validation relayed in recipe terms (step, problem, proposed fix or stated blocker); invalid recipes are never persisted
- REQ-5.2.7.6: Recipe round-trip (any test retrievable as a recipe, with an explicit signal for inexpressible hand-built constructs)
- REQ-5.2.7.7: Self-serve recipe format guide and worked examples served over MCP
- REQ-5.2.7.8: Discover tools (block catalog, Function library contract cards with deep inspect, tests, suites, regressions, environments, profiles, system and global variables, executions)
- REQ-5.2.7.9: Author tools (tests via recipe, suites, regressions, profiles, system and global variables)
- REQ-5.2.7.10: Run tools (start and monitor test, suite, and regression executions)
- REQ-5.2.7.11: Report tools (structured failure report with outcome, failed blocks and errors, warning-and-above log excerpts, pending manual action items; generated report retrieval)
- REQ-5.2.7.12: Secret write modes per secret (provided value, server-side file reference, or declare-empty with manual action items in the authoring summary)
- REQ-5.2.7.13: Secret read policy mirrors system masking settings; reveal requires explicit per-retrieval user consent; the MCP never widens reveal policy
- REQ-5.2.7.14: MCP status visibility in System Config (running state, port, connected clients, recent activity) with per-session detail (client identity, address, connection and activity times) and administrative session termination
- REQ-5.2.7.15: Single-active-session workbench ownership (one lock holder mutates; all other viewers mirror live, read-only, with holder badge and take-over control)
- REQ-5.2.7.16: Take-over semantics (continues the visible draft; discard is always explicit; take-over of uncommitted changes requires explicit consent; human take-over pauses the assistant resumably)
- REQ-5.2.7.17: Watch-mode authoring (assistant builds through the live session step by step, creating and saving the test immediately; releases the session on completion or when human input is needed)

## 6. Data Requirements

### 6.1 Storage Structure
- REQ-6.1.1: YAML/JSON test definitions
- REQ-6.1.2: Versioned component files
- REQ-6.1.3: Metadata in all entities
- REQ-6.1.4: Artifact directory structure
- REQ-6.1.5: Configuration file hierarchy

### 6.2 Data Models
- REQ-6.2.1: Plugin schema with parameters
- REQ-6.2.2: Block schema with dependencies
- REQ-6.2.3: Test workflow schema
- REQ-6.2.4: Suite organization schema
- REQ-6.2.5: Regression collection schema

### 6.3 Version Management
- REQ-6.3.1: Semantic versioning for components
- REQ-6.3.2: Version pinning capability
- REQ-6.3.3: Dependency resolution
- REQ-6.3.4: Version compatibility checks

### 6.4 Data Validation & Sanitization
- REQ-6.4.1: Metadata field sanitization for security (test names, descriptions, tags, suite names, etc.)
  - Sanitize user input to prevent XSS attacks in UI
  - Sanitize for file system safety when names are used in paths
  - Sanitize for log injection prevention
  - No arbitrary character restrictions - sanitize on output, not input

## 7. Constraints and Assumptions

### 7.1 Constraints
- CON-7.1.1: No daemon sets in cluster
- CON-7.1.2: File-based storage initially
- CON-7.1.3: Single-user execution (no auth)
- CON-7.1.4: Linux host requirement
- CON-7.1.5: Network access requirements

### 7.2 Assumptions
- ASM-7.2.1: SSH access to nodes available
- ASM-7.2.2: K8s API credentials configured
- ASM-7.2.3: Git installed on host
- ASM-7.2.4: Python 3.12+ available (latest stable)
- ASM-7.2.5: Node.js 22 LTS for frontend build
- ASM-7.2.6: Modern browser for GUI
- ASM-7.2.7: GUI served by backend API (single process)

### 7.3 Future Considerations
- FUT-7.3.1: Database migration path
- FUT-7.3.2: Multi-user authentication
- FUT-7.3.3: Notification extensions (Teams, email, Slack)
- FUT-7.3.4: Cloud deployment options
- FUT-7.3.5: Advanced analytics
- FUT-7.3.6: AI-powered test generation
- FUT-7.3.7: Automated test execution in CI/CD
- FUT-7.3.8: Test failure blocks deployment
- FUT-7.3.9: Performance benchmarking and optimization
- FUT-7.3.10: Security vulnerability scanning
- FUT-7.3.11: State machine-based testing
- FUT-7.3.12: Event-driven test orchestration
- FUT-7.3.13: Message queue integration (Kafka, RabbitMQ)
- FUT-7.3.14: Test execution recording and replay
- FUT-7.3.15: Remote debugging capabilities
- FUT-7.3.16: Sub-test/nested test calling
- FUT-7.3.17: Advanced parallel patterns (fork/join)
- FUT-7.3.18: Container image execution environment for Shell Script and Python plugins
- FUT-7.3.19: Distributed execution across multiple nodes
- FUT-7.3.21: Execution time estimation based on historical averages
- FUT-7.3.22: Parallel test execution in suites
- FUT-7.3.23: Jenkins/GitLab CI integration
- FUT-7.3.24: Grafana dashboards integration
- FUT-7.3.25: Prometheus metrics integration

