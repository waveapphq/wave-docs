### 4.13 Import & Export Endpoints

#### 4.13.1 Import/Export Operations

##### REQ-5.2.4.9 Import and export operations endpoints

Import and export endpoints enable portable serialization and deserialization of WAVE entities, supporting backup, version control integration, cross-environment migration, and workspace sharing. The system provides synchronous entity-specific export and import for granular control, and backup/restore (4.13.4) as the workspace-level bulk operation.

**Export Architecture**: Export operations serialize entities to YAML or JSON formats preserving complete definitions including metadata, configuration, and structural relationships. Each entity type has dedicated export endpoints under its own route prefix: a single-entity export returning a file download, and a batch export returning an individual file for one entity or a ZIP archive for two or more. All export responses carry appropriate content-type and content-disposition headers so browsers download with meaningful filenames. Dependencies are captured as reference identifiers rather than embedded content; multi-Function archives are ordered dependency-first so imports commit a dependency before its dependents.

**Import Architecture**: Import operations accept file uploads (YAML, JSON, or ZIP) containing serialized entity definitions, validating structure against schemas before persistence. Import follows a validate-then-apply two-step flow with per-entity conflict resolution (see 4.13.2).

**Entity Operations**: Exports and imports cover tests (workflow structures, variables, control flow), suites (test references with ordering), regressions (suite collections, profile associations), functions (definitions with version history), profiles (connection configurations with sensitive fields masked), environments (metadata), executions (export only - append-only records are not importable), and system variables. Each import validates type-specific constraints: test cycle detection, suite/regression reference existence, function dependency satisfaction, profile configuration schemas, and variable type declarations.

**Error Handling**: Invalid entity identifiers return not found with 404 status. Invalid or unsupported export formats return bad request with 400 status. Malformed files return bad request with parsing details. Schema violations return bad request with field-level errors. Import conflicts are surfaced through validation classification rather than hard failures, resolved per entity by the caller.

**Security Considerations**: Every export of secret-carrying content takes an explicit secrets handling choice with three modes. Mask, the default, replaces each secret value with the canonical placeholder, producing a file that is safe to share. Keep encrypted emits the sealed envelopes unchanged, producing a file whose secrets open on any WAVE holding the same secret key and remain unreadable everywhere else. Decrypt writes real values in cleartext for deliberate migration; it requires the encryption key, fails cleanly with a conflict error when the key is missing or wrong or a ciphertext fails its integrity check, and the interface warns visibly before download. The chosen handling covers every secret carrier in the document, including credentials inside historical entity versions. Executions are the exception: their variable snapshots are masked when written, so exported execution data is always masked and execution export offers no choice. Imports bring every secret value back to the at-rest guarantee by the one rule of REQ-4.5.19 and check for path traversal attacks during archive extraction.

#### 4.13.2 Synchronous Entity Import/Export

Tests, suites, regressions, functions, profiles, environments, executions, and system variables use synchronous endpoints under their respective route prefixes, returning results directly rather than using asynchronous job tracking. This approach fits the expected payload sizes and provides immediate feedback.

**Export Flow**: Export operations accept format selection (YAML or JSON) and, on secret-carrying entity types, the secrets handling mode (mask by default). Single-entity exports return a file download with appropriate content-type and content-disposition headers. Multi-entity exports return an individual file when exporting one entity, or a ZIP archive containing all selected entities when exporting two or more. Exported filenames are sanitized from entity names, with duplicate names disambiguated by appending numeric suffixes.

**Import Flow**: Import follows a two-step process. First, a validation endpoint accepts a file upload (YAML, JSON, or ZIP) and returns per-entity validation results without persisting anything. Each entity receives a status classification: ok (ready to import), conflict (name matches existing entity), fixable (has auto-fixable errors), or fatal (has blocking errors that prevent import). Validation also attaches per-entity warnings that never block import, rendered by the review interface as amber sublabels: masked secret values (placeholders left by a masked export) and secret values locked with a different key, each named. Second, the actual import endpoint accepts the same file along with conflict resolution instructions per entity (rename, overwrite, or skip). A value locked with a different key is stored as null (REQ-4.5.19). Entities with fixable errors are automatically repaired during import. Entities with fatal errors are rejected.

**Conflict Resolution**: When an imported entity name matches an existing entity, the user chooses per-entity resolution. Rename appends a numeric suffix starting with "(2)", incrementing until a unique name is found. Overwrite replaces the existing entity. Skip omits the entity from import. Default resolution when unspecified is rename. Entity types whose names act as reference targets restrict the offered resolutions: system variables support only overwrite and skip, because renaming a variable on import would silently break every expression that references it.

**Identity Handling**: Imported entities always receive new UUIDs and fresh timestamps, regardless of any identifiers present in the source file. This prevents ID collisions and ensures imported entities are treated as new. System variables are identified by name rather than UUID; importing them merges into the existing collection per the chosen resolution. Two operations keep identity instead: a Function import (3.5.6) and a backup restore (REQ-5.2.4.9.5).

**Shared Infrastructure**: All synchronous import/export services reuse the shared `determine_import_item_status` function for consistent status classification (fatal, conflict, fixable, ok) and the shared `parse_file`/`parse_zip_file` utilities for YAML, JSON, and ZIP parsing.

#### 4.13.3 Entity-Specific Import/Export Details

Each entity type extends the shared synchronous pattern with type-specific validation, export options, and import behavior.

**Tests**: Export supports the secrets handling choice. Import validation reuses the existing two-phase test validation system (fatal checks then fixable/warning checks) with context set to "import". Fixable errors are auto-corrected during import, with details of applied fixes included in the response. A test that has test data exports as an archive holding the document and the files, and the batch archive carries each test's files the same way. Import validation and import accept the archive as well as the bare document: the test data members are set aside before the documents are read, so a data file with a document-like extension is never taken for a test; the file entries are validated with the test data path and name rules, and a bad entry is fatal for that test; the preview reports the file count per test; and import writes the files under the imported test's identity, an overwritten test keeping only the archive's files.

**Environments**: No secret masking needed since environments do not contain secret fields. Validation checks for the required name field (missing name is fatal). Tags are normalized during import, with non-list values converted to an empty list as an auto-fix.

**Profiles**: Export supports the secrets handling choice, applied to credential fields across both the base configuration and the per-environment values. In exported profiles, environment_values keys (which are environment IDs internally) are replaced with environment names for human readability and portability. Import validation checks include: missing name (fatal), missing or invalid profile type (fixable - defaults to 'custom'), unresolvable environment names in environment_values (fixable - stripped with warning), and the secret-value warnings of 4.13.2. During import, environment names are resolved back to IDs.

**Suites**: Export supports the secrets handling choice for suite-level secret variables. Import validation checks include: missing name (fatal), unresolvable test IDs referenced in the suite's test list (fixable - stripped during import with warning), and the secret-value warnings of 4.13.2. An initial version entry (1.0.0) is created on import with "Imported suite" as the change description.

**Regressions**: Export supports the secrets handling choice for regression-level secret variables. Import validation checks include: missing name (fatal), unresolvable suite references (fixable - stripped during import with warning), and the secret-value warnings of 4.13.2. An initial version entry (1.0.0) is created on import.

**Functions**: Export serializes the complete definition together with its full version history, so an imported Function retains its published lineage, and supports the secrets handling choice applied to the definition and to every version in the history. Batch archives are ordered dependency-first so a Function is committed before any Function that depends on it. Detailed Function import/export design lives in the Function Management design (3.5.6).

**Executions**: Export only - executions are append-only records and are never imported. A single-execution export serializes the whole execution record or, via test selection, a per-test slice. A batch export accepts multiple execution identifiers and returns an individual file for one execution or a ZIP archive for two or more. Executions default to JSON format, reflecting their nature as runtime data rather than hand-edited definitions; YAML remains selectable. Execution variable snapshots are masked when they are written, so exported execution data is always masked and execution export takes no secrets handling choice.

**System Variables**: Export and import are server-side operations under the variables route prefix. Export serializes the stored system-variable collection to YAML or JSON with the secrets handling choice applied; the default masks secret values. Import follows the shared validate-then-apply flow with per-variable classification: ok, conflict (a variable of the same name exists), or fatal (missing type declaration), plus the secret-value warnings of 4.13.2. Conflict resolutions are restricted to overwrite and skip.

#### 4.13.4 Backup and Restore

Backup and restore operations provide disaster recovery and migration of a whole WAVE. Unlike entity-level import/export, which operates on one entity type at a time, a backup is one archive holding the parts an administrator selects, with a manifest. Settings has one Backup item, whose page holds three sections - Create Backup, Restore from Backup and Periodic Backup (REQ-5.2.1.42).

**Backup API**: A dedicated route prefix holds the operations: creating a backup (returns the archive as a download), keeping an uploaded archive for the restore in progress and dropping it again, validating an archive (returns the preview), restoring from an archive (applies it with conflict resolution) and reading and saving the periodic schedule.

**Periodic Backup**: A schedule creates backups automatically. Its configuration - the cron expression, evaluated in UTC (validated, with a human-readable translation), the parts to include, the format, compression, the secrets handling, the number of backups to keep (the oldest is deleted first) and the save folder - is stored in the installation's config file and changes only when the administrator saves the Periodic Backup section (REQ-4.7.20). A scheduled backup is written under a temporary name and renamed when complete, so the folder never shows a partial backup. The schedule state shows whether it is on, the last run's outcome and the next run time, which the page shows in UTC and in the browser's time zone.

**Reuse of Import/Export Infrastructure**: Backup reuses the per-entity serialization and validation of the entity-level import/export services, including the Function export document with its full version history and the dependency-first Function commit. Each part within the archive goes through the same classification (ok, conflict, fixable, fatal) established for entity imports, and every secret value restored follows the one import rule (REQ-4.5.19).

##### REQ-5.2.4.9.2 Backup contents and restore order

A backup holds what the administrator selects, each part a choice of its own:

| Group | Part | What the archive holds |
|---|---|---|
| Shared work | Tests | Each test document with its version history and the test's own test data files |
| Shared work | Suites, Regressions | Each document with its version history |
| Shared work | Functions | Each Function's export document with its full version history |
| Shared work | Environments, System Variables, Profiles | Each document; profile credentials and secret variables follow the secrets handling |
| Shared work | SSH Hosts, Additional Certificates | The trusted SSH hosts list (REQ-4.5.14) and the Additional Certificates (REQ-4.5.16) |

The administrator chooses the inner format (YAML or JSON), the compression (Stored, Deflate, Bzip2 or LZMA, with a level for the two that take one) and the secrets handling (mask, keep encrypted or decrypt, applied to every secret carrier in the archive).

Restore follows a validate-then-apply pattern. Uploading an archive shows a preview: the backup's details and, per part, the total, ok, conflict, fixable and fatal counts and, by name with the reason, every conflict and every fatal entry (REQ-5.2.4.9.6). The administrator selects the parts to restore - every part present is selected - chooses one conflict resolution (skip, overwrite or rename) and optionally auto-repair. Restore applies parts in dependency order: system variables, environments, profiles, Functions, tests, suites, regressions, SSH hosts and Additional Certificates. Fatal entries are always skipped, counted and named. Test data files are written under the restored test's identity, an overwritten test keeping only the archive's files.

##### REQ-5.2.4.9.5 Restored items keep their identity

A restore brings each item back as itself. Tests, suites, regressions, environments, profiles and Functions keep the identifier they had when the backup was made, with their name, description, tags, version, version history and timestamps, and a Function keeps every version. Items point at each other by identifier - a test at the Functions and profiles it uses, a suite at its tests, a regression at its suites, a profile at the environments it holds values for - so keeping identity is what keeps every link of a restored workspace intact. A backup restored onto an empty installation gives back the same workspace.

**Conflicts**: An entry conflicts when the installation already holds an item with the same identifier or, failing that, another item of the same type with the same name. Names are compared the way each type compares them when an item is created, and items written earlier in the same restore count as present. The administrator makes one choice for every conflict of the restore:

| Resolution | What is written | Identity | What later parts point at |
|---|---|---|---|
| Skip | Nothing | The item here is untouched | The item here |
| Overwrite | The backup's whole content over the item here: the item with the same identifier, else the one with the same name | The item here keeps its identifier | The overwritten item |
| Rename | The backup's item beside the one here, under its own name when that name is free, otherwise under the next free numbered name | Its own identifier when that is free, otherwise a new one | The renamed copy |

When the identifier and the name of an entry point at two different items here, the item with the same identifier is the target, and an overwrite that would give it a name another item holds is refused.

An overwritten Function merges versions as a Function import does (3.5.6), so versions held only by the installation survive. An overwritten test keeps only the archive's test data files.

**Links follow**: Parts are applied in dependency order, and each part records where every one of its entries landed. Every later part is rewritten through that record before it is checked and written: Function references anywhere in a test workflow or a Function body, including local Function bodies and edit-mode drafts; the profile bindings of blocks; the environments a profile holds values for; the tests of a suite; the suites of a regression and of its version history. A test's data files are written under the identity the test ended with.

**What may be written**: An entry is restored only when the rules of its own first save hold after the planned restore: a test passes its save validation, which includes that every Function it uses is present in the pinned version; a Function's references resolve and form no cycle; a regression's suites exist. Guards that protect an item against in-place edits do not apply to a restore, so a profile in use can be overwritten. A test that is running or paused is never replaced. Entries of one archive never overwrite each other. An entry that fails any of these is fatal (REQ-5.2.4.9.6).

**Parts without identifiers**: System variables are identified by name, trusted SSH hosts by host, port and key type, and Additional Certificates by fingerprint. They keep their own conflict rules, and the rename choice never renames them.

##### REQ-5.2.4.9.6 A restore never stops halfway

**One plan**: From the archive, the selected parts, the conflict resolution and the auto-repair choice the system computes one restore plan: for every entry whether it is new, a conflict, fixable or fatal, where it lands and why. The preview shows that plan and the restore applies the same plan, so what the administrator saw is what happens. The preview is computed again whenever a selection changes. A part that is not selected is still shown as it would restore, and selected parts never count on entries of a part that is not selected.

**One upload**: The administrator's browser sends the archive once. The server keeps it in private temporary storage of its own, outside the data folder, and answers with a reference; every later preview and the restore send that reference with the selections and never the archive again, so changing a selection costs no new upload. The kept archive is dropped when the administrator clears the file, picks another one or leaves the page, when it has not been used for thirty minutes, and when the server stops. When the server no longer knows a reference, the page sends the archive again on its own and repeats the request, and the administrator sees no error. A client that restores in one call sends the archive with the request itself, and nothing is kept.

**Named entries**: The preview lists, per part, every conflict and every fatal entry by name with its reason, fatal entries first. The result lists, per part, every skipped entry and every entry that could not be restored, by name with its reason. Each reason is one of a fixed set of plain statements about the entry: that it is already here, that its name is in use, that another entry of the backup or several items here share its name or identifier, that it has no name, that it has no type, that it holds no variables, that the name it would take belongs to another item here, that it uses something that is neither in the backup nor in the installation, that it is part of a circular Function reference, that the item here is running, that it fails its own save rules, that its host key cannot be read, that it is not a valid certificate, or that it could not be written. An archive member that cannot be read is a fatal entry of its part, named by its file name in the archive. On the page, the line of an entry that cannot be restored states that outcome before the reason.

**Isolation**: Each entry is applied on its own. An entry that fails, at its checks or while it is written, is reported as fatal and the restore continues with the next entry and the next part; the response always carries the result of every selected part. A new entry that fails leaves nothing behind.

**No transaction across entries**: A restore is not atomic across entries and has no undo: entries restored before a failing one stay restored.

##### REQ-5.2.4.9.3 Backup protection

##### REQ-5.2.4.9.4 Config file backup and restore

##### REQ-4.7.20 Periodic backup schedule

The periodic backup schedule is one of this WAVE's own settings and lives in its config file (REQ-4.7.17). A backup serves the case where one person works without a shared repository, so the schedule and the archives it writes stay with this WAVE. The save folder defaults to wave-backups in the home folder of the account WAVE runs under; a folder that starts with the home marker or is given as a relative path starts at that home folder, and a folder that is the data folder or inside it is refused, so backups never land in the shared work they protect.

The Periodic Backup section of Settings > Backup has Cancel and Save like every settings page (REQ-4.7.16). Turning the schedule off and saving discards every other value, because the page hides them (REQ-4.7.16), so turning it back on starts from the defaults; the record of the last run stays. After each scheduled run WAVE records the time and the result in the config file; when that record cannot be written, the backup itself still completes and the server log says the result was not recorded.
