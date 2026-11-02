### 15.8 User Experience Design

#### 15.8.1 Usability Features

##### REQ-4.2.1 No-code test creation for basic users

##### REQ-4.2.2 Code access for power users

Power users work with test assets as code through two complementary surfaces: the entity import/export round-trip and in-place expression editing.

**Import/export round-trip**: Every persistable entity - tests, suites, regressions, profiles, environments, and library Functions - exports to YAML or JSON, individually or in bulk (bulk exports produce a ZIP archive). The export dialog offers a format picker and an optional secrets-masking toggle that replaces sensitive values with masked placeholders. The exported documents are the full entity definitions: a user can edit a test's structure, parameters, variables, and wiring in any text editor and re-import the result. Import is a two-step flow - upload and validate first (structural errors and warnings are reported per item before anything persists), then commit with an explicit conflict-resolution strategy per colliding name (auto-rename, overwrite, or skip). The round-trip property holds in both directions: an unmodified export re-imports cleanly, and a hand-edited export is held to the same validation rules the visual builder enforces.

**Expression editing**: Parameter fields accept Python expressions wherever the expression toggle (`{}`) is present. Each expression field offers an inline editor with variable autocomplete and a full editor modal providing a micro-IDE experience: Python syntax highlighting, live design-time validation with inline error and warning decorations, type-aware completion for workflow variables and Python builtins, and single- or multi-line modes. Variable references inside expressions are validated at design time against the variable scope visible to the element. Computed variables use the same expression engine with reference-cycle detection.

##### REQ-4.2.3 Intuitive drag-drop interface

##### REQ-4.2.4 Contextual help system

Help is contextual and version-aware: documentation surfaces next to the element it describes, scoped to the version in use, without leaving the configuration flow.

**Per-element documentation**: Every control and plugin ships with markdown documentation bundled into the application - a summary description, per-version detail pages, and a per-version changelog. The element configuration surface exposes this through two dialogs: **Details** renders the documentation for the currently selected version as rich text (with an explicit fallback line when a version carries no documentation), and **Changelog** renders the version history as a navigable list. Because documentation is versioned alongside the element's parameter schema, a user configuring an older version reads the help that matches that version's behavior. Published library Functions follow the same pattern: their per-version details and changelog, authored at publish time, render through the same Details and Changelog dialogs.

**In-context parameter help**: Parameter tables carry an info affordance per row - hovering the `(i)` icon shows the parameter's description, type expectations, and requirements where declared. This applies to plugin and control parameters as well as declared Function inputs and outputs.

**Shortcut discoverability**: The canvas help popover (see REQ-4.2.5) doubles as the contextual reference for keyboard interactions, grouped by category with platform-aware modifier labels.

##### REQ-4.2.5 Keyboard shortcuts for common operations

The application provides keyboard shortcuts for common operations across the canvas workflow builder. Shortcuts follow platform conventions - Ctrl on Windows/Linux, Cmd (⌘) on macOS - and are suppressed when focus is inside text input fields or modal dialogs.

Canvas editing shortcuts cover clipboard operations (copy, cut, paste), element duplication, selection management (select all, box select via Shift+drag), deletion, disconnection, and undo/redo history navigation. All shortcuts map to existing store actions that are also accessible through context menus and toolbar buttons, ensuring feature parity between keyboard and mouse workflows.

Keyboard-driven element insertion is provided by the canvas Quick Insert window (Ctrl+Space): a catalog of every insertable item - plugins, controls, library Functions, and the New Function creator - searched by substring and opening top-center over the canvas. Two insert keys give the keyboard its range: Enter inserts where the flow is, anchored to the single selected block, otherwise the last block the user interacted with, otherwise the end of the flow; Ctrl+Enter inserts at the viewport center, unwired. Result rows are also drag sources, and the placed element always becomes the selection so consecutive insertions chain naturally. The Quick Insert surface and its interaction rules are specified in section 15.4 of the visual workflow builder design.

Discoverability is supported through the Interactions Hint Popover, opened by a circle button with a `?` icon at the bottom right of the canvas, which lists all available shortcuts and mouse interactions grouped by section. Modifier key labels adapt to the user's platform.

Detailed shortcut registry and interaction behavior are documented in section 15.4.1.6.

#### 15.8.2 Settings and Preferences

##### REQ-5.2.1.42 Settings and preferences screen

The Settings screen provides centralized access to user preferences and system configuration through a two-column layout with tabbed navigation. The left column presents section navigation while the right column displays the content for the selected section. This design pattern supports extensibility as new configuration categories are added over time.

**Layout Architecture**: The page divides into a fixed-width navigation column approximately 240 pixels wide and a flexible content area consuming remaining horizontal space. A subtle vertical divider separates the two regions. The "Settings" title appears above the navigation list, establishing the page context. Section items display as vertical navigation links with the active section highlighted using the primary accent color background with contrasting text.

**Section Organization**: Eight sections organize related settings logically, in this order in the navigation; Network is a collapsible group: it holds SSH Hosts, Certificates and Proxy, which is itself a group of two pages (SETTINGS-2, NET-10):

| Section | Purpose | Content |
|---------|---------|---------|
| General | System preferences | Log Level; Time Zone, a preference of this browser; Secret Masking. Cancel and Save on the Logging heading line |
| Environments | Environment definitions | Environment management table with CRUD operations |
| Variables | System-wide variables | Variables table with inline authoring, export and import |
| Network | How WAVE trusts and reaches other systems | SSH Hosts: Host Key Policy and the Trusted SSH Hosts table; Certificates: the Public Certificate Authorities source, the Built-in Authorities table or the WAVE Host's Store summary, and the Additional Certificates table; Proxy Setup: Use a proxy, its source, the custom proxy and its address list; Proxy Blocks: which blocks use the proxy |
| MCP Server | The assistant endpoint | Status, enablement, endpoint configuration with a password strength score, sessions, recent activity; Cancel and Save on the MCP Server heading line (MCP-1 to MCP-9) |
| WAVE Server | The running servers and their browser sessions | Read-only Frontend and Backend Server cards, the Backend Server card carrying the Data folder and Config file rows; the sessions roster (WSRV-1 to WSRV-11) |
| Backup | Backup and restore of a whole WAVE | One page with three parts. Create Backup: the parts to include, format, compression and secrets handling; Restore from Backup: upload, the preview, and restore; Periodic Backup: the periodic schedule, whose Cancel and Save sit on its heading line (SETTINGS-21) |
| About | System information | Application version, build information, support links |

**Environments Section**: This section enables management of named environments that categorize connection profiles for different deployment targets. The environment list displays as a paginated table with columns for Name, Description, Tags, Usage (profile count), and Actions.

The "New Environment" button in the section header opens a creation dialog. Each environment row provides edit and delete actions through an actions column. The edit action opens a modal form for modifying name, description, and tags. Tags enable categorization and filtering of environments (e.g., "production", "staging", "development"). The delete action asks for confirmation. When profiles use the environment, the confirmation says how many, warns that their per-environment values are removed with it, and asks the user to type the environment's name.

Usage calculation reflects the count of profiles referencing each environment, enabling informed decisions about environment consolidation or cleanup. The subtitle "Manage environment names used across profiles" clarifies the purpose of this section for users.

**General Section**: Holds this WAVE's logging and secret masking settings and this browser's time zone. Like every settings page it has Cancel and Save, on the header of its first section: every change waits for Save, and both buttons are disabled until something changed (REQ-4.7.16). Save writes the log level first, then the secret masking controls, then the time zone into the browser. When a write fails, the ones before it stay saved and the page keeps the changes not yet written.

**Log Level Control**: The first row of the General section sets how much the WAVE server writes to its own log; block output in runs is not affected (REQ-4.7.18).

| Element | Description |
|---------|-------------|
| Section Header | "Logging", carrying the page's Cancel and Save |
| Row | "Log Level" with a select of DEBUG, INFO, WARN, ERROR, FATAL |
| Subtitle | One line under the Logging heading: "Verbosity of the WAVE server's own log. Block output in runs is not affected." |
| Current Value | Read from the backend when the page opens |
| Behavior | A selection is held until Save; Save applies it at once and writes it to the config file |

The control shows a loading skeleton during the first read; a failed first read replaces the page with the shared section error and its Try Again. A failed Save shows the save-failure toast with the reason, and the page keeps its changes.

**Time Zone Preference**: The second row selects how this browser renders timestamps (REQ-4.7.19). It is a preference of the browser with no backend interaction - the backend stores and returns UTC ISO-8601 exclusively and the frontend formats on read. Its section carries one subtitle saying how timestamps are shown in this browser, and the row is a label and a select with no help line. The select is disabled until the page's first read has answered, so nothing is staged before Cancel and Save are on the page.

| Element | Description |
|---------|-------------|
| Section Header | "Time Zone" with subtitle "How timestamps are shown in this browser." |
| Dropdown Label | "Time Zone" |
| Options | Browser local (default), UTC |
| Persistence | localStorage key `wave.timezone`; per-browser, survives reload and navigation |
| Behavior | A selection is held until the page is saved; Save re-renders every visible timestamp immediately, with no page reload. No success toast: Cancel and Save going back to disabled shows the save |

The dropdown uses the standard Select component. The current selection affects every timestamp the UI renders, which includes but is not limited to:

- Console panels (Workbench live, Execution Detail live, Historical Test Canvas)
- Relative-time cells on the Executions, Tests, Suites, Regressions, and Profiles lists
- Execution Detail Info panel (Started, Completed, Duration)
- Dashboard metric cards and Recent Executions entries
- Entity detail metadata ("Last run", "Created", "Updated")
- About > Build Date and other About-tab timestamp metadata
- Any timestamp rendered inside a modal, toast, or tooltip

**Uniform display via a central formatter**: A single `<Timestamp>` component reads the preference and renders accordingly. Every surface that displays a timestamp uses this component - direct `new Date().toLocale*()` calls in feature code are discouraged. This guarantees that toggling the setting flips every timestamp in one frame and prevents the inconsistency where the console formerly forced UTC while everything else ran in browser local.

**Display styles**: The formatter supports a small, closed set of display styles that match existing uses. Every style emits a trailing time-zone token in both modes - the browser's IANA short name in Browser local mode (e.g. `PDT`, `PST`, `CET`) and the literal `UTC` in UTC mode. The suffix removes the "same digits, different instant" ambiguity that an unsuffixed render would leave behind, and applies even to date-only displays where the calendar day depends on the selected zone.

| Style | Format (example, Browser local / UTC) | Used By |
|-------|----------------------------------------|---------|
| `time` | `14:32:14.027 PDT` / `18:32:14.027 UTC` | Console log entries |
| `datetime` | `Apr 23, 14:32 PDT` / `Apr 23, 18:32 UTC` | Info panels, compact timestamp cells |
| `date` | `Apr 23, 2026 PDT` / `Apr 24, 2026 UTC` | Audit fields, About > Build Date, relative-time fallback |
| `relative` | `3 min ago`, `yesterday`, ... ; `Apr 23, 2026 PDT` / `Apr 24, 2026 UTC` past 7 days | List columns (Last run, Started) |

The relative style is timezone-agnostic (all arithmetic is in absolute time), so only the fallback absolute date it emits after 7 days is affected by the preference; that fallback inherits the `date` style's zone suffix.

**Scope**: The preference does NOT affect backend log files, backend stdout, execution storage files, CLI output, or any API request/response payload - those remain UTC ISO-8601 everywhere. It is purely a UI display preference. Filename builders that incorporate a timestamp (for example, downloaded backup filenames) are out of scope because they are not rendered to the screen.

**Default**: Browser local, because most users want timestamps to read naturally in their own day/time. UTC is the operator/cross-team-coordination choice and must be picked explicitly.

**Network Section**: Network is a collapsible parent in the section navigation, between Variables and MCP Server, with three children: SSH Hosts, Certificates and Proxy, and Proxy is itself a parent with two pages, Setup and Blocks (NET-1, NET-10). The group opens when a Network page is shown and then stays as the user leaves it, open or closed, across sections, across leaving Settings and across a page load, for the browser session (SHELL-9). Each page that holds settings carries Cancel and Save in its header; Proxy Blocks holds none. Every change on a page is held on the page - a switch, a choice, rows added from a dialog, a row's trash icon, Mass Action Delete: Save and Cancel are disabled until something changed, Save writes everything in one step, and Cancel restores the saved state. Leaving a page with unsaved changes asks first: Cancel stays, Discard drops the changes and leaves, Save saves and leaves once the save succeeds. A staged removal therefore asks for no confirmation, since Cancel is the undo, and an addition shows no toast: the new rows appear in the table and Save enables. Save sends only the changes, so entries added elsewhere while the page is open are kept, and a conflict found at Save is shown with the usual save-failure toast while the page keeps its changes. Every table is the shared data table with search, sortable headers and pagination; editable tables carry Mass Action with Delete. Section headers are Title Case, with no subtitle that repeats the title. A row with a help line centers its control against the label and the help line; this is a rule of the shared settings row, so the Secret Masking rows in General and the MCP Server rows follow it too (NET-11). The network settings are managed only here; the MCP tools do not expose them.

- **SSH Hosts** (NET-2 to NET-4): Host Key Policy holds Lock trusted hosts and Default block action (REQ-4.5.14), then the Trusted SSH Hosts table lists Host, Port, Key type, Fingerprint and Added. Add hosts opens a dialog that either detects a host's keys or reads pasted lines in an editor with line numbers, previews what it found with a checkbox per row, marks keys already trusted or changed, and lists each bad line by its number.
- **Certificates** (NET-5 to NET-9): Public Certificate Authorities offers two option cards, WAVE's built-in list and this WAVE host's store (REQ-4.5.16). Under the built-in list, the Built-in Authorities table lists the bundled authorities read-only; under the host store, a read-only summary describes the WAVE Host's Store. The Additional Certificates table (Name, Type, Expires) always follows. Add certificate takes pasted PEM text or a dropped file and previews every certificate found. An eye icon on any certificate row opens the certificate's details.
- **Proxy Setup** (NET-10): One line under the title says what the proxy is for: "For web requests (HTTP and HTTPS) made from the WAVE host. Your browser and blocks on a remote host do not use it." The card holds Use a proxy, then Source - the WAVE host's settings, shown read-only with passwords masked, or Custom with IP or Hostname, Port, Username, a Password in the masked field every protected value uses, Use the proxy for with its two modes ("All addresses except the list" and "Only the addresses in the list"), the address list of Exact and Regex rows, and Use the proxy for local addresses too (REQ-4.7.14); the page validates while the user types and disables Save until everything is valid. With the proxy off the card holds only Use a proxy.
- **Proxy Blocks** (NET-10): Three sections that open and close, "Blocks that use this proxy", "Blocks that do not use this proxy" and "Not applicable", the last one closed at first, each listing its blocks in their palette groups and colors (REQ-3.4.2.10). The page holds nothing to save, so it carries its title alone, with no Cancel and no Save.

**Saving settings**: General, the Network pages, MCP Server and the periodic backup schedule follow one rule (REQ-4.7.16, NET-1): every change waits for the page's Save, Cancel and Save are disabled until something changed, a successful Save shows no toast and leaves both disabled, and a failed Save shows the save-failure toast with the reason while the page keeps its changes. Leaving such a page with unsaved changes shows the Unsaved changes dialog: Cancel stays, Discard drops the changes and leaves, Save saves and leaves once the save succeeds; on MCP Server that Save first asks the page's restart or exposure confirmation. What a page saves goes to this WAVE's config file, or to the data folder when it is shared work (REQ-4.7.17).

**About Section**: Displays read-only system information including application version number, build identifier or timestamp, and support contact information or documentation links. This section provides operators and users with the information needed for troubleshooting and support requests.

**Visual Integration**: The Settings page follows design system tokens for consistent styling:

| Element | Property | Token |
|---------|----------|-------|
| Navigation column | Width | 240px fixed |
| Navigation item | Height | Standard nav item |
| Active item | Background | `--color-primary` |
| Active item | Text color | `--color-white` |
| Divider | Color | `--color-border-default` |
| Table | Styling | Standard table component |
| Pagination | Styling | Standard pagination component |

**Interaction Patterns**: Section navigation updates URL path for bookmarkability and browser history support. Tab changes are instantaneous without loading indicators since content is lightweight. Form submissions in dialogs provide immediate feedback through toast notifications. Delete confirmations protect against accidental data loss.

**Route Configuration**: The Settings page is accessible at `/settings` with the default section being General. Direct links to specific sections use path parameters: `/settings/general`, `/settings/environments`, `/settings/variables`, `/settings/network/ssh-hosts`, `/settings/network/certificates`, `/settings/network/proxy/setup`, `/settings/network/proxy/blocks`, `/settings/mcp`, `/settings/server`, `/settings/backup`, `/settings/about`; `/settings/network` alone opens SSH Hosts, and `/settings/network/proxy` alone opens Proxy Setup.

##### REQ-4.7.19 Time Zone is a browser preference

The Time Zone in Settings > General is a preference of the browser it is chosen in: it changes how that browser shows every timestamp and nothing else. Its section says so in one subtitle, how timestamps are shown in this browser. The WAVE server is not affected and records and sends all its times in UTC: log lines, stored timestamps, API payloads, backup and artifact file names and report times are UTC on every WAVE host, and the periodic backup schedule runs on UTC. Like the page's other settings, a new choice applies when the page is saved; it is kept in the browser's own storage and never reaches the server.

#### 15.8.3 Profile Management

##### REQ-4.7.4 Profile management UI

The Profile Management interface enables users to create, configure, and maintain connection profiles that encapsulate reusable connection configurations for external systems. Profiles live in the LIBRARY section of navigation, providing a centralized repository for connection definitions used across tests and workflows.

**Profiles List Page**: The main entry point displays all profiles in a paginated table following the standard LIST pattern established by other entity management screens. The table presents essential profile information enabling users to identify, filter, and manage their connection configurations.

| Column | Content | Behavior |
|--------|---------|----------|
| Name | Profile display name | Sortable, searchable |
| Type | Profile type badge (database, ssh, http, etc.) | Filterable by type |
| Tags | Tag badges | Filterable |
| Usage | "N tests" count | Determines edit/delete constraints |
| Actions | Edit button + kebab menu | Duplicate, Delete in menu |

The page header contains the title "Profiles" and a "New Profile" button that opens the creation modal. Unlike execution entities (Tests, Suites, Regressions), profiles have no Status filter since they do not execute. Mass Action offers Export, Duplicate and Delete (PROFILE-5).

**Usage Constraints**: When a profile has non-zero usage count (referenced by tests), existing configuration fields become locked to prevent breaking changes. Users may still add new parameters. Delete action is blocked for profiles in use, with the menu item disabled and tooltip explaining the constraint. Duplicate action creates a new profile with Usage = 0, enabling full edit freedom on the copy.

**Profile Create/Edit Modal**: The configuration modal provides a compact form layout approximately 460 pixels wide. The header displays "Create Profile" or "Edit Profile" with a close button. The body organizes configuration into metadata and configuration sections. A form taller than the viewport scrolls within the modal, so the footer actions stay reachable for every profile type.

**Metadata Section**: The top section captures profile identity:
- Name and Description inputs displayed side-by-side
- Type selector (dropdown) for selecting profile type - only editable during creation, locked after save
- Tags input for categorization and filtering

**Type Selection**: Profile type determines the configuration schema. Available types include database, SSH, HTTP, Kubernetes, OpenSearch, and custom. Each type provides a predefined schema with suggested fields. The custom type accepts arbitrary key-value configuration for systems not covered by predefined schemas. Creation opens on the custom type with an empty parameter list, so a profile can be named and created without first choosing a type. Choosing a different type during creation replaces the parameter rows with that type's fields; when the rows hold values the author entered, the change asks for confirmation first, and canceling leaves the type, the rows and the entered values exactly as they were. Type cannot be changed after profile creation to maintain configuration integrity.

**Configuration Section**: The core configuration displays type-specific fields based on the selected profile type. For predefined types, the form presents relevant fields (host, port, username, etc.) with appropriate input controls, each row opening on the starting value its type declares, in every environment, and blank where the type declares none. For custom type, an editable key-value table allows arbitrary parameters with columns for Name, Type (string, integer, float, boolean), secret indicator (lock icon), Value, and Actions.

**Per-Environment Values and Copy Values**: Profile parameters store values per environment, allowing the same profile to have different configurations for different deployment targets. An environment selector dropdown enables switching between environments to view and edit their respective values. A "Copy Values" button opens a modal that enables copying parameter values from one environment to another. The copy values environment selector supports searching by both environment name and tags, enabling users to quickly locate source environments in large environment collections. The selector organizes environments into Recommended (highest tag overlap with the current environment), Other (remaining environments with values), and disabled entries (environments with no values set). Tag match counts display alongside each environment name to guide selection.

**Secret Handling**: Fields marked as secret in the type schema display masked values (asterisks) in the interface. The secret indicator (lock icon) identifies sensitive values that receive encryption at rest and masking in logs. Secret values are accepted during creation and update but never returned in API responses - retrieval shows masked placeholders.

**Modal Footer**: Create mode shows Cancel and Create buttons. Edit mode shows "Delete Profile" button (red, left side, disabled when Usage > 0), Cancel, and Save Changes buttons. The submit button stays disabled until something has changed in the form or in a value editor open from it, and closing the form with changes asks the shared unsaved-changes question first (see Editing surfaces and the floating editors opened from them).

**Route Configuration**: The Profiles page is accessible at `/profiles` in the LIBRARY section of navigation. The page follows standard navigation patterns with sidebar visibility and breadcrumb context.
