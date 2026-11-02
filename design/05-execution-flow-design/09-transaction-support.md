### 5.3 Reliability Design

#### 5.3.5 State Management

##### REQ-4.3.5 Transaction support for state changes

Transaction support covers how WAVE saves its system state including test definitions, suites, regressions, profiles, and configuration, and how it detects conflicting writes.

**File-Based Transactional Writes**: Functions, settings files and test data are written beside their target and renamed into place. Other entity documents are saved in place.

**Version Immutability and Conflict Detection**: Transactions enforce version immutability by verifying versions don't exist before creation, creating files or records, and failing on conflicts. This prevents overwrites and race conditions during concurrent version creation. Optimistic locking using version numbers or timestamps detects concurrent modifications, failing transactions when expected versions mismatch actual versions.

**Metadata Consistency**: A Function publish writes the version, then the Function index.

**Error Recovery**: Failures like validation errors return immediately. The layer logs failures with diagnostic context including operations and causes.

**Concurrent Access Coordination**: The layer detects concurrent modifications through version checks or timestamps, failing on conflicts rather than overwriting. Clear conflict messages enable manual merge or retry.

**Backup and Restore Consistency**: A restore is applied entry by entry: each entry is written whole or reported as not restored, and a restore that meets a failing entry goes on with the rest (REQ-5.2.4.9.6). One limit holds: a Test being overwritten whose data files fail part way is reported as not restored and can be left partly written.
