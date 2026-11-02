### 4.11 Configuration & Artifacts Endpoints

#### 4.11.1 Configuration Management

##### REQ-5.2.4.12 Configuration and artifacts endpoints

Configuration and artifacts endpoints provide access to system configuration parameters and execution artifacts, enabling operational control over WAVE behavior and retrieval of test execution outputs. Configuration endpoints expose runtime settings while artifact endpoints provide access to execution results, logs, and generated files stored during test runs.

**Configuration Architecture**: The configuration system exposes a hierarchical key-value store containing runtime settings, feature flags, and operational parameters. Configuration retrieval returns the complete configuration tree or individual values by key path; these endpoints only read. The configuration model supports primitive types (strings, numbers, booleans) and marks the settings that take effect only at the next start.

**Configuration Retrieval**: The full configuration endpoint returns all accessible settings organized by category, excluding sensitive values like credentials which are masked or omitted. Individual key retrieval accepts dot-notation paths for nested values, returning the value along with type information and metadata indicating whether the setting is mutable at runtime. Missing keys return not found responses with suggestions for similar keys when available.

**Artifact Storage Model**: An artifact is any file inside an execution's artifact directory under the configurable artifact root - WAVE-generated outputs and plugin-written files alike. Artifact identity is path-shaped: the execution directory name combined with the artifact's relative path inside it, giving deterministic, human-readable addressing derived directly from the self-describing directory layout. There is no separate artifact metadata store; the filesystem is the source of truth. Identifiers are validated for containment within the artifact root before any filesystem access.

**Artifact Listing**: The artifact list endpoint scans execution artifact directories and returns paginated artifact metadata including identifier, execution reference, logical name, compression state, size, creation timestamp, and content type. Logical name and compression are derived from the artifact filename convention. Filtering supports the execution identifier; results are sorted by creation time, newest first, with page-based pagination. Temporary workbench session directories are excluded from this listing.

**Artifact Retrieval**: Individual artifact retrieval returns metadata including execution context, content type, size, and an integrity checksum computed on demand from the file content. The download endpoint streams artifact content with appropriate content-type headers for binary or text data. Large artifacts support range requests for partial retrieval. Missing artifacts return not found with the requested identifier for troubleshooting.

**Artifact Deletion**: Artifact removal frees the artifact's bytes and records a tombstone (see 5.3.3). Deletion requires explicit artifact identifier, with no bulk deletion through this endpoint to prevent accidental data loss. Successful deletion returns no content. Deletion of in-progress execution artifacts is rejected with conflict status. Missing artifacts return not found.

**Error Handling**: Invalid key paths return bad request with path parsing details. Artifact operations validate identifiers and access permissions. Missing resources return not found with context. File system errors during artifact operations return internal server error with masked details to prevent path disclosure.

**Security Considerations**: Sensitive configuration values are masked in responses. Artifact paths are validated to prevent directory traversal attacks. Downloaded content includes appropriate security headers to prevent browser-based attacks.

**Caching and Performance**: Configuration retrieval caches the full configuration tree. Individual key lookups use the cached tree avoiding repeated parsing. Artifact content is not cached at the API layer, relying on file system caching.

#### 4.11.2 Artifact Operations

Artifact-specific operations beyond basic CRUD support artifact integrity verification and lifecycle management.

**Artifact Integrity**: Retrieval operations compute content checksums on demand from the file itself, so integrity information always reflects the current bytes on disk and requires no stored bookkeeping.

**Content Disposition**: Artifact content retrieval supports caller-selected disposition. The default treats content as a download attachment; callers may request inline disposition so browser-viewable artifacts, such as generated HTML reports, render directly in the browser instead of downloading. Invalid disposition values are rejected with a validation error.

**Workbench Session Artifacts**: Session-scoped endpoints list and stream the artifacts of the current user's active workbench session, letting interactive runs inspect what an execution will produce. A caller with no active session gets an empty list, never a not found. Session artifact paths are validated for containment within the session's directory.

**Retention and Cleanup**: Execution artifact directories use timestamped naming that supports operational retention policies (external cleanup by age) without framework involvement. Temporary workbench session directories are cleaned automatically by the session lifecycle and a startup sweep. Manual cleanup is available through the deletion endpoints.

#### 4.11.3 Test Data Endpoints

Test data belongs to one test, so every test data endpoint lives under that test's own resource, and every path in a request or response is relative to the test's folder - a plain relative path with forward slashes, no leading slash, the empty path standing for the folder itself. There are no root identifiers and no separate test data resource: the test is the address.

**Listing**: The list endpoint returns a flat list of entries, each with its relative path, its kind (file or folder), its size and its modification time. Every folder is its own entry, so empty folders are visible; entries are sorted by path; dot-directories and symbolic links are skipped. A test that has no files yet answers an empty list, never an error. The client builds the tree and computes folder totals from the flat list; the response also carries the test's own file count and byte total, so the sheet's summary line is not summed by the client. Listing, content and zip retrieval are never blocked by a running execution.

**Upload**: One request carries one file, so a refusal never ends the others in a batch. It names a target folder (the root when none is given), a replace flag, the identifier of the row it fills, the session sending it, the size it declares and, on a retry, the notice it is answering. Every one of those is read before the first byte of the file, so the request is checked before anything is written: a file whose name already exists is a conflict unless replace is set, a folder of the same name is always a conflict, and the request is refused when storing the file would leave less than 1 GB free. A separate check answers, for a given name and size, whether the file would fit and whether the name is already taken, so a client can be told a transfer cannot succeed before any bytes cross the wire. The free space is measured again while the bytes arrive, so an upload that outgrows the disk is stopped instead of filling it. Each file is written beside its target and renamed into place, so a partially written file is never visible and a failure leaves the folder as it was. Written files carry no write permission. Changes to one folder do not interfere with each other: a file still being written is never locked out by another change to the same folder finishing first (TDATA-19).

**Upload State**: An endpoint under the test reports what is on its way in and what did not land, so a session arriving late sees the same rows as everybody else. One endpoint stops an upload in flight, refusing any caller that neither holds the test nor started it; one clears the notice a file left behind; one records a refusal the client never got to send. Naming the notice it answers is what lets a retry displace its own abandoned attempt instead of colliding with it. What is in flight is state of the test rather than of the session that started it, and the progress and the time remaining are computed once on the server, so every session watching reads the same figures. Half-written bytes are swept five minutes after nothing more arrives. Live changes to a test's files and to what is in flight reach every session watching that test over the existing event channel.

**Folder Creation, Rename and Delete**: Folder creation makes one folder whose parent already exists. Rename and move within the test share one operation that takes the entry's new relative path; the interface exposes rename. Delete removes a file, or a folder with everything inside it. Each change answers with the entry's path and reports a missing entry, a missing parent or a name collision by name.

**Content and Zip**: Content retrieval serves a file from disk with a caller-selected disposition (attachment by default, inline on request) and the same security headers the artifact endpoints send; the file is streamed, never read into memory. Zip retrieval packages the whole test data folder, or any folder inside it, into an archive named after the test or the folder.

**Containment and Names**: Every relative path is validated with the shared safe-path rule before any file system access: no leading slash, no empty segment, no `..`. Every created or renamed name is one path segment of 1 to 255 bytes with no separators, control characters or reserved names. Containment is checked on the real location, so an entry whose real path lies outside the test's folder is refused as an invalid path.

**Running-Test Refusal**: While any execution of the test is in progress - alone or as a child of a suite or regression, tracked or in the workbench - upload, folder creation, rename and delete are refused with a conflict status (409) and a message naming the test, carrying the execution identifiers involved in the same shape the test delete guard uses. An unknown test answers not found (404) on every endpoint.

**Lifecycle and Reset**: Deleting a test removes its test data with it and cloning copies it (see 4.6); export, import, backup and restore carry each test's files with the test document (see 4.13). The administrative reset offers a test data scope that empties the whole zone across every test. The MCP test data tools go through the same service as these endpoints, so they answer with the same messages and observe the same refusals.
