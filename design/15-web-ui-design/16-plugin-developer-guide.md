### 15.16 Plugin Developer Guide

This guide explains how to add a new plugin to WAVE's visual workflow builder. The HTTP plugin serves as the canonical reference implementation.

#### Architecture Overview

WAVE uses a schema-driven plugin architecture. No plugin-specific UI code is needed - the generic parameter table and editor modals handle all parameter types automatically. A plugin developer defines data schemas in four frontend registry files and implements the backend execution logic.

#### Minimum Touchpoints Checklist

Adding a new plugin requires changes to these files:

**Frontend Definitions (4 touchpoints):**

| File | What to Add |
|------|-------------|
| `frontend/src/features/workbench/lib/plugin-definitions.ts` | One `def(...)` row: id, name, category, iconName, remotable (color derives from category) |
| `frontend/src/features/workbench/lib/node-parameters/plugins/<plugin-id>.ts` | New file exporting `inputs` / `outputs` (+ optional `groups`, `secretMaskFields`); register it in the maps in `node-parameters/index.ts` |
| `frontend/src/features/workbench/docs/plugins/<plugin-id>.md` | Version history frontmatter, changelog, details markdown (the first `versions:` entry IS the plugin's live version; `0.0.x` renders as a stub) |
| `frontend/src/features/workbench/lib/icon-registry.ts` | Import Lucide icon and add to `ICON_REGISTRY` map |

**Frontend Node Rendering (1 file):**

| File | What to Add |
|------|-------------|
| `frontend/src/features/workbench/common/canvas/workflow-node/icon-registry.ts` | Add the element type to all THREE maps: icon component, kebab icon name, color class |

**Backend Implementation (2 files):**

| File | What to Add |
|------|-------------|
| `backend/wavecore/plugins/my_plugin.py` | Plugin class inheriting `PluginBase` |
| `backend/wavecore/plugins/__init__.py` | Import and register the plugin |

#### Parameter Schema Reference

Each plugin defines its parameters using the `ControlParameter` interface:

```
ControlParameter {
  name: string           // Must match backend parameter name exactly
  label?: string         // Human-readable display label (falls back to name)
  type: string           // One of the 7 canonical types (see below)
  required: boolean      // Whether the parameter must have a value
  description?: string   // Tooltip text shown on info badge
  defaultValue?: unknown // Pre-populated value on first open
  enumValues?: string[]  // Dropdown select options (for string type)
  enumGroups?: object[]  // Partitions enumValues into headed sections in the menu
  enumSearchable?: boolean // Renders the enum as a searchable picker, not a plain select
  showWhen?: object      // Conditional visibility rule
  requiredWhenAll?: object[] // Required only while EVERY listed condition matches
  group?: string         // Collapsible section name
  multiline?: boolean    // Opens text editor modal instead of single-line input
  fileReference?: 'file' | 'file-or-folder' | 'folder' | 'save-as' // Adds a Browse picker over the test's files and the session's artifacts (save-as = destinations, Artifacts only)
  fileReferenceArtifactsOnly?: boolean // Hides the Test Data tab (sources that must be writable)
}
```

**Canonical Parameter Types (7 types):**

| Type | UI Rendering |
|------|-------------|
| `string` | Text input (or text editor modal if `multiline: true`). Add `enumValues` for dropdown select. |
| `integer` | Numeric input (no decimals) |
| `float` | Numeric input (allows decimals) |
| `boolean` | Checkbox |
| `map` | "Edit…" button opening MapEditorModal (key-value pairs) |
| `array` | "Edit…" button opening ArrayEditorModal |
| `table` | Tabular data (rows and columns with typed columns) |

**Conditional Visibility (`showWhen`):**

Parameters can be shown or hidden based on another parameter's value. Example from the HTTP plugin - the body field only appears for methods that support request bodies:

```
showWhen: { field: 'method', values: ['POST', 'PUT', 'PATCH', 'DELETE'] }
```

**File References (`fileReference`):**

A string parameter whose value names a file or folder declares `fileReference` in one of four modes, fixed per field: `file` (pick an existing file), `file-or-folder` (pick either), `folder` (pick a folder; files are shown muted), or `save-as` (name a destination: a folder plus a name, or an existing file to replace). The row gains a Browse button that opens the file picker over the test's Test Data and the session's Artifacts; the picker's Reference line keeps the scheme of the tab in front and accepts a typed path under it; any other path is typed in the field itself. Save-as rows never offer Test Data, because test data is read-only to blocks; a source field that must also be writable (a Move source) sets `fileReferenceArtifactsOnly` to drop the Test Data tab. Browse is hidden in expression mode and on read-only canvases. Inside a Remote Execution body it is hidden only for a block that runs on the remote host, which refuses `testdata://` and `artifact://` paths at run time; a block that runs on WAVE inside the body (the L badge), such as Shell Script or Shell Exec, keeps it, because its paths are WAVE paths.

**Ordered Lists (array parameters):**

An `array` parameter can be declared as an ordered list. It is edited as rows directly on the config sheet: each row a value field with its own expression toggle and a position label, the up/down arrow pair the branch rows use (up disabled on the first row, down on the last) and a delete button, with an add button under the list ("+ Add argument" on Shell Script). Row order is the value's order - the backend receives the list top to bottom - and an empty row is still an entry. Shell Script's Arguments is the reference use.

**At Least One of a Set of Rows:**

At least one of a set of rows is expressed on the last row of the set: that row is required only while every other row of the set is empty (and, optionally, while a further condition holds), and it carries the one message for the whole set, so the message shows once, under that row. Any number of the rows may be set. Shell Script's and Shell Exec's Pass pattern and Fail pattern are the reference: while Stop on pattern is ticked, an empty pair reports "Set a pass pattern, a fail pattern or both". The backend validates the same rule with the same message.

**Map Key Rules:**

A map parameter can constrain its keys with a name pattern and with prefixes a key may not start with, each carrying its own message, and a key is checked against the pattern first and against the prefixes only once it passes. Shell Script's and Shell Exec's Input Variables are the reference: they use the shell identifier rule and refuse the WAVE_ prefix, with "Invalid variable name: '{name}'" and "Variable names may not start with WAVE_: '{name}'".

**Parameter Groups (`group`):**

Parameters with the same `group` string are rendered in a collapsible section. The HTTP plugin uses three groups: "Timeouts & Retries", "Security", and "Connection". Ungrouped parameters appear at the top of the config modal.

#### Parameter Value Modes

Every input parameter supports three value modes: literal, expression, and template. The mode determines how the backend processes the parameter value before passing it to the plugin's execute method.

**Literal Mode (default)** - The parameter value is used as-is. No evaluation or substitution occurs.

**Expression Mode** - The entire parameter value is evaluated as a Python expression with safe builtins. The result must match the declared parameter type. In the config modal, each input parameter has a `{x}` toggle button that switches the field to an expression editor (CodeMirror) with variable autocomplete.

**Template Mode** - The parameter value is a string containing `{{expression}}` markers. Each marker is evaluated and replaced inline, producing a final string. Single braces `{}` are safe and will not trigger evaluation, making template mode compatible with JSON and YAML content. Template mode is available via a toggle in the TextEditorModal for multiline string parameters.

Expression and template modes are mutually exclusive per parameter. This constraint is enforced by the backend `BlockParameter` validator.

**Storage format** - Node configuration tracks which parameters use non-literal modes:

- `expressionFields: string[]` - parameter names in expression mode
- `templateFields: string[]` - parameter names in template mode

Parameters not listed in either array use literal mode.

#### Output Bindings

Output parameters appear in the Output Variables section of the config modal. Each output shows a combobox where users type a variable name or press Ctrl+Space for autocomplete suggestions. Type compatibility is enforced via `OUTPUT_TYPE_COMPAT` - for example, an `integer` output can only bind to an `integer` variable.

Output parameters use the `ControlOutput` interface:

```
ControlOutput {
  name: string        // Output field name (matches backend return key)
  label?: string      // Human-readable display label
  type: string        // Variable type (string, integer, float, boolean, map, array)
  description?: string // Tooltip text for info badge
  typeWhen?: object[]  // Type rules: the first rule whose every condition matches wins
}
```

An output whose shape depends on how the block is configured declares type rules instead of a second output. The declared `type` is the answer when no rule matches, and each rule pairs a set of input conditions with the type the output takes while they all hold - so a result that is a parsed map in one mode and raw text in another stays one output, and the binding combobox filters variables against the type the current configuration produces.

Some outputs are not fixed by the plugin: the author declares them per block, by name. Shell Script's variable exports are the reference - each name the author adds becomes an output of that block. Such an output is bound like any other output through the same combobox, takes its type from the variable it is bound to rather than from the schema (when the binding names a variable that does not exist yet, the author picks the type, String by default, and the variable is created with that type when the sheet is saved, holding null until the block runs), and can be marked secret with the Secret checkbox on its row, like every output, so its value is masked. An author-declared name may not reuse the name of one of the plugin's fixed outputs and may not be listed more than once. A declared output the run never produces is not a failure: the variable it is bound to keeps the value it had, and a DEBUG line records it.

#### Metadata and Documentation

Each plugin version includes three documentation elements:

**Changelog** - An array of concise strings, one per feature. Shown in the changelog dialog when users click the version badge. Be specific and action-oriented:
- Good: "Retry with exponential backoff (Retry Count, Retry Delay, Backoff Factor)"
- Bad: "Added retry support"

**Details** - A markdown string shown in the details dialog. Structure with these sections:
- Use Cases: Bullet list of what the plugin is for
- Behavior: How the plugin works at runtime
- Parameters: Every parameter with name, type, and default
- Outputs: Every output with name and description

**Parameter Descriptions** - Shown in info badge tooltips. Be specific and include examples:
- Good: "Full URL including scheme (e.g. https://api.example.com/v1/users)"
- Bad: "Request URL"

#### Plugin Categories

Following categories are available for organizing plugins in the toolbox palette:

| Category | Use For |
|----------|---------|
| `network` | HTTP, Ping, Netcat |
| `remote` | SSH Session, SSH Tunnel, SFTP, Remote Execution |
| `database` | SQL, OpenSearch |
| `system` | Shell Script, Shell Exec, Python, file I/O, Log |
| `kubernetes` | The K8s verb family: Connect, then one kind-agnostic block per kubectl verb (Get, Events, Logs, Top, Apply, Delete, Scale, Rollout, Exec, Script, Copy) |
| `helm` | The Helm verbs: Install, Upgrade, Rollback, Uninstall, Test, Status |
| `data` | CSV, Excel, XML, JSON |

#### Testing Checklist

Every new plugin needs tests in these files:

**Frontend Unit Tests:**

| Test File | What to Verify |
|-----------|---------------|
| `frontend/tests/features/workbench/lib/node-parameters/plugins/<plugin-id>.test.ts` | Registration in the parameter maps, exact field order, types, defaults, conditional visibility/requiredness, groups, outputs, palette entry, and BOTH icon registries |
| `frontend/tests/features/workbench/lib/plugin-definitions.test.ts` | Total and per-category counts, category order, remotable list |
| `frontend/tests/features/workbench/lib/node-parameters/secret-mask-fields.test.ts` | Declared credential sub-field catalogs stay in parity with the backend |

**Backend Tests:**

| Test File | What to Verify |
|-----------|---------------|
| `backend/tests/plugins/my_plugin/test_my_plugin.py` | Metadata declaration, parameter validation, execution, error handling |

**E2E / Playwright Visual Tests:**

| Test File | What to Verify |
|-----------|---------------|
| `tests/e2e/visual/canvas/v-plugin-*.spec.ts` | Drag to canvas, config modal interaction, parameter editing, output bindings |

#### Secret Handling

Secret protection is provided by the execution engine, not by each plugin. A plugin contributes declarations; the engine does the masking, sealing, registration, and reveal controls. This keeps every plugin - and every future plugin - correct by construction.

**A plugin declares, it does not mask.** Three declarations cover every case:

- **Credential input fields.** Mark an input parameter the plugin knows carries a credential (a password, private key, token, passphrase) as secret in its schema. The engine then seals a literal typed into that field at rest, masks it write-only in the form, shows the protection-status indicator, and honors the per-export secret choice - with no plugin code. An input still has no user-facing secret toggle; the plugin declares the field, and the user chooses the carrier (a protected literal or a secret variable reference).
- **Credential-bearing sub-fields.** When the plugin emits a map whose entries include known credentials (an HTTP client's sensitive headers), declare that field catalog. The engine masks those entries individually inside the otherwise-viewable container and drives the per-block "Mask specific fields" control from the catalog. The catalog is exact protocol or API knowledge - never inference from names, shapes, or entropy.
- **Secret outputs.** Mark an output the plugin inherently produces as a credential as secret in its output schema. The engine masks it and protects the variable it binds to.

**A plugin does not log its own inputs or outputs.** The engine emits one masked inputs line before execution and one masked outputs line after, identically for every plugin. A plugin printing its own parameter or output values would bypass the declarations and leak - output values only exist after execution, while a plugin logs during it. Reserve plugin logging for genuine diagnostics (connection progress, retries, transport configuration). A plugin that acquires a credential mid-execution (a token from a login, a server-issued cookie) registers it with the run's secret registry the moment it is acquired, which also protects credentials no declaration covers yet.

The result: a plugin gets runtime masking, encryption at rest, the reveal controls, and the leak-canary coverage by declaring its credentials - and inherits every future refinement of the masking model without changing.

#### Critical Rules

Lessons learned from the HTTP implementation:

1. **Parameter names must match backend exactly** - The frontend stores parameter values by name. If the frontend uses `retryCount` but the backend expects `retry_count`, execution will fail silently.

2. **No unit conversion** - Values pass through as-is. If timeout is in seconds in the config modal, the backend must also expect seconds. Do not convert between units.

3. **Release dates must use the current year (2026)** - Outdated dates break time-based filters and cause confusing test failures.

4. **All parameters need non-empty descriptions** - Info badges render empty tooltips if descriptions are missing. Be specific and include examples.

5. **100% test coverage required** - Every parameter, output, and code path must be covered by tests. No exceptions.

6. **No plugin-specific UI components** - The schema-driven architecture handles all parameter types. If you think you need a custom component, the parameter schema likely supports your use case already.

#### HTTP - Complete File Inventory

The HTTP plugin (id: `http-client`) touches these files across the codebase:

**Frontend Definitions:**
- `frontend/src/features/workbench/lib/plugin-definitions.ts` - Plugin registry entry
- `frontend/src/features/workbench/lib/node-parameters/plugins/http-client.ts` - Input parameters + outputs + groups + secret mask fields
- `frontend/src/features/workbench/docs/plugins/http-client.md` - Version history with changelog and details
- `frontend/src/features/workbench/lib/icon-registry.ts` - Globe icon mapping

**Frontend Components:**
- `frontend/src/features/workbench/common/canvas/workflow-node/icon-registry.ts` - Canvas icon registry entries (three maps)

**Backend:**
- `backend/wavecore/plugins/http_client.py` - Plugin implementation
- `backend/wavecore/plugins/__init__.py` - Plugin registration

**Frontend Unit Tests:**
- `frontend/tests/features/workbench/lib/plugin-definitions.test.ts`
- `frontend/tests/features/workbench/lib/node-parameters/plugins/http-schema.test.ts`
- `frontend/tests/features/workbench/lib/control-metadata.test.ts`
- `frontend/tests/features/workbench/lib/icon-registry.test.ts`
- `frontend/tests/features/workbench/common/canvas/workflow-node.test.tsx`
- `frontend/tests/features/workbench/common/canvas/workflow-node-actions.test.tsx`
- `frontend/tests/features/workbench/common/canvas/workflow-node-disabled-skip.test.tsx`
- `frontend/tests/features/workbench/common/canvas/workflow-node-selection.test.tsx`

**Backend Tests:**
- `backend/tests/plugins/http/test_http_plugin.py`

**E2E / Playwright Tests:**
- `tests/e2e/visual/canvas/v-plugin-http-01.spec.ts`
- `tests/e2e/visual/canvas/v-plugin-http-01-profiles.spec.ts`
- `tests/e2e/visual/canvas/v-plugin-http-02.spec.ts`
- `tests/e2e/visual/canvas/v-plugin-http-03.spec.ts`
- `tests/e2e/visual/canvas/v-plugin-http-03-editors.spec.ts`
- `tests/e2e/visual/canvas/v-plugin-http-03-validation.spec.ts`

**Design Documents:**
- Plugin Development Interface - Plugin configuration architecture
- Plugin Developer Guide - This guide
