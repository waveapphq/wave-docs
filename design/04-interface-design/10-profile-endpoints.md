### 4.10 Connection Profile Endpoints

#### 4.10.1 Profile CRUD Operations

##### REQ-5.2.4.8 Connection profiles endpoints

Connection profile endpoints provide lifecycle operations for connection profiles enabling creation, modification, connectivity testing, type discovery, and schema retrieval. Profiles encapsulate connection configuration for external systems including databases, SSH servers, HTTP services, Kubernetes clusters, and OpenSearch instances, providing reusable connection definitions across tests and regressions.

**Lifecycle Management**: The system supports full CRUD operations for profiles. Profile creation establishes metadata including name, type, and type-specific configuration. Each profile type defines its own configuration schema covering connection parameters, authentication credentials, and behavioral options. Retrieval supports individual access by identifier and bulk listing. Deletion enforces usage checking to prevent orphaned references in connectivity plugins, returning conflict status when profiles are in active use.

**Type Discovery**: Type listing endpoints expose available profile types enabling dynamic client configuration. Type discovery returns the complete enumeration of supported types with descriptions. Schema retrieval for specific types returns JSON Schema definitions describing required and optional configuration fields, enabling client-side form generation and pre-submission validation. Each field carries its declared starting value where the schema declares one, so a client can open a new profile on the value the type is meant to start at rather than on a blank.

**Connectivity Testing**: Profile validation through connectivity testing verifies configuration correctness without requiring full execution context. Test endpoints attempt connection establishment using profile configuration, returning success status, connection latency measurements, and detailed error information on failure. Connectivity tests are non-destructive read-only operations that do not modify target systems. Test results include diagnostic information assisting configuration troubleshooting.

**Configuration Structure**: Profile configurations follow type-specific schemas while sharing common patterns. All profiles support optional description fields for documentation. Connection-oriented profiles name their target by host and port, or by one URL where the plugin takes one, plus a timeout where the type keeps one. Authentication-aware profiles support multiple credential types including passwords, API keys, certificates, and token-based schemes.

**Secret Handling**: Credential fields within profile configurations receive special treatment throughout the system. Passwords, API keys, private keys, and tokens are marked as secret fields in type schemas. Secret values are accepted during creation and update and are masked by default in API responses: retrieval operations return masked placeholders indicating secret presence without exposing values. The reveal flags in Settings > General are the deliberate exception.

**Failover Configuration**: The database profile names one endpoint and carries no failover list (SQL-14).

**Caching and Performance**: Profile metadata caches with invalidation on modification operations. Type schemas cache with extended TTLs as schema definitions change infrequently.

**Error Handling**: Operations enforce validation through consistent error responses. Invalid profile types return bad request with valid type enumeration. Missing profiles return not found with identifier context. Configuration validation failures return bad request with schema violation details including field paths and constraint descriptions. Connectivity test failures return success response with connected false and detailed error information. Deletion of in-use profiles returns conflict with referencing entity information.

##### REQ-4.7.3 Connection profiles (database, SSH, HTTP)

**Profile Types**: The system supports predefined profile types covering common integration targets:

- **Database**: Configures a relational database target under the SQL block's own names - host, port, database, username, password, the TLS trust material (verify_ssl, ca_cert, client_cert, client_key) and driver_options; the database type, the TLS toggle, the timeouts and the row and batch bounds are picked on the block, never stored on the profile
- **SSH**: Defines remote shell access with host, port, username, and key or password authentication; it is also how the SFTP block connects, since that block rides a session SSH Session Open opened with this profile
- **HTTP**: Establishes a base URL with default headers, credentials, TLS trust and client identity, and a request timeout
- **Kubernetes**: Configures cluster access with a service account token or client certificate (see REQ-4.7.7)
- **OpenSearch**: Configures a search cluster target under the OpenSearch block's own names - one cluster URL, username, password, API key, bearer token and the TLS trust material (verify_ssl, ca_cert, client_cert, client_key); there is no host and port pair, since the URL carries both, and the index pattern, the time range and the timeouts are picked on the block, never stored on the profile
- **Custom**: Provides an extensible escape hatch for systems not covered by predefined types, accepting arbitrary configuration objects validated at runtime

##### REQ-4.7.5 Profile fields based on plugin requirements

**Type Fidelity**: A profile type's fields mirror the plugin that type exists for, under that plugin's own names. They carry what describes the target - where it is, how it is trusted, and who WAVE is to it - and never per-request tuning, which belongs on the block that makes the request.

**No SFTP Type**: There is no `sftp` profile type. The SFTP block has no connection rows of its own: it names an SSH session, and the `ssh` profile bound on SSH Session Open carries the connection (SFTP-1).

##### REQ-4.7.7 Kubernetes cluster profiles with namespace

Kubernetes profiles carry exactly what the K8s Connect block consumes: the API server URL, a bearer token or a client certificate and key, the server CA certificate, a server certificate check that starts on, an override for the server name the certificate is matched against, and a default namespace that starts at "default". Token and client key are secret fields with the standard write-only treatment. The profile stores explicit fields, never the kubeconfig itself, so nothing at runtime parses kubeconfig files or reads them from the backend host. Multiple profiles target multiple clusters or namespaces; blocks receive the values through the standard profile binding and expression surface.

#### 4.10.2 Profile Listing and Filtering

##### REQ-4.7.4 Profile management UI support

Profile listing endpoints support pagination, sorting, and filtering to enable efficient UI presentation and navigation.

**Pagination**: List endpoints accept pagination parameters enabling efficient retrieval of large profile collections. The `page` parameter specifies the 1-based page number (default 1). The `per_page` parameter controls results per page (1-100, default 20). Response includes pagination metadata with total count, current page, items per page, and total pages enabling client-side pagination controls.

**Sorting**: The `sort_by` parameter accepts field names for result ordering (name, type). The `sort_order` parameter specifies direction (asc, desc, default asc). Sorting applies before pagination ensuring consistent result ordering across pages.

**Filtering**: Multiple filter parameters enable narrowing results:

- **search**: Case-insensitive text search matching profile names
- **type**: Filter by profile type (database, ssh, http, etc.)
- **tags**: Filter by tags using AND logic (profile must have all specified tags)

Filter parameters combine with AND logic. Empty or omitted filters return all matching profiles.

**Tags Support**: Profiles support categorization through tags - an optional list of string labels. Tags enable organization and filtering of profiles (e.g., "production", "staging", "database", "critical"). The `/profiles/tags` endpoint returns all unique tags across all profiles, sorted alphabetically, enabling tag autocomplete and filter suggestions in the UI.

Tag validation enforces:
- Tags must be strings
- Empty strings and whitespace-only tags are rejected
- Leading/trailing whitespace is trimmed
- Duplicate tags are preserved as provided

**Profile Duplication**: The duplicate endpoint creates a copy of an existing profile with a new unique name. Duplication resets usage count to zero, enabling full edit freedom on the copy. The default copied name appends a counter suffix (e.g., "Profile Name (Copy)"). Custom names can be provided in the request body.
