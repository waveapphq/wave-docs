### 15.5 Configuration Editors

#### 15.5.2 Multi-Format Editors

Multi-format string editing (per-format validation, conversion, and formatting across plain / JSON / XML / YAML) is provided by the shared value editor's string body - the same type-aware surface described under component-specification Variable Type Editors (§ 15.15.5), opened from any string value's editor (variable, Variable Setter, profile parameter, plugin parameter, entity-scope variable). There is no separate text-editor dialog.

##### REQ-3.6.2.1 YAML syntax validation

The shared string value editor provides real-time YAML syntax validation via a custom CodeMirror linter that wraps `js-yaml`. On each document change, the linter parses the content and reports parse errors with position information. Error positions are clamped to document bounds to prevent crashes on partial edits. Non-YAML exceptions are silently ignored (empty diagnostics returned).

##### REQ-3.6.2.2 JSON syntax validation

JSON syntax validation uses CodeMirror's built-in `jsonParseLinter` from `@codemirror/lang-json`. Errors appear inline as the user types. For a free-form string the lint is advisory - a string is valid as a string, so an invalid-JSON body never blocks saving. For a parameter whose format is locked to JSON (a strict data parameter such as a JSON parser's content), the same well-formedness check is promoted to a blocking gate: an invalid literal body raises a validation strip and disables Save until the body parses. A template or expression body bypasses the gate - its final text is produced at runtime and validated there.

##### REQ-3.6.2.3 JSON Schema validation for JSON files

##### REQ-3.6.2.4 XML editor with XSD validation support

The shared string value editor provides XML syntax validation via a custom CodeMirror linter that uses the browser's native `DOMParser`. Invalid XML produces a `parsererror` element whose text content is extracted as the error message. The linter attempts to extract line and column numbers from the error message (format varies by browser engine) and maps them to document positions for inline error display. As with JSON, this check is advisory for a free-form string and blocking for a parameter whose format is locked to XML: an ill-formed literal body disables Save until it is well-formed. Schema (XSD) validation is not applied - well-formedness only, matching the runtime parser.

##### REQ-3.6.2.5 XSD loading and referencing in XML editor

##### REQ-3.6.2.6 Format conversion (JSON↔YAML)

The shared string value editor supports cross-format conversion. When the user selects JSON format and clicks Format on YAML content, the modal parses via `js-yaml` and serializes to JSON with 2-space indentation. This enables a one-click YAML-to-JSON workflow. The reverse direction (JSON-to-YAML) works the same way - select YAML format, click Format on JSON content.

##### REQ-3.6.2.7 Diff view for comparing versions

##### REQ-3.6.2.8 JSON formatter/beautifier (indent, sort keys)

JSON formatting uses `JSON.parse` followed by `JSON.stringify` with 2-space indentation. If content is not valid JSON but is valid YAML, the formatter falls back to parsing via `js-yaml` and serializing to indented JSON.

##### REQ-3.6.2.9 YAML formatter (proper indentation, alignment)

YAML formatting uses `js-yaml` parse-then-dump with 2-space indentation and infinite line width (no wrapping). A fallback path handles inconsistent indentation by stripping all leading whitespace from each line before re-parsing, enabling recovery from copy-paste artifacts.

##### REQ-3.6.2.10 XML formatter (indent, attribute formatting)

XML formatting uses the `xml-formatter` library with 2-space indentation. The `collapseContent` option keeps text-only elements on a single line (e.g., `<item>Hello</item>` remains inline rather than splitting the text node onto a separate line). This preserves readability for data-oriented XML where element values are typically short strings. The formatter handles processing instructions, self-closing tags, and nested structures. Malformed XML is returned unmodified (the formatter does not throw).

#### 15.5.3 Expression Validation

##### REQ-5.2.4.8 Client-side expression validation

The frontend provides comprehensive expression validation for expression-mode parameter fields and computed variable definitions. This client-side validation provides immediate feedback without requiring backend round-trips, improving the editing experience.

**Validation Categories**: Expression validation detects three types of issues:

| Category | Code | Severity | Description |
|----------|------|----------|-------------|
| Unknown variable | `unknown_variable` | Error | Variable not in available variables list |
| Invalid function | `invalid_function` | Error | Function not in allowed builtins |
| Blocked function | `blocked_function` | Error | Security-blocked pattern detected |

**Expression Classification**: The validator identifies expression types for appropriate validation:

| Pattern | Example | Classification |
|---------|---------|----------------|
| Simple variable | `my_var` | Variable reference |
| Method chain | `var.strip().upper()` | Method call |
| Function call | `len(items)` | Function invocation |
| Arithmetic | `a + b * c` | Binary operation |
| Comprehension | `[x for x in items]` | List comprehension |
| Lambda | `filter(lambda x: x > 0, items)` | Lambda expression |

**Variable Name Suggestions**: When an unknown variable is detected, the validator suggests similar variable names using Levenshtein distance:

| Setting | Value | Purpose |
|---------|-------|---------|
| Maximum distance | 2 | Catches typos, not unrelated names |
| Suggestion limit | 3 | Prevents overwhelming suggestions |

For example, `my_vra` with available `my_var` produces: "Unknown variable 'my_vra'. Did you mean: my_var?"

**Local Scope Detection**: The validator correctly identifies local variables that should not trigger undefined variable errors:

| Construct | Local Variables | Example |
|-----------|-----------------|---------|
| List comprehension | Loop target | `[x.name for x in items]` - `x` is local |
| Dict comprehension | Key and value targets | `{k: v for k, v in pairs}` - `k`, `v` are local |
| Generator expression | Loop target | `(x for x in items)` - `x` is local |
| Lambda parameters | All parameters | `lambda x, y: x + y` - `x`, `y` are local |
| Tuple unpacking | All unpacked names | `[a + b for a, b in pairs]` - `a`, `b` are local |

**Keyword Argument Detection**: Named arguments in function calls are not variable references:

```
sorted(items, key=name, reverse=True)
#             ^^^       ^^^^^^^
#             These are keywords, not variable references
```

**Type-Aware Method Validation**: The validator checks method compatibility with variable types when type information is available:

| Variable Type | Valid Methods | Invalid Example |
|---------------|---------------|-----------------|
| string | `.strip()`, `.upper()`, `.split()` | `.append()` (list method) |
| array | `.append()`, `.pop()`, `.sort()` | `.upper()` (string method) |
| integer | None (use functions) | `.upper()` (string method) |

**Security Pattern Blocking**: The frontend mirrors backend security rules:

| Pattern Type | Examples |
|--------------|----------|
| Dunder access | `__class__`, `__import__` |
| Dangerous functions | `import`, `exec`, `eval`, `open` |
| Introspection | `globals`, `locals`, `vars` |
| Attribute manipulation | `getattr`, `setattr`, `delattr` |

**Syntax Validation**: The validator rejects invalid Python constructs:

| Rejected Construct | Example | Error |
|--------------------|---------|-------|
| Assignment statement | `x = 5` | "Assignment not allowed" |
| Walrus operator | `x := 5` | "Assignment expression not allowed" |
| Unbalanced brackets | `func(arg}` | "Syntax error: unbalanced brackets" |
| Invalid method call | `1.upper()` | "Cannot call method on numeric literal" |

**String Literal Handling**: Variable references inside string literals are not validated as variables:

```
f"Hello {name}"  # 'name' inside f-string IS a reference
"name"           # 'name' inside regular string is NOT a reference
```

**Validation Result Format**: Each validation issue includes:

| Field | Type | Purpose |
|-------|------|---------|
| type | `"error"` or `"warning"` | Severity level |
| code | string | Issue classification |
| message | string | Human-readable description |
| startIndex | number | Position in expression |
| endIndex | number | End position for range highlight |
| suggestions | string[] | Alternative names (for unknown_variable) |

**Integration with Expression Input**: The `ExpressionInput` component uses validation to provide:

| Feature | Behavior |
|---------|----------|
| Inline error highlighting | Red underline on invalid portions |
| Error tooltip | Hover reveals full error message |
| Autocomplete filtering | Suggestions exclude blocked functions |
| Save prevention | Cannot save with validation errors |

##### Template Mode Expression Validation

The shared string value editor provides a Template mode (selected via the value's Kind control, where the surface offers one) that supports `{{expression}}` markers within structured content. Template validation reuses the same AST-based expression validator as expression fields:

**Validation Flow**: The modal scans content for `{{...}}` regions using regex, extracts the expression content between delimiters, validates each expression independently, then maps expression-local positions back to document-global positions for inline display.

**Visual Feedback**: Template expressions are highlighted with cyan styling that covers the full `{{ expr }}` region including delimiters. Unknown variables show amber wavy underlines; invalid functions show red wavy underlines. Hover tooltips display error messages on validation issues.

**Autocomplete**: `Ctrl+Space` inside `{{ }}` triggers variable and function autocomplete using the same completion source as expression fields. Tab accepts the selected completion; Escape closes the dropdown before closing the modal.

**Format Interaction**: Template mode disables the Format button since `{{expression}}` markers would break JSON/YAML/XML parsers. Syntax highlighting for the underlying content format (auto-detected) remains active.

#### 15.5.4 AST-Based Parsing Architecture

##### Parsing Strategy

The expression validator uses Abstract Syntax Tree (AST) parsing to provide accurate Python syntax validation in the browser. This approach ensures validation behavior aligns with Python's actual parsing rules, avoiding edge cases that regex-based patterns cannot handle.

**Parser Selection**: The frontend uses tree-sitter with a Python grammar compiled to WebAssembly (WASM). Tree-sitter provides incremental parsing, error recovery, and precise position tracking for syntax errors.

**Architecture Layers**:

| Layer | Responsibility |
|-------|----------------|
| WASM Loader | Lazy initialization of tree-sitter runtime and Python grammar |
| AST Parser | Parses expression content into syntax tree |
| Node Visitor | Traverses AST to extract identifiers, function calls, method chains |
| Semantic Validator | Checks extracted elements against available variables and allowed functions |
| Issue Reporter | Formats validation issues with positions for UI highlighting |

**Lazy Loading Strategy**: The WASM files load on-demand when expression validation is first invoked, avoiding impact on initial page load performance. The grammar is cached after first load for subsequent validations.

| Asset | Approximate Size | Loading Trigger |
|-------|------------------|-----------------|
| tree-sitter.wasm | ~200KB | First expression validation |
| tree-sitter-python.wasm | ~200-300KB | First expression validation |

**Error Handling**: If WASM loading fails, the validator displays an error state indicating validation is unavailable. No fallback implementation is maintained since all modern browsers support WebAssembly.

##### AST Node Processing

The parser extracts semantic information from AST nodes:

| Node Type | Extracted Information |
|-----------|----------------------|
| `identifier` | Variable or function name with position |
| `call` | Function name and argument expressions |
| `attribute` | Object and method name for chain validation |
| `list_comprehension` | Loop variable names (local scope) |
| `lambda` | Parameter names (local scope) |
| `assignment` | Detected as invalid in expression context |

**Scope Tracking**: The visitor maintains a scope stack to correctly identify local variables introduced by comprehensions and lambdas, preventing false "undefined variable" errors.

##### Performance Characteristics

| Metric | Target |
|--------|--------|
| Parse latency (typical expression) | < 5ms |
| Memory overhead | < 2MB for cached parser |
| First validation (cold start) | < 100ms including WASM load |
| Subsequent validations | < 10ms |

The parser is designed for interactive use during typing. Validation debouncing prevents excessive parsing during rapid input.

##### Backend Alignment

The frontend AST-based validation mirrors the backend's Python `ast.parse()` behavior. Both systems use the same validation rules, ensuring consistent error messages between immediate client-side feedback and authoritative server-side validation.

#### 15.5.5 Plugin Parameter Value Editing

Plugin parameter values are edited through the same shared value editor as variables, with a per-parameter contract that keeps strict fields strict and free-form fields flexible.

##### REQ-3.4.2.2 Plugin parameter definition

Each string parameter declares the set of text formats its value may take, and that declared set drives the editor:

- A parameter that permits a single format locks the editor to it, and the value is never auto-detected as another format. A structured-content field (a JSON, XML or YAML body) locks to its data format: the Format control is shown but disabled, pinned to that format. A field that takes plain text only (a URL, a host, a file path, a PEM certificate, a secret, a script body read as plain text) locks to plain text and shows no Format control at all, so a literal value is never mis-highlighted or mis-detected as JSON or YAML. A profile field follows the same rule: the profile type marks its URL, host and certificate fields as plain text, and a secret parameter is always plain text.
- A parameter that permits several formats, or declares none, offers the full enabled Format control with auto-detection (a free-form HTTP request body or a log message can be authored in any format).
- A parameter declared as a Python script body opens the editor in a Python mode (PYRUN-13): Python coloring using the editor's existing highlight styles for keywords, strings, numbers, comments and function names, with identifiers, operators and brackets in the normal text color in both themes; line numbers; a syntax check by the same Python parser the expression editor already loads, and live checks of names, methods and operators on that same parse; and autocomplete offering the block's Input Variables by name plus the full Python builtins catalog, because a script is not an expression and the blocked-builtins filter does not apply. The editor shows neither the Kind nor the Format control for the body: it is never template-expanded, so `{{` in the script is just Python, and its format is fixed. The footer's left side carries one chip, `F1 Help`, which opens a help popover with the keyboard list and a collapsed `Show tips` section; no hints or tips appear anywhere else on the sheet. Syntax errors show as underlines in the text and in the validation strip, the first message with a "(+N more)" expander; a syntax error never blocks Save but marks the block invalid on the canvas, so a block with a syntax error cannot run. The check has two halves. While the author types it is the editor's own Python parser, in the browser, with no request: it reports one entry per error, each naming the line and saying what that parser can say about it. On Save and on Run the server checks the same script with the interpreter's own parser and its precise sentence replaces the editor's wording in the strip and in the block's issue. Neither half ever blocks a save - not the editor's Save, not the configuration dialog's, not the test's. The same parse, with no request, also checks the script for what the expression field already catches, wherever that check holds for a whole script: a name read that nothing defines where Python would look (no binding in a scope Python searches, following Python's module, function, lambda, comprehension and class scopes, and not an Input Variable, a global the bridge sets or a Python builtin), reported as the expression field's unknown-function message for a call and its unknown-variable message, with its "Did you mean" suggestion, otherwise; an unknown method or a wrong argument count on an Input Variable whose type is known and that the script never rebinds, checked against the Python type the script receives; and a binary operator on literals or such Input Variables whose types Python rejects at run time, such as a float Input Variable, which arrives as Decimal, times 1.5. There is no flow analysis. A script with a star import or dynamic namespace access is not checked, and a script with a syntax error shows its syntax errors only. Each finding is underlined on its name, method or operator with the expression field's underline and listed in the validation strip with the syntax errors, in line order; the findings are warnings that never block Save, never disable Run and never mark the block invalid, and they appear in the block's issues as warnings. The editor opens at 720 by 460 in this mode, has a maximize control in its header, and offers no pretty-print. Python mode adds the editor facilities of PYRUN-18: auto-indent, bracket matching and auto-close, comment toggle, find and replace, go to line, code folding and active-line and selection-match highlight, and offers no format on save, hover or select-next-occurrence. Completion, documentation and rename are server-backed (PYRUN-19): the server introspects the block's interpreter for member completions with a documentation panel beside the list, and F2 Rename symbol renames every use in the script and the matching Input Variable or Variable Export row; a request never blocks typing and degrades to the catalog when it fails. The footer's right side reads Close, Save, Run, with Run primary and Shift+Enter as its shortcut (PYRUN-21). Run never saves and leaves the sheet open; the saved block, on the canvas and on the server, is untouched. When a debug run is paused at this block, Run is the Paused Toolbar's Repeat Step with the editor's script used for that one execution, and the session stays paused on the block afterwards, so the next Run is another repeat. This is the one editor a run in flight leaves live: the author edits the script and runs it although the configuration sheet around it is read-only, and because that sheet offers no Save while it is read-only, the edited script runs without ever being written. Otherwise Run executes only this block, exactly as the canvas's Run Current does, and it runs what the block's Save would save: the configuration sheet's whole unsaved draft, every editor open from it, and this editor's script, for that one run only. It runs against the Environment selected in the canvas toolbar and asks for one only when none is selected, and an Environment picked there becomes the toolbar's selection; it runs this block alone and ends when the block ends, so the next Run again runs this block alone. Run is disabled while a run is in progress, unless that run is paused at this block, and while something other than this script would block the sheet's Save - then it says why with that same note. For every parameter of every block the Cancel button is Close; Save and Ctrl+Enter apply the value and keep the editor open, and Save is disabled until the value differs from what was last saved; Close closes the editor, and Close with unsaved changes asks Keep Editing / Close / Save, where Save applies the value and then closes (PYRUN-20).

A parameter value can be a literal, a template, or an expression. Literal versus template is chosen inside the value editor; expression versus non-expression is toggled on the parameter row itself, so the editor only ever offers literal and template. The three kinds are mutually exclusive per parameter. A parameter declared as never taking an expression shows no Kind control in the editor at all: its value is always a literal, and its row already hides the expression toggle (PYRUN-16).

For a parameter locked to a data format, a literal body is validated eagerly against that format, and the validation is blocking: an ill-formed body raises a validation strip (with a "(+N more)" expander when several problems exist, identical to the complex-value Code view) and disables Save until the body parses. A template or expression body skips this gate, because its final text is produced at runtime and validated there.

When a node cannot be saved, the reason is never hidden behind a silently-disabled Save. Every missing or invalid field that blocks the save is outlined and labeled at the point of the problem as soon as it is invalid - a query with no assigned name, a required body left empty, an invalid query path - so the obstacle is always visible without the user probing each field. The value editor's minimum size always accommodates its full footer (kind, format, format action, cancel, save) so no control is ever clipped outside the sheet.

The configuration sheet owns the floating editors opened from it. Its Save stays disabled while nothing has changed in the sheet or in any editor open from it. Once anything has changed, Save writes every open editor's unsaved value into the sheet's draft and saves once, as a single undo step, and closing the sheet - its X, its Cancel button or Escape - asks first: Keep Editing returns, Close discards the sheet's draft and every open editor's, and Save is the sheet's own Save. A read-only sheet asks with Keep Editing and Close only. When the sheet's own fields are invalid, or an editor that has changes cannot be saved, both Saves are disabled and carry a red note naming what blocks them - "Save is disabled: the settings have errors" for the sheet's own fields, "Save is disabled: the Input Variables editor has errors" for an editor, by the row it was opened from. An editor showing errors it has made no change to blocks nothing. An action that would take away a row whose editor holds unsaved changes - a mode switch that hides the row, or deleting the row - asks the same question with Keep Editing and Close only: Keep Editing leaves the row exactly as it was, Close carries the action out and the row, its editor and its unsaved text all go.

A run in flight makes the configuration sheet read-only: the READ-ONLY badge in its header, no Save in its footer, and frozen fields both on the sheet and in every editor open from it. The one exception is the open script editor of the block a debug run is paused at. That editor stays live, so the author edits the script and presses Run, which repeats the paused block with the edited script. The row it was opened from is frozen like every other, so the editor is the one the author already had open when the run reached the block. Every other editor of that sheet, and every other block's sheet, stays frozen, and because the sheet still offers no Save the edited script reaches neither the canvas nor the server until the run ends.

A query-parser configuration (JSON or XML) is organized into four sections in execution order, each a peer of the others: the input content, the queries, the output mode, and the output bindings. The content field is the only input parameter. The queries are their own section (an alias, a path that can itself be a literal or an expression, and - in individual output mode - a per-query result type). The output mode is its own section presenting the combined-versus-individual choice as a labeled option group, consistent with the branching-mode option group used by the assertion control; combined returns one results map, individual emits each query result as its own output variable. The output bindings (the parser's success, errors, and results outputs) are the final section, with the results binding hidden in individual mode. The output mode is a fixed enumeration, not an expression-bearing field.

**Ordered argument lists** (SHRUN-4): an array parameter declared as an ordered list is edited as rows directly on the config sheet. Each row is a value field with its own expression toggle, labeled with its position (`$1`, `$2`, `$3` and so on), followed by the same up/down arrow pair the If-Else, Switch and Parallel branch rows use - up disabled on the first row, down disabled on the last - and a delete button. "+ Add argument" sits under the list and appends a row. The order of the rows is the order of the value: the block receives the list top to bottom, and an empty row is passed as an empty value.

**Variable exports** (SHRUN-6): the Output Variables section of a Shell Script block lists its fixed outputs first and ends with a table of variable exports. Its columns are Script Variable, Type, Secret and WAVE Variable: an editable shell variable name, the Variable Setter's type select (VARSET-5: locked to the variable's type when the WAVE Variable cell names a variable that exists; otherwise, for an empty cell or a name that is not a variable yet, a free select, String by default, and a variable that does not exist yet is created with the chosen type when the sheet is saved; picking an existing variable in that cell sets the row's type to that variable's type), the Secret checkbox every output row carries, and the ordinary binding picker; a remove button closes the row and the table's footer row is the "Add variable export" button. A name must be a shell identifier - a letter or underscore followed by letters, digits or underscores - may not take the name of one of the block's fixed outputs (stdout, stderr, exit_code, success, error, reason) and may not repeat another row's name. Like the Variable Setter (VARSET-8), a name error shows once the name field has lost focus and from then on updates as the author types, so a just-added row, whose name field takes focus, shows no error until the author leaves it; the error sits on its own line under the row, so it never moves the row's other fields, and it does not repeat the name the field shows: "Name is required" (empty), "Name must start with a letter or underscore, and contain only letters, numbers, and underscores" (not a shell identifier), "A block output already has this name" (the name of a fixed output), "Another variable export has this name" (a repeated name). The canvas issue and the validation message for a repeated name carry the name: "Script variable '{name}' is listed more than once". The captured value is read as the bound variable's type, a row left unbound is read and discarded, and an export the script never set leaves its WAVE variable with the value it had.

**At least one of a group of rows** (SHRUN-13): a group of rows can require that at least one of them holds a value, optionally only while a condition holds. No row in the group is required on its own and any number of them may be set; when every row is empty the group reports one message for the group rather than a required-field message on each row. Shell Script's and Shell Exec's Pass pattern and Fail pattern form such a group while Stop on pattern is ticked, and the message reads "Set a pass pattern, a fail pattern or both".

**Map key rules** (SHRUN-5): a map parameter can constrain its keys with a name pattern and with prefixes a key may not start with, each carrying its own message. A key is checked against the pattern first and against the prefixes only once it passes; a failing key is flagged in the map editor and as a canvas issue. Shell Script's and Shell Exec's Input Variables use the shell identifier rule and refuse the WAVE_ prefix, with "Invalid variable name: '{name}'" and "Variable names may not start with WAVE_: '{name}'".

**Numeric bounds** (PING-17, PING-18): a numeric parameter declares the lowest and the highest value it accepts, once, in the block's parameter definition, and that declaration is real on both sides of the wire. The configuration sheet flags a literal number outside the declared range on the block, and the block's own validation still refuses the value when the block runs, in the block's own words: the declaration feeds the sheet, it never takes over what a block says at run time. A bound may name another row rather than a fixed number, so Minimum replies is flagged while it exceeds Count; that is the numeric form of the ordering rule already used for ordered choices, and it carries its own message the same way. The rule is deliberately narrow. A value written as an expression is never range-checked, because its value is not known until the run, and neither is a blank row. Nothing is clamped, rewritten or refused at the keystroke: what the author typed is kept and the block is flagged instead, because silently rewriting a typed value is worse than refusing it. A range problem draws no new surface; it appears wherever a block's other configuration problems already appear, in the sheet and on the canvas. And a numeric row that declares no bound is not range-checked at all, so a row is exactly as strict as its own declaration.

**File references** (TDATA-8, TDATA-9, TDATA-10): a parameter descriptor marks a string value as a file reference in one of four modes fixed per field - file, file-or-folder, folder, or save-as - and the row gains a Browse icon button between the value field and the expression toggle. Browse is hidden in expression mode and on read-only canvases. Inside a Remote Execution body it is hidden only for a block that runs on the remote host, which refuses `testdata://` and `artifact://` paths at run time; a block that runs on WAVE inside the body (the L badge) keeps it, because its paths are WAVE paths. It opens one floating picker sheet of the config sheet's sub-editor family (the shell the value editor and the cipher picker use, 520 px). A field opens its Edit value sheet or its picker, never both: pressing the field while its picker is open, or Browse while its Edit value sheet is open, closes the open sheet first, and when that sheet holds unsaved changes it asks Keep Editing, Close or Save - Keep Editing leaves it open and brings it to the front, Close discards its changes, Save writes them into the field so the other sheet opens on the new value. Switching the field to an expression closes whichever of the two is open, under the rule for a row the surface takes away. The picker's primary button reads Select in every mode. The file mode is titled "Choose a file" and selects files; file-or-folder is titled "Choose a file or folder" and selects whichever of the two is picked; folder is titled "Choose a folder", shows files muted and selects folders; save-as is titled "Save as" and lists the session's artifacts only under the hint "Folders and files in this session's artifacts".

The picker body offers a Test Data tab and an Artifacts tab; save-as has no Test Data tab, and a field marked artifacts-only drops it too, because those fields name something that must be writable. The Test Data tab lists the open test's files with a filter and an Upload button. Single click selects, double click chooses, Enter confirms. Save-as adds a Name field under the tree: picking a folder keeps the Name, picking a file takes its name, and for Copy and Move the Name is preset to the source's file or folder name. Every mode ends with a Reference line showing the exact value that will be written - the scheme of the tab in front, fixed (`testdata://` on Test Data, `artifact://` on Artifacts, the only tab of save-as and of an artifacts-only field), followed by the path under it, which can be typed - and Cancel. Each tab keeps its own path, and the picker opens on the tab of the scheme the field already holds; a field value under neither scheme opens with an empty path and stays as it is until Select. A path outside both schemes is typed in the field itself, not through Browse. The Artifacts tab states that a block reads only what an earlier block in the same run wrote; its empty state reads "Files this session wrote appear here. Type a path an earlier block writes."

The fields that carry Browse, and their modes: File I/O Read Path (file); Write Path, Copy and Move Destination, and Create folder Path (save-as); Copy Source, Delete Path and Stats Path (file-or-folder); Move Source (file-or-folder, artifacts only); List Path (folder); HTTP File Source and multipart file parts (file) and File Path, the download target (save-as; empty keeps the automatic name and location, an explicit path replaces an existing file, an automatic name keeps its numbered suffix); Excel Workbook on Read (file) and on Write (save-as); K8s Copy's WAVE-side path, Source when To Pod (file-or-folder) and Target when From Pod (save-as); Helm Install and Upgrade Chart (file-or-folder; a WAVE-side file or folder is resolved before helm is invoked when the block runs on WAVE, and helm's single-argument model of name, path or OCI reference is kept); Shell Script's Script File (file); Python Script File (file); Netcat File (file); SFTP Upload Source (file-or-folder) and Download Destination (save-as); Shell Script and Shell Exec Working Directory (folder). The SSH Script working directory, SFTP server-side paths, pod-side paths, certificates and keys, the K8s Apply manifest and the CSV and Excel schema sources are not file references. On every row that carries Browse, the value preview truncates from the start so the file name at the end of the reference stays visible.
