### 15.15 Component Specification

Authoritative reference for shadcn/ui component mapping. Design decisions are in the Design Decisions document.

---

#### 15.15.1 Base Library

| Library | Purpose |
|---------|---------|
| shadcn/ui | Component primitives (MIT) |
| Radix UI | Accessibility foundation (MIT) |
| Tailwind CSS | Styling system (MIT) |
| Lucide Icons | Iconography (ISC) |

**shadcn/ui components**: sidebar, breadcrumb, switch, command, sonner, scroll-area, input, collapsible, accordion, resizable, popover, tabs, dropdown-menu, dialog, select, badge, table, button, checkbox, radio-group, toggle, card, kbd, textarea, pagination, skeleton, alert, alert-dialog, form, label, navigation-menu, progress

---

#### 15.15.2 Application Shell

> SHELL-1 through SHELL-6

| Element | Component | Customization |
|---------|-----------|---------------|
| Sidebar container | `sidebar` | 200px expanded, 56px collapsed |
| Nav links | `sidebar` built-in | Color-coded icons per SHELL-1 |
| Section headers | `sidebar` built-in | Uppercase 12px, 0.08em spacing |
| Collapse toggle | `sidebar` built-in | PanelLeftClose/PanelLeft icons |
| Theme toggle | `switch` | Sun/Moon icons, footer position |
| Breadcrumb | `breadcrumb` | Route-based |
| Toasts | `sonner` | Bottom-right |

The shell is never blocked by a page. When route content is made inert after a failed load step (15.11.3), the boundary falls between the shell and the page it hosts: the sidebar, its nav links and its theme and collapse controls stay live and focusable, so navigation out of a blocked page is always available.

---

#### 15.15.3 Canvas Panels

> CANVAS-9 through CANVAS-151

##### Block Palette (Left)

| Element | Component | Customization |
|---------|-----------|---------------|
| Container | `scroll-area` + `resizable` | 260px expanded, 64px collapsed |
| Search | `input` | Search icon prefix |
| Sections | `collapsible` | CONTROLS, PLUGINS |
| Categories | `accordion` | Network, Database, etc. |
| Collapsed (64px) | Custom icon strip | `popover` flyout on hover |

##### Variable Manager (Right)

> VMGR-1 through VMGR-14

| Element | Component | Customization |
|---------|-----------|---------------|
| Container | `resizable` | 320px default, 64px collapsed |
| Header | Custom | Uppercase title `VARIABLE MANAGER` (left) and `+ Add` button (right). No count badge. |
| Filter | `input` | Filters list by variable name |
| Sort | shadcn `select` | Three options: Name A-Z (default), Name Z-A, Type. Persisted in localStorage under `var-mgr.sort`. |
| Variable list | Custom | Flat list, no section grouping. Virtualized when > 50 rows. |
| Variable rows | Custom | Two-line grid: name (mono) + action cluster (eye, expand, edit, delete) on top; value (type-colored) + flag badges + type badge (rightmost) on bottom. `Unused` indicator surfaces when the variable has no references. |
| Row actions | `button` (icons) | eye, expand, edit (pencil), delete (trash); fixed left-to-right order; hover-visible on the row. |
| Create button | `button` | `+ Add` in the header opens the variable window in Add (Variable Window below). |

**System Variable Refresh**: On workbench mount (including navigation back from Settings), system variables are re-fetched to ensure namespace expressions resolve against current values without disturbing unsaved edits.

**Unused Indicator**: An amber `Unused` badge (VMGR-105) appears on a variable row when no block reads or writes its name - through a setting, an output binding, a condition row, a Variable Setter row, a Function Block field or a map / list entry - and no other variable's expression references it. The indicator reflects the union of workflow references and cross-variable expression references; both must be empty for a variable to read as Unused. The indicator updates whenever the workflow or any variable definition changes.

**Eye Toggle**: ON shows the runtime value (default), OFF shows the definition value italic + dim. For literals: runtime = current/last-known, definition = initial. For expressions: runtime = evaluated, definition = expression text. Disabled on secret rows (tooltip: secrets are always masked). For an expression with no runtime value, the runtime view shows muted `<not evaluated yet>`. Per-row state held in memory; resets only when the runtime values map is cleared (Reset signal). See 06-execution-monitoring.md § 15.6.2 for the full behavior spec.

**Edit / Delete Locking**: Edit and delete are disabled only while the run is actively running - enabled before a run, paused and after a run (VMGR-107) - and in the history (read-only) workbench and while mirroring. On a read-only definition (a Function opened in View, or a Shared Function drilled into) delete is disabled and Edit opens the window with only the Current value editable; the Initial view stays selectable and read-only. Eye toggle remains usable in both states.

##### Variable Window

> `src/features/workbench/common/variables/variable-inspect-modal/`

One window adds and edits a variable, a Function local, an input or an output (VMGR-106): `+ Add` opens it empty, titled "New ...", and the row pencil opens it filled, titled "Edit ...", with the value switch (VMGR-108). The inspect modal only shows values. The window supersedes the former Create Variable Dialog, Scalar Variable Edit Dialog, and Variable Rename Dialog (all removed); creating, editing, and renaming share one surface.

| Element | Component | Behavior |
|---------|-----------|----------|
| Container | `DraggableSheet` | Floating, draggable, corner-resizable; opens tall enough for its fields, up to the screen height, and scrolls; each Add opens a new window, cascaded like watchers, with no cap; an Edit window is one per variable, and the pencil focuses an open one |
| Title | Custom | 18px semibold: "New ..." in Add, "Edit ..." in Edit; inputs and outputs carry an `IN` / `OUT` header badge (VMGR-114) |
| Name | `NameField` | Python identifier validation, max 64 characters; on its own row; editable in Add and in Edit, in either value, with reference refactoring |
| Description | multiline input | Resizable; inputs and outputs only |
| Type | `select` | string / integer / float / boolean / array / map / table (an enum input: string / integer / float); a change in Edit asks to confirm and starts both values over as a new variable starts (VMGR-108) |
| Kind | `select` | Literal / Expression / Template (Template for strings only); drives which value field is shown; present for variables, locals and outputs, absent for inputs; hidden while Set to null is ticked; a change in Edit asks to confirm and starts both values over as a new variable starts (VMGR-108) |
| Required | `checkbox` | Inputs only: "Required (the Function Block must set a value)"; locked on while Create enum is on |
| Mark as secret | `checkbox` | Declares the variable or parameter secret; editable in Add and in Edit; in Edit, turning it on for a saved plain variable or off for a saved secret asks to confirm first (VMGR-109); on an input, off and locked while Create enum is on, and Create enum is off and locked while it is ticked (VMGR-104) |
| Set to null | `checkbox` | On its own row under Mark as secret, in Add and in Edit, for either value; ticked, one grayed `null` field replaces the Kind select and the value field; hidden while Kind is Expression or Template; off and locked on a Required input ("off while Required is set") and on an enum input ("off while enum is set"); ticked and locked, with a tooltip, on a secret input's default, as a secret input has no default (VMGR-111, VMGR-103) |
| Create enum | `checkbox` | Inputs only, the last checkbox: "Create enum (limit the Function Block to a fixed set of values)"; adds Allowed values right under it (VMGR-112) |
| Allowed values | value field | A list of the input's Type, written out as text with the Open editor button; no per-item type column |
| Value field | type-shaped field | Last; one same-height field per type (VMGR-110): a list, map or table written out as text and typeable, a long value cut with "…", an icon-only Open editor button on text, list, map and table values that opens the shared "Edit value" sheet holding the Visual / Code and format pickers, an expression box for an expression or a template, dots for a secret. Labeled "Value" with the value switch pill in Edit; an input's is "Default value" in Add (VMGR-113) |
| Footer | `button` | Add: Cancel, Create (disabled until valid); Edit: Cancel, Save, busy while writing |

**Name Validation**: must be a valid Python identifier `[a-zA-Z_][a-zA-Z0-9_]*`, not a Python keyword, and not a duplicate test-scoped name; inside a Function, not a name another input, output or local already uses, and an input or output name starts with a letter. The window also runs the design-time expression-cycle check.

**Parameter modes**: While drilled into a Function the `+ Add` menu offers Input parameter / Output parameter / Variable. Parameter modes share the variable window's layout and value field, add an `IN` / `OUT` header badge, and order their fields Name, a resizable Description, the Type row (outputs: Type and Kind), the checkboxes together (inputs: Required, Mark as secret, Set to null, Create enum; outputs: Mark as secret, Set to null), Allowed values right under Create enum when it is on, and the value last. An input's value is its default, literal-only: a Function Block that sets nothing uses it unless the input is Required; a Required input's default, if any, is used only when the Function runs on its own (VMGR-113); an enum input's default is required and is one of its allowed values (VMGR-112). An output's value offers the variable Kind choice and is returned as the fallback when nothing wrote the output, an expression output counting as written. The value is preset with the type's default; Set to null means a null default.

**Set to null**: Ticked, one grayed `null` field with no editor button takes the place of the Kind select and the value field, whatever the kind, and the window saves a plain null. Unticking brings back the kind and value from before the tick unless the null was saved. An emptied number box ticks it when it loses focus, where it is not locked. Where the null option is off (a Required or enum input, the Variable Setter's literal values, the suite / regression / system editors), an emptied number box blocks Save with that editor's existing invalid-number message and never means 0, except a Required input's default, where an emptied box means no default. A whole `null` typed as the text of a list, map or table Code view is refused in every editor with that view's existing message (`Must be a JSON array`, `Must be a JSON object`, and for a table `Table must be a list of rows`, or `Must be a JSON array of objects` in the suite, regression and system editors); `null` inside the text is a null item, map value or cell. In the window a wholly null value opens with Set to null ticked where it is offered; the other editors open it on the type's default.

**Default Values by Type**: string `""`, integer `0`, float `0.0`, boolean `false`, array `[]`, map `{}`, table `[]`. The window in Add and the suite / regression / system editors preset these.

**Type-Change Confirmation**: In Add, changing the Type away from a list, map or table that holds entries triggers a confirmation (reusing `ConfirmationDialog`): "Changing type to {newType} will clear N existing {oldType} {entry|entries|value}. This cannot be undone." Canceling leaves both type and value unchanged; confirming resets the value to the type-default for the new type. Any other type change resets the value to the new type's default without asking; nothing is converted. In Edit a type change or a Kind change asks to confirm and starts both values - Initial (an input's Default) and Current - over as a new variable starts: the new type's default, except a Required input's Default, which restarts with no default; an empty source for Expression or Template; Set to null unticked; a saved secret's stored value is cleared with them; the save check flags uses that no longer fit (VMGR-108). The Kind confirm reads "Change variable kind?" with "Changing kind to {newKind} will clear {the existing expression | the existing template | N existing {type} {entry|entries|value}}. This cannot be undone." and buttons Cancel / Change kind.

**Rename**: Renaming an existing variable, input, output or local happens in the window's Name field, in either value, with reference refactoring (see Execution Monitoring REQ-3.1.3.11 "Editing in the Window"); there is no separate rename dialog.

##### Console Panel (Bottom)

| Element | Component | Customization |
|---------|-----------|---------------|
| Container | `resizable` | 200px default |
| Header | Custom | Toggle, title, filter controls |
| Output | `scroll-area` | Auto-scroll |
| Filter | `dropdown-menu` | Log level |

---

#### 15.15.4 Canvas Toolbar

> CANVAS-74 through CANVAS-88, RUN-1 through RUN-15

| Element | Component | Customization |
|---------|-----------|---------------|
| Save/Undo/Redo | `button` | Primary/icon, disabled states |
| Environment | `popover` + `command` | Searchable (ENV-1 through ENV-16) |
| Run/Debug | `button` + `dropdown-menu` | Split button pattern |
| Stop | `button` | Destructive variant |
| Status | `badge` | Idle/Running/Paused/Passed/Failed |
| Debug controls | `button` | Step Over/Into, Continue icons |

---

#### 15.15.5 Modal Dialogs

> CANVAS-124 through CANVAS-151

##### Element Configuration

| Element | Component | Customization |
|---------|-----------|---------------|
| Container | `dialog` | 480px, draggable |
| Version | `select` | Dropdown |
| Tabs | `tabs` | INPUT, OUTPUT, CONFIG |
| Parameters | `table` | Name, Type, Required, Value |
| String/Expression fields | CodeMirror editor | Expression syntax highlighting, autocomplete. Multiline strings open the shared "Edit value" sheet |
| Integer/Float fields | `input` | Text input with regex masking (digits-only for integer, digits + one dot for float), supports negatives, no spinners. Default value shown as placeholder |
| Boolean fields | `checkbox` | Simple toggle |
| Enum fields | `select` + `toggle`, or `popover` + `command` | Dropdown with expression toggle; expression mode uses raw expression editor. Two opt-in variants per descriptor: grouped (options partitioned under headed sections) and searchable (a select-styled trigger opens a popover with a search box, filtering on label or short name without reordering; may also be grouped). Both variants mark the chosen option with the same check as the plain dropdown |
| Array / Map fields | `button` + sheet | Opens the shared "Edit value" sheet for structured editing |
| Autocomplete | `popover` + `command` | Variable suggestions via Ctrl+Space |

**Expression Mode**: All parameter fields support an expression toggle (`{}` button) that switches between literal input and expression mode. In expression mode, the field uses a CodeMirror-based editor with Python expression syntax highlighting and autocomplete for available variables and functions. The entire field content is a pure Python expression - no wrapper syntax needed. Multiline string parameters open the shared "Edit value" sheet in literal mode (format-aware highlighting, optional template markers). Enumeration fields (e.g., log level dropdown) support the same toggle to switch between fixed dropdown selection and expression mode.

**Map / Array Parameter Input**: Parameters of type `map` or `array` - canonical WAVE types used across the variable manager and profile schemas - render as a button that opens the shared "Edit value" sheet (the same type-aware bodies described under Variable Type Editors) for structured Visual editing plus a raw Code view, synchronized bidirectionally. Map values stay string-keyed string values, matching backend expectations for parameters like HTTP request headers and query parameters; the resulting value is stored as a dictionary (map) or list (array). The Visual and Code views stay available for every list and map field, per-item {x} included: Code shows an {x} item as `{{ expression }}` and reads it back. A field whose item type is fixed shows that type as a read-only pill.

**Conditional Parameter Visibility**: Parameters may declare visibility conditions based on other parameter values. The parameter table evaluates these conditions against current form state and hides or shows parameter rows accordingly. Hidden parameters have their values cleared. This enables context-sensitive forms - for example, showing a request body field only when the HTTP method supports a body.

##### Variable Type Editors

> REQ-3.1.3.1, REQ-3.1.3.11

Complex variable and parameter values (string, array, map, table) are edited through one shared family of type-aware value bodies, not per-surface dialogs. The same bodies render both read-only and editable, in a Visual view (structured, per-element / per-column typed inputs) and a Code view (raw JSON with syntax highlighting and linting), with bidirectional conversion.

The bodies are hosted by:
- the **Variable inspect modal** (workbench Variable Manager) for viewing a watched variable;
- the **variable window** (Add and Edit), whose one-line value field opens the "Edit value" sheet from its Open editor button;
- the shared **"Edit value" sheet**, opened from a row's Edit affordance or the variable window's Open editor button, for the variable window, the Variable Setter, profile parameters, element-config plugin parameters, and the entity-scope variable tables (Suite / Regression / System Variables).

**Common behavior**:
- **Dual view**: Visual / Code toggle with automatic conversion. The Code view never truncates; the Visual view caps very large collections with an overflow line that points to the Code view.
- **Typed elements**: array items and map values carry a per-element type, stored with the value and read back on reopen (REQ-3.1.3.19); table columns carry a per-column type, enforced per cell. An element's type changes only when it is picked: digits typed into a row never make it a number. A parameter map that feeds text-only systems (HTTP headers, query parameters, bind values) shows no type; a parameter map that feeds typed values (the SQL block's Driver options) shows the type picker beside each row's expression toggle.
- **Shared validation**: one validator family gates Save identically across both views (structure, per-cell type, empty / duplicate column names, expression cycles); invalid JSON or a structural error locks the Visual switch until it is fixed.
- **Exact numbers**: integer and float values are carried as exact text end to end, never an IEEE-754 double.
- **Rename with refactoring**: where the host surface owns the variable name, renaming refactors downstream references.
- **Read-only mode**: editing is disabled while a workflow is executing.

This shared surface replaces the former standalone array, map, and table editor modals.

##### Entity Picker Modal

> PICKER-1 through PICKER-5

Shared modal for searching, filtering, and selecting entities to add to a parent. Invoked by Suite Detail (add tests to a suite) and Regression Detail (add suites to a regression). Results are server-paginated and loaded incrementally as the user scrolls.

| Element | Component | Customization |
|---------|-----------|---------------|
| Container | `dialog` | Title, description, filter row, list, footer |
| Filters | `input` (search), `popover` + `command` (tags) | Debounced search, multi-select tags; both apply to every page request |
| Row | Custom `button` | Checkbox + name + status/tags badges; excluded rows disabled |
| List | Scroll container | Renders the first page; appends the next page when scrolled near the bottom |
| Incremental load | Floating badge | Spinner + "Loading…" anchored bottom-center of the list while the next page loads; overlays rows without displacing them |
| Failed next page | Floating `button` | Refresh icon + "Try Again" in the badge's place; the rows already loaded stay on screen and no further page is requested until it succeeds |
| More-available hint | Custom | Subtle line below the list while more remain: "{total} {entity} - refine your search to narrow down" |
| Error | `alert` + `button` | Alert icon, "Unable to load {what failed}", "Try Again"; reuses the shared list error pattern (LIST-5), replacing the list when the first page or the tag list fails |
| Footer | `button` | Cancel, Add Selected (disabled when none selected) |

The page size is fixed; changing the search or tag filter restarts paging from the first page. Selection persists across page loads and filter changes. What a failed read does depends on whether there is a list to keep. The reads that build the list - the first page and the tag list when the modal opens, the first page again after every search or tag change - replace the list with the error state whichever of them failed, and that one Try Again re-reads the first page and the tag list together. A next page requested by scrolling has a list behind it, so it keeps it: the loaded rows and the selection made from them stay, the loading badge is replaced in the same spot by a Try Again that re-requests only that page, and paging waits there until it succeeds. The modal is the surface its own reads serve, so a failure lands in the modal and never blurs the page behind it (LIST-18).

##### Quick Insert

> CANVAS-231 through CANVAS-237, CANVAS-241 through CANVAS-247

Keyboard-first insert surface over the canvas: a non-blocking movable window, not a modal. One search field drives one grouped list (PLUGINS / CONTROLS / FUNCTIONS) whose first row is highlighted on open and after every query change. The built-in rows are held in the browser and filter as the author types; the published-Function rows are server-paginated and appended as the list is scrolled. A permanent status line between the list and the hint strip describes the Functions half at all times.

| Element | Component | Customization |
|---------|-----------|---------------|
| Container | Draggable sheet | Fixed size, non-resizable; header bar is the drag handle and carries the title plus a close control; opens top-center over the canvas, dragged position kept for the session |
| Search | `command` input | Single field that keeps the caret wherever inside the window the author clicks; substring match on block name via the shared matcher the Tools Palette uses; debounced before it reaches the server, cleared undebounced |
| List | `command` list | Fixed height showing about ten rows; grouped headings; a group with no rows renders nothing; appends the next page when scrolled near the bottom |
| Row | `command` item | Icon + name + trailing meta: category for plugins and controls, version `badge` for published Functions, "empty Local Function" for the creator row; every row is a drag source onto the canvas |
| Functions status | Custom | Permanent single line of fixed height below the list; carries loading, count, refine, empty and failure wording (see the copy table in the design decisions); failure adds a warning icon and a `Try again` action |
| Footer | Custom + `kbd` | Key hints wrapping to two rows; the Enter hint names the anchor currently in force |

The Functions half requests a fixed page size, sends the active query with every page, sorts by name ascending, and restarts at the first page whenever the query changes, the window is reopened, or the Function library changes. A response for a superseded query is discarded rather than allowed to overwrite newer rows. A failed first page leaves no stale rows behind; a failed append keeps the rows already loaded. Retry re-issues exactly the request that failed. Confirm keys act on the highlighted row at all times and are never blocked or queued while a request is in flight.

Unlike the Entity Picker Modal, this surface draws no floating loading pill, no separate refine line and no error block inside the list; all three converge on the single status line, because the list is short and the hint strip sits flush beneath it.

---

#### 15.15.6 Entity Lists

> LIST-1 through LIST-26, BULK-1 through BULK-4

| Element | Component | Customization |
|---------|-----------|---------------|
| Table | `table` | Sortable, sticky header, hover |
| Pagination | `pagination` + `select` | Count, 10/25/50/100 |
| Row checkbox | `checkbox` | Selection column |
| Row actions | `dropdown-menu` | Edit, Delete, Duplicate, Run |
| Mass action | `button` + `dropdown-menu` | Disabled when 0 (BULK-4) |
| Loading | `skeleton` | 9 rows (LIST-4) |
| Empty | Custom + `button` | Icon, message, CTA (LIST-6) |
| Error | `alert` + `button` | Red icon, Try Again (LIST-5) |
| Blocked | `card` + `collapsible` + `button` + `tooltip` | Warning icon, alert role, Try Again, folded Details with a copy control; non-modal card rendered outside the inert route content, shell unaffected (LIST-10 through LIST-26) |
| Filters | `input`, `select`, `popover` + `command`, `dropdown-menu` | Search, Status, Tags, Columns |

Error and Blocked answer different failures. Error replaces the page when the records themselves could not be read; Blocked leaves the page rendered but unusable when any other read failed, which is the only honest state for a list whose rows arrived but whose filter menus or editing surfaces would open on data that never came. Only the records are primary: the types and tags a list filters by are supporting reads and block on failure like every other. The two states never appear together, because a failed records read wins outright and clears any block the same load would otherwise have raised; a card over the full-page error state would render that state's own Try Again inert underneath it and make recovery two steps instead of one. When several supporting reads fail in the same load, the Blocked card names all of them and its Try Again re-runs exactly that set. Both retries behave the same way: the Error state's Try Again and the card's each re-run every read of that page that failed, so leaving a failed load is one press and nothing on screen is a remembered reason (LIST-24).

##### Lightweight Dropdown Endpoints (FILTER-1)

Entity list filters that populate dropdowns from **other entities** (e.g., filtering tests by suite) use dedicated lightweight `GET /{entity}/names` endpoints rather than the full paginated list endpoint. These return only the fields needed for selection UI (`id`, `name`), sorted by name, with no pagination overhead.

Established pattern: `GET /environments/names` → `[{id, name, tags}]` for environment dropdowns. Same pattern applies to suites.

---

#### 15.15.7 Entity Forms

> FORM-1 through FORM-38

##### Create/Edit Modal

| Element | Component | Customization |
|---------|-----------|---------------|
| Container | `dialog` | 400px (FORM-5) |
| Form | `form` | react-hook-form + zod |
| Fields | `input`, `textarea`, `select`, `label` | Name, Description, Parent |
| Buttons | `button` | Cancel (secondary), Submit (primary), Delete (destructive, left, edit only) |

##### Delete Confirmation

| Element | Component | Customization |
|---------|-----------|---------------|
| Container | `alert-dialog` | Standard confirmation |
| Icon | AlertTriangle | Warning |
| Enhanced | Custom children list, `input` | Type name to confirm (FORM-25 through FORM-30) |

---

#### 15.15.7a Custom Form Components

> `src/components/ui/`

Custom reusable form components extending shadcn/ui primitives.

##### TagsInput

> `tags-input.tsx`

Multi-select tag input with inline tag creation. Used in Profile and Environment forms.

| Element | Component | Behavior |
|---------|-----------|----------|
| Container | `div` | Input wrapper with border |
| Selected tags | `badge` | Displayed inline with X remove button |
| Trigger | `button` | Opens dropdown/popover |
| Dropdown | `popover` + `command` | Search, filter, select existing tags |
| Create new | `command` item | "Create \"{search}\"" option when no match |
| Keyboard | native | Tab, Enter, Arrow keys navigation |

| Prop | Type | Purpose |
|------|------|---------|
| value | `string[]` | Selected tags |
| availableTags | `string[]` | Suggestions from existing data |
| onChange | `(tags: string[]) => void` | Selection callback |
| disabled | `boolean` | Disable all interactions |
| placeholder | `string` | Input placeholder |
| maxTags | `number` | Optional limit |

##### SecretInput

> `secret-input.tsx`

Password/secret field with visibility toggle and lock indicator. Used for sensitive profile configuration fields.

| Element | Component | Behavior |
|---------|-----------|----------|
| Container | `div` | Input wrapper |
| Lock icon | `Lock` (lucide) | Left side indicator |
| Input | `input` | `type="password"` by default |
| Toggle | `button` | Eye/EyeOff icons, toggles visibility |
| Masked | text | Shows `••••••••` when hidden |

| Prop | Type | Purpose |
|------|------|---------|
| value | `string` | Input value |
| onChange | `(value: string) => void` | Change callback |
| disabled | `boolean` | Disable input |
| error | `string` | Error message |
| placeholder | `string` | Input placeholder |

---

#### 15.15.8 Profiles

> PROFILE-1 through PROFILE-53 | REQ-4.7.4

##### Profiles List

| Element | Component | Customization |
|---------|-----------|---------------|
| Table | `table` | Name, Type, Tags, Usage, Actions columns |
| Type badge | `badge` | Profile type (database, ssh, http, etc.) |
| Tags | `badge` | Multiple tag badges per row |
| Usage | text | "N tests" count |
| Actions | `button` + `dropdown-menu` | Edit (pencil) + kebab (Duplicate, Delete) |
| Filters | `input`, `popover` + `command` | Search, Type dropdown, Tags multi-select |

##### Profile Create/Edit Modal

| Element | Component | Customization |
|---------|-----------|---------------|
| Container | `dialog` | ~460px width |
| Name/Description | `input` + `textarea` | Side-by-side layout |
| Type selector | `select` | Enabled on create, locked on edit |
| Tags | `TagsInput` (custom) | Multi-select with create-new |
| Config fields | Dynamic from schema | Type-specific form fields |
| Secret fields | `SecretInput` (custom) | Lock icon, masked, visibility toggle |
| Footer (create) | `button` | Cancel + Create |
| Footer (edit) | `button` | Delete (red, left, disabled if usage > 0), Cancel, Save |

##### Parameter Value Editing

Profile parameter values are edited through the shared type-aware value bodies - the same family used across the Variable Manager and the Variable Setter - so the profile form carries no private value editors.

| Type | Affordance |
|------|-----------|
| integer / float / boolean | Edited inline in the parameter row (typed input with validation; boolean as a true/false control) |
| string | Opens the shared "Edit value" surface (single-line and multi-line text with format-aware highlighting) |
| array / map / table | Opens the shared "Edit value" surface (typed Visual and raw Code views, per-column / per-element types, and validation that gates saving) |

The editor operates on the value for the currently selected environment; each environment keeps its own value. Profile values are literal only - a profile parameter has no expression or template editing, because a profile value is bound verbatim and has no run-time render step.

Secret parameters present a masked value with a clear-and-edit affordance: editing a protected value clears it before a new value is entered, and removing the secret flag from a parameter that holds a value clears that value. The secret flag itself is toggled by the per-parameter secret control in the row, not inside the value editor.

Table values are presented to the typed editor as named columns and rows and stored back in the profile's column / row form.

##### Environment-Specific Configuration (REQ-4.7.10)
| Element | Component | Purpose |
|---------|-----------|---------|
| Copy popup | `popover` | Copy values from another environment |
| Env selector | `select` | Per-environment configuration values |

---

#### 15.15.9 Dashboard

> DASH-1 through DASH-23

| Element | Component | Customization |
|---------|-----------|---------------|
| Page | `div` | `h-full overflow-hidden`, `px-6 pt-6 pb-4` |
| Metrics grid | CSS Grid | 4 cols |
| Metric card | `card` | Label, count (text-5xl), subtitle |
| Middle row | CSS Grid | `flex-1 min-h-[263px] auto-rows-fr` |
| Activity section | `section` + `card` | `h-0 flex-grow`, `divide-y overflow-y-auto` |
| Activity icons | Lucide | TestTube2 #4ADE80, FolderKanban #F59E0B, GitBranch #A78BFA, Play #60A5FA |
| Activity name | `span` | `text-card-title` (theme-aware) |
| Execution item | `article` | `py-3 flex flex-col gap-1`, not card |
| Status dot | `span` | `w-2 h-2 rounded-full`, `animate-pulse` for running |
| Status colors | CSS | info #22d3ee, success #4ade80, error #f87171, warning #fbbf24 |
| Progress bar | `progress` | Running/paused only, `h-1 pl-4` |
| Quick Actions | `section` | `flex-shrink-0` |
| Scrollbar | CSS | 8px, `--color-bg-muted` thumb |

---

#### 15.15.10 Settings

> SETTINGS-1 through SETTINGS-22, NET-1 through NET-11, BACKUP-1 through BACKUP-9

| Element | Component | Customization |
|---------|-----------|---------------|
| Layout | Custom CSS | Nav + content columns |
| Nav | `navigation-menu` | Vertical links (General, Environments, Variables, Network as a collapsible group of SSH Hosts / Certificates / Proxy, MCP Server, WAVE Server, Backup, About) |
| Fields | `form`, `input`, `select`, `switch` | Various settings |
| Environments | `table` + `button` + `dialog` | CRUD operations |

Each section reads its own data when the user opens it, and nothing a section needs is read while another section is showing. A failed read can therefore only block Settings over data the user is actually looking at (15.11.3).

##### Network Section

The Network pages share one set of parts; on SSH Hosts, Certificates and Proxy Setup every edit is staged until the page's Save (NET-1), and Proxy Blocks holds nothing to save (NET-10).

| Element | Component | Customization |
|---------|-----------|---------------|
| Page header | Section header + `button` | Title Case title, Cancel and Save; Save disabled until a change (NET-1) |
| Settings rows | Shared settings row group | Label, help line, control centered vertically (NET-11) |
| Tables | Shared data table | Search, sortable headers, pagination; Mass Action with Delete only on editable tables (NET-3, NET-7) |
| Public authorities | Option cards | WAVE's built-in list / This WAVE host's store (NET-5) |
| Read-only facts | Lock-marked read-only fact | WAVE Host's Store summary, the WAVE host's proxy values (NET-6, NET-10) |
| Add Hosts | Edit dialog + segmented control + plain-text editor | Detect from host / Paste lines, preview table with checkboxes (NET-4) |
| Add Certificate | Edit dialog + compact file drop zone | PEM text area, preview table with checkboxes (NET-8) |
| Certificate details | `dialog` + copy buttons | Read-only certificate fields and PEM (NET-9) |
| Proxy source | Segmented control | WAVE host's settings / Custom; the Custom password is the Settings password field shared with MCP Server: a saved password shows in the masked field every protected value uses, a click asks "Edit Protected Value?", "Delete & Edit" empties it, and a new password is typed hidden with a show / hide toggle (NET-10, MCP-7) |

##### System Variables Section

The System Variables tab provides CRUD management, export, and import for system-wide variables accessible to all tests via `system.var_name` namespace expressions. Variables are displayed in an editable table whose name / type / secret / delete columns are authored inline; the Value column uses the shared type-aware value editing surface (see Variable Type Editors) - integer / float / boolean inline, string and array / map / table via the "Edit value" sheet, and a masked clear-and-edit affordance for secrets.

| Element | Component | Customization |
|---------|-----------|---------------|
| Variable table | Custom table rows | Inline name / type / secret authoring; the Value column uses the shared value editing surface |
| Add variable | `button` | "+" adds empty row at bottom |
| Row actions | `button` (icons) | Delete individual variable |
| Save | `button` | Bulk save all changes |
| Export | `button` + `dropdown-menu` | JSON or YAML format, optional secret masking |
| Import | `button` + `dialog` | File upload with validation preview |

**Export**: JSON or YAML format, with optional secret masking ("***"). Masked exports are safe for sharing; unmasked for backup.

**Import**: Accepts JSON or YAML file with validation preview. Merges with existing variables (same-name overwrites). Atomic operation.

##### Backup Section

One page with three sections: Create Backup, Restore from Backup and Periodic Backup (SETTINGS-2).

| Element | Component | Customization |
|---------|-----------|---------------|
| Part selector | `checkbox` | BACKUP-1 |
| Format | `select` | YAML / JSON |
| Compression method | `select` | None (Stored) / Deflate / Bzip2 / LZMA |
| Compression level | `slider` | 1-9 |
| Secret values | `radio-group` | EXPORT-4 |
| Create button | `button` | "Create & Download" |
| Upload zone | `button` with the shared file drop behavior | Click or drop a .zip backup |
| Part summary | `table` | BACKUP-2, BACKUP-3, BACKUP-9 |
| Auto-repair toggle | `switch` | "Auto-repair fixable entities" |
| Conflict resolution | `radio-group` | Skip / Overwrite / Rename |
| Overwrite confirmation | Shared confirmation dialog | BACKUP-8 |
| Periodic Backup actions | `button` pair | Cancel and Save on the heading line (SETTINGS-21) |

---

#### 15.15.12 Suite Management

> REQ-5.2.1.36 | SUITE-21 through SUITE-25, LIST-1 through LIST-9, FORM-1 through FORM-19

**List**: Same as Entity Lists. Columns: Name, Tests count, Status, Last Run, Actions.

**Creation**: Modal dialog from list page (Name, Description, Tags). On save, navigate to suite detail view.

**Detail/Edit** (2-panel, same route with toggle): Collapsible left sidebar (~250px) | Tests list (flex)

Left sidebar contains:
- **Top section**: Name, Description, Tags (read-only in detail mode, editable in edit mode)
- **Bottom section**: Suite variables table (read-only in detail mode; in edit mode the name / type / secret columns are authored inline and the Value column uses the shared type-aware value editing surface - see Variable Type Editors)

Suite variables are accessed via `suite.var_name` namespace in test expressions (same pattern as profile references). They are NOT shown in the test's variable manager.

Tests list contains:
- Summary bar: pass/fail/never_run counts
- Table columns: #, Test Name, Status, Last Run, Actions
- Drag-and-drop reordering via grip handles (edit mode only)
- Run order: a suite runs its tests one at a time, in list order; tests have no per-test parallel flag

| Header Action | Behavior |
|---------------|----------|
| Edit/Save/Discard | Toggle between detail and edit mode |
| + Add Tests | Opens test picker modal (large, searchable, filterable, multi-select) |
| Run | Launches execution |
| Export | Suite export dialog |

| Test Action | Behavior |
|-------------|----------|
| Run | Single test execution |
| Edit | Opens Test Canvas |
| Remove | Removes test reference from this suite (non-destructive - test itself unaffected) |

**Test-Suite Model**: Many-to-many. Tests are independent entities referenced by suites like playlists. A test can belong to multiple suites. Removing a test from a suite only removes the reference.

**Tests List Page Changes**: Suite column shows count ("3 suites") with click-to-open popover listing suite names. "Move to Suite" bulk action replaced with "Add to Suite". New "Remove from Suite" bulk action.

**Import/Export**: Same pattern as tests - YAML/JSON format, ZIP for multiple, mask secrets option, conflict resolution on import (rename/overwrite/skip).

---

#### 15.15.13 Regression Management

> REQ-5.2.1.37 | REG-1 through REG-22, FORM-31 through FORM-38, LISTRUN-1 through LISTRUN-12

**List**: DataTable with columns: Name, Version, Suites, Tests (total), Status, Last Run, Tags, Created At, Updated At, Actions. No Mass Action. Server-side sorting with manualSorting. Column visibility toggle. Default visible: Name, Version, Suites, Tests, Status, Last Run.

**Detail** (2-panel): Collapsible metadata sidebar (left) + Suites DataTable (main). Same layout pattern as Suite Detail.

- Sidebar: "REGRESSION INFO" header, Suites count, Avg Duration, Created/Updated, Execution History, Changelog, Tags, Description, Variables editor
- Suites table: Name, Description, Tags, Tests (count), Avg Duration, Actions. Filter: Search, Tags, Columns.
- Edit mode: drag-to-reorder suites, add/remove via suite picker modal, save/cancel
- Suite row actions: Run/Edit (opens Suite Detail), Remove (non-destructive), Duplicate, Delete
- Header actions: Save/Undo/Redo | Add Suites, Create Suite, Run
- Variables: editable, regression scope only, collision validation against suite variables

**Forms**: Create modal (400px, Name/Description/Tags). Edit inline on detail page. Delete (simple confirmation only - warning: "Suites will not be deleted").

**Import/Export**: Same pattern as Suite import/export. Single export from detail page + row actions. Batch export from list. Import from list page with dry-run validation and conflict resolution.

**Suite Picker Modal**: Large searchable, filterable modal with multi-select checkboxes for adding suites to regression. Same pattern as test picker modal for suites.

**Expression Autocomplete**: `regression.` namespace in workbench expression fields. Same architecture as `suite.` namespace - namespace entries registered via variable helpers, recognized by expression completion source.

**Breadcrumb Context**: Detail page passes regressionId as query param when navigating to suite detail or workbench. Workbench breadcrumb builds hierarchical path: Regression Name > Suite Name > Test Name. Back button returns to originating detail page.

##### Shared Run Dialog

> `src/components/shared/run-dialog.tsx` | LISTRUN-1 through LISTRUN-12

| Entity | Dialog Size | Test Selection |
|--------|-------------|----------------|
| Test | 400x480 | None (single test) |
| Suite | 800x560 | Flat checkbox list |
| Regression | 800x560 | Grouped by suite (collapsible) |

| Element | Component | Customization |
|---------|-----------|---------------|
| Container | `dialog` | Resizable, max 90% viewport |
| Header | dialog built-in | "Run {Entity}", entity name subtitle |
| Test selection | `checkbox`, `collapsible`, `scroll-area` | Select All + counter, suite groups |
| Environment | `input`, `radio-group`, `scroll-area` | RECENT + ALL sections, $blue10 highlight |
| Footer | dialog built-in | Cancel + Run (disabled if no tests) |

---

#### 15.15.14 Executions List

> REQ-5.2.1.40 | LIST-8, EXEC-1 through EXEC-3

**Key differences**: No "New" button, no Mass Action, no checkboxes (read-only).

| Column | Content |
|--------|---------|
| Name | Entity name |
| Type | Test/Suite/Regression |
| Triggered By | User |
| Started | Date/time |
| Duration | Elapsed time |
| Status | Passed/Failed/Running/Stopped with color |
| Actions | View (eye), Stop (running only), menu |

**Empty**: Play icon, "No executions yet", "Go to Tests" CTA.

---

#### 15.15.15 Design Token Integration

All components use tokens from 15.9 Design System:

| Category | Pattern | Reference |
|----------|---------|-----------|
| Colors | `--color-{semantic}` | 15.9.2 |
| Typography | `text-{size}` | 15.9.3 |
| Spacing | `space-{scale}` | 15.9.4 |
| Border Radius | `rounded-{size}` | 15.9.5 |
| Shadows | `shadow-{level}` | 15.9.6 |
| Animation | `duration-{ms}`, `ease-{type}` | 15.9.7 |

---

#### 15.15.16 Requirements Coverage

| Requirement | Coverage |
|-------------|----------|
| REQ-5.2.1.1 | Visual workflow builder (React Flow + shadcn) |
| REQ-5.2.1.2 | Block palette (resizable, collapsible) |
| REQ-5.2.1.3 | Entity management forms |
| REQ-5.2.1.4 | Execution controls |
| REQ-5.2.1.5 | Variable manager panel |
| REQ-5.2.1.6 | Console output panel |
| REQ-5.2.1.7 | Design system integration |
| REQ-5.2.1.36 | Suite management |
| REQ-5.2.1.37 | Regression management |
| REQ-5.2.1.40 | Execution history |
