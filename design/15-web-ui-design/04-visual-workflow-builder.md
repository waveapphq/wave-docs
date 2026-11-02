### 15.4 Visual Workflow Builder

#### 15.4.0 Canvas Page Layout

The canvas page provides a dedicated full-screen workspace for visual workflow editing. Canvas pages employ specialized styling that maximizes workspace area while maintaining full sidebar functionality.

**Canvas Routes**: The application recognizes specific routes as canvas contexts requiring specialized layout styling:

| Route Pattern | Context | Purpose |
|---------------|---------|---------|
| `/workbench?editTest={testId}` | Test editing | Edit existing test workflow |
| `/workbench` | Workflow creation | Create new test from scratch |

**New Test Workflow**: The "New Test" action does not have its own route. Instead, clicking "New Test" navigates to `/workbench` and automatically opens the Save Test Dialog. This ensures test metadata (name, description, tags, suite) is captured before any workflow editing begins. The workflow is:

1. User clicks "New Test" button (from Tests list or navigation)
2. Application navigates to `/workbench`
3. Save Test Dialog opens automatically
4. User enters test metadata and clicks Save
5. Test is created, Workbench continues with the new test context
6. If user clicks Cancel, they remain in Workbench with an unsaved draft

**Layout Behavior**: Canvas routes receive specialized styling:

- Standard breadcrumb bar is replaced with canvas-specific breadcrumb bar
- Page-level padding is removed to maximize canvas space
- Sidebar behavior follows viewport-based responsive rules (not route-based)

**Panel Structure**: The canvas page organizes into distinct functional panels:

| Panel | Position | Default Width | Behavior |
|-------|----------|---------------|----------|
| Navigation | Left edge | 200px expanded, 56px collapsed | Viewport-responsive (see below) |
| Block Palette | Left of canvas | 260px expanded, 64px collapsed | User-collapsible |
| Canvas + Console | Center | Fills remaining | Console expands upward, resizable 100-600px |
| Variable Manager | Right edge | 320px expanded, 64px collapsed | User-collapsible, resizable 240-600px |

**Panel Resize Mechanism**: The Console and Variable Manager panels use `react-resizable-panels` (shadcn `resizable` primitive) for drag-to-resize behavior. The canvas area is wrapped in a nested `ResizablePanelGroup` structure: an outer horizontal group (Canvas+Console | Variable Manager) containing an inner vertical group (Canvas | Console). The Block Palette sits outside the resize group - it uses its own expand/collapse toggle (64px ↔ 260px) and does not support drag-to-resize. Resize handles provide keyboard accessibility (Tab, Arrow keys). Individual panel collapse/expand remains controlled by each panel's toggle button.

**Sidebar Responsive Behavior**: The navigation sidebar responds to viewport width using a 1200px threshold:

| Viewport Condition | Sidebar Behavior | User Override |
|--------------------|------------------|---------------|
| Viewport > 1200px | Follows user preference (localStorage) | N/A - normal behavior |
| Viewport ≤ 1200px | Auto-collapses | User CAN expand manually |
| User manually toggles | State persisted to localStorage | Remains until browser cache cleared |

**Sidebar State Management**: The layout maintains responsive sidebar behavior with localStorage persistence:

- **Large viewport (>1200px)**: Sidebar follows user preference from localStorage
- **Small viewport (≤1200px)**: Auto-collapses by default on initial load
- **Manual toggle**: User's choice is saved to localStorage and persists across sessions
- **localStorage key**: `wave-sidebar-collapsed` stores the user's preference
- **Persistence scope**: User preference remains until browser localStorage is cleared

**Visual Continuity**: Sidebar transitions preserve user orientation:

- Sidebar collapse uses smooth animation (300ms ease)
- Navigation icons remain accessible for quick route changes
- Tooltips on hover provide label context in collapsed state
- User maintains full control over sidebar state at all viewport sizes

#### 15.4.0.1 Canvas Toolbar

The canvas toolbar provides primary actions for workflow editing and execution. It spans the full width of the canvas area, positioned as the TOP bar (above the breadcrumb bar).

**Toolbar Structure**: The toolbar organizes actions into logical groups separated by visual dividers:

| Group | Position | Contents | Purpose |
|-------|----------|----------|---------|
| Close | Left | Close | Dismiss workflow |
| Edit | Left (after Close) | Save | Persist changes |
| History | Left (after Edit) | Undo, Redo | Undo/redo history |
| Environment | Right | Environment selector | Runtime context selection |
| Execution | Right (after Environment) | Debug options, Debug, Run | Test execution controls |

**Left Side Layout**: Close | Save | Undo Redo (with separators between Close/Save and Save/History groups)

**Close Action**: The close button (×) dismisses the active workflow and returns to the parent list:

- Always visible on all canvas pages (including drafts)
- Triggers unsaved changes dialog if workflow is dirty
- Navigates back to Tests list (or parent context)
- Disabled during execution (running/paused states)

**Toolbar Dimensions**: The toolbar maintains consistent sizing for visual stability:

| Property | Value | Rationale |
|----------|-------|-----------|
| Height | 44px | Matches collapsed panel heights for visual consistency |
| Horizontal padding | 12px | Provides breathing room from panel edges |
| Button gap | 8px | Standard spacing between action buttons |
| Group gap | 16px | Visual separation between action groups |

**Action Button States**: Toolbar buttons reflect workflow and execution state:

| Button | Idle State | Running State | Paused State |
|--------|------------|---------------|--------------|
| Close | Enabled | Disabled | Disabled |
| Save | Enabled when dirty | Disabled | Disabled |
| Undo | Enabled when history available | Disabled | Disabled |
| Redo | Enabled when future available | Disabled | Disabled |
| Run | Enabled | Hidden | Hidden |
| Debug | Enabled | Hidden | Hidden |
| Stop | Hidden | Visible, enabled | Visible, enabled |
| Continue | Hidden | Hidden | Visible, enabled |
| Step Over | Hidden | Hidden | Visible, enabled |
| Step Into | Hidden | Hidden | Visible, enabled |
| Focus Current | Hidden | Visible, enabled | Visible, enabled |

**Environment Selector**: The environment selector appears as a compact dropdown in the toolbar:

- Displays currently selected environment name
- Opens searchable popup on click with recent and all environments
- Required before first execution (Run/Debug disabled until selected)
- Selection persists per test/workbench context
- "Remember selection" option sets default for future sessions

**Split Button Pattern**: Run and Debug actions use split button design for scope selection:

- Primary click executes the currently-opened scope (sub-block when the user has drilled in, the test otherwise)
- Dropdown arrow reveals scope options (Run Scope, Run All)
- Visual indicator distinguishes primary action from dropdown trigger

"Run Scope" targets only the currently-opened scope; "Run All" always targets the outermost parent test from its Start node, regardless of where the user is viewing. Per-element execution (single-element runs and partial-graph "from Here" runs) lives on the right-click context menu, not on this split button - see Execution Monitoring § 15.6.5 for the menu inventory.

**Debug Options**: A gear icon adjacent to Debug opens a popover with execution modifiers:

- "Pause on first block" checkbox
- "Pause on errors" checkbox
- Settings persist per user session
- Gear icon highlights when options are active

#### 15.4.0.2 Canvas Breadcrumb Bar

The breadcrumb bar provides navigation context and workflow status. It appears BELOW the toolbar (second bar from top), spanning the full canvas width.

**Breadcrumb Structure**: The bar organizes into distinct functional regions:

| Region | Position | Contents | Purpose |
|--------|----------|----------|---------|
| Navigation | Left | Back arrow, entity path | Context and navigation |
| Status | Center-right | Test Data chip, Artifacts chip (count badge), Reset action, status indicator | Workflow/execution state and the session's file surfaces |

**Breadcrumb Dimensions**: The bar maintains compact vertical presence:

| Property | Value | Rationale |
|----------|-------|-----------|
| Height | 36px | Minimal but readable |
| Horizontal padding | 12px | Aligns with toolbar |
| Icon size | 16px | Matches text scale |

##### REQ-5.2.1.25 Entity Path Display

The breadcrumb is contextual - it reflects how the user reached the workbench, not a derived canonical path. Two top-level mode families share the same visual structure: navigation-origin (when editing a test) and execution-context (when viewing a tracked execution).

Each entity segment carries a leading entity-type icon that matches the application sidebar so users build one mental model for "what kind of thing this is":

| Entity | Icon | Color token |
|--------|------|-------------|
| Executions (list page) | `Play` | #60A5FA |
| Regression | `GitBranch` | #A78BFA |
| Suite | `FolderKanban` | #F59E0B |
| Test | `TestTubeDiagonal` (alias `TestTube2`) | #4ADE80 |

The last segment is non-interactive (it represents the current page). All preceding segments are clickable links to their entity detail page; hover affordance is the same ghost-button treatment as the canvas toolbar's Save / Edit buttons (transparent at rest, `var(--color-bg-subtle)` rounded pill on hover, 150ms transition, no underline). Separators are `/` in muted text color between segments.

Navigation-origin path formats (no tracked execution is loaded):

| Origin | Path Format | Notes |
|--------|-------------|-------|
| Draft (unsaved) | "Draft Workflow" | No back arrow - nothing to return to. |
| Tests-origin (any list page) | "{Test Name}" | Single non-link segment. |
| Suite-origin (opened from a suite detail page) | "{Suite Name} / {Test Name}" | Suite name is a link to `/suites/{id}`. |
| Regression-origin (opened from regression -> suite -> test) | "{Regression Name} / {Suite Name} / {Test Name}" | Both Regression and Suite are links. |

Execution-context path formats (the loaded workbench session is a tracked execution - the breadcrumb root becomes "Executions", which is the list page, not a specific entity):

| Execution shape | Path Format | Notes |
|-----------------|-------------|-------|
| Standalone Test execution | "Executions / {Test Name}" | "Executions" links to `/executions`. Test name is non-interactive (current page). |
| Suite child execution (a Test running as part of a Suite execution) | "Executions / {Suite Execution Name} / {Test Name}" | Middle segment links to `/executions/{parentExecutionId}`. |
| Regression child execution (Test running inside a Suite-of-a-Regression execution) | "Executions / {Regression Execution Name} / {Suite Execution Name} / {Test Name}" | Both intermediate segments are links to their respective execution detail pages. |

Origin context is propagated by the source page when navigating to the workbench. The Suite detail page appends `?suiteId=<id>` and forwards `regressionId` if present; the Regression detail page appends `?regressionId=<id>` when navigating to a suite. The workbench reconstructs the path from these query parameters plus the loaded session's execution context.

##### REQ-5.2.1.25 Back Arrow Behavior

The back arrow navigates exactly one step up the breadcrumb hierarchy as the user reads it left to right. The back-arrow target follows the leftmost interactive segment:

| Mode | Back target |
|------|-------------|
| Draft | (hidden) |
| Tests-origin | `/tests` |
| Suite-origin | `/suites/{suiteId}` |
| Regression-origin | `/suites/{suiteId}?regressionId={regressionId}` (suite detail preserves regression context; the suite's own back arrow then returns to the regression) |
| Standalone Test execution | `/executions` |
| Suite child execution | `/executions/{parent_execution_id}` |
| Regression child execution | `/executions/{suite_exec_id}` (one step up from the test exec) |

The back arrow triggers the unsaved changes dialog when the workflow is dirty before navigating, regardless of mode.

##### REQ-5.2.1.25 Overflow Handling (workbench-specific)

The bar must keep the right-side Reset + Status indicator visible at all viewport widths. When segment content exceeds the available left-half width, the breadcrumb cascades through three states:

1. **Full**: all segments render with their icons + labels. Activated when the total content width plus a comfortable buffer fits the left-half allocation.
2. **Collapsed**: ALL parent segments (regardless of count) collapse into a single actionable ellipsis button (`...`). The leaf segment renders in full. Clicking the ellipsis opens a popover listing each parent with its icon + name; each row is a clickable link to its detail page. The popover uses the same shared component as the Suite Detail collision-resolution modal's entity list.
3. **Truncated leaf**: after collapse, if the leaf still overflows, it truncates with `text-overflow: ellipsis`. On hover or keyboard focus, the full leaf name surfaces in a multiline tooltip - same shared component and visual contract as truncated name cells in list pages. The tooltip is observer-gated and appears only when the leaf actually clips.

The collapse-then-truncate cascade is preferred over per-segment shrinking because it preserves the user's most relevant context (the current entity name) and never displays a row of nested ellipses.

**Status Indicator**: The status region displays current workflow state:

| State | Display | Color | Additional Elements |
|-------|---------|-------|---------------------|
| Idle | "Idle" | Muted gray | None |
| Running | "● Running" | Blue | Pulsing dot animation |
| Paused | "● Paused" | Amber | Static dot |
| Passed | "● Passed" | Green | "Reset" action link |
| Failed | "● Failed" | Red | "Reset" action link |
| Stopped | "Stopped" | Muted gray | "Reset" action link |

**Reset Action**: The Reset button is always visible on the right side of the breadcrumb bar:

- Clears all element execution states
- Returns workflow to Idle status
- Does not affect workflow structure or connections
- Positioned left of status indicator
- **Always visible** but **disabled** when status is idle, running, or paused
- **Enabled** only after execution completes (passed, failed, stopped, skipped)

**Right Cluster**: The right side of the bar reads, left to right, Test Data chip, Artifacts chip (with a count badge whenever the session holds artifacts), Reset, then the status indicator. Both chips share Reset's gating - present on the live Workbench canvas, absent on historical and read-only canvases (TDATA-3, ARTIF-14); the chips and the sheets they open are described in 15.6.4.4 and 15.6.4.5.

#### 15.4.0.3 Canvas Empty State

When a canvas contains no user-added elements, it displays an onboarding experience that guides new users while remaining unobtrusive for experienced users.

**Empty State Components**: The empty canvas presents a structured introduction:

| Component | Position | Purpose |
|-----------|----------|---------|
| Start node | Upper-center | Entry point for execution flow |
| Ghost connector | Below start | Suggests where first element connects |
| Placeholder block | Below ghost | Drop target and hint container |
| Onboarding hint | Inside placeholder | Instructional text |

**Start Node Behavior**: The start node is an intrinsic element that cannot be removed:

- Always visible, positioned at a fixed location
- Displays "Start" label with play icon
- Has output handle for connecting to first element
- Pulses subtly when no outgoing connection exists (prompting user action)
- Not draggable - remains anchored at designated position

**Ghost Connector**: A visual guide connecting start to placeholder:

- Rendered as dotted/dashed line
- Indicates the expected connection path
- Disappears when user adds first real element
- Uses muted visual treatment to avoid distraction

**Placeholder Block**: A drop target that transforms on first element addition:

- Dashed border indicating it's not a real element
- Contains onboarding content when empty
- Accepts drops from palette
- Disappears permanently once first element is placed
- Positioned at grid-snapped location below start

**Onboarding Hint Content**: Text guidance within the placeholder:

- Brief instructional message about adding elements
- Suggests dragging from palette or using keyboard

**State Transitions**: The empty state evolves as users interact:

| Action | Result |
|--------|--------|
| Drag element to placeholder | Placeholder replaced by real element, ghost connector becomes real edge |
| Drag element elsewhere | Element placed, empty state components remain |
| Complete first connection | Ghost connector and placeholder hidden |
| Delete all elements | Empty state reappears |

**Visual Treatment**: Empty state elements use subtle styling to distinguish from real workflow:

| Element | Visual Property | Value |
|---------|-----------------|-------|
| Ghost connector | Stroke style | Dashed |
| Ghost connector | Opacity | 50% |
| Placeholder | Border style | Dashed |
| Placeholder | Background | Subtle muted tint |
| Hint text | Color | Muted foreground |

#### 15.4.0.4 Canvas Loading Experience

##### REQ-3.1.1.12 Workbench loading experience with centered canvas initialization

The canvas loading experience ensures that users see a polished, professional transition when opening the workbench - whether loading an existing test, restoring a draft, or starting a blank canvas. The experience eliminates visual flicker from intermediate states and provides clear feedback during asynchronous preparation.

**Loading Scenarios**: Three distinct scenarios require coordinated loading behavior:

| Scenario | Trigger | Data Source | Centering Strategy |
|----------|---------|-------------|-------------------|
| Existing test | Navigate to `/workbench?editTest={testId}` | API fetch | Fit view on all workflow blocks |
| Draft restoration | Navigate to `/workbench` with saved draft | localStorage | Restore saved viewport position |
| Blank canvas | Navigate to `/workbench` with no draft | None | Center on Start node and onboarding area |

##### REQ-3.1.1.12.1 Loading overlay during test fetch and canvas preparation

A loading overlay covers the canvas area during the preparation phase, providing visual feedback while hiding unready content underneath. The overlay communicates what is happening and prevents interaction with partially loaded state.

**Overlay Structure**: The overlay sits above the canvas content within the canvas panel boundaries. It does not cover the toolbar, breadcrumb bar, palette, or variable manager - only the workflow canvas area where blocks and edges render. The overlay consists of a semi-transparent backdrop with a centered loading indicator and contextual message.

**Loading Indicator**: The overlay uses the large spinner variant (40px diameter, 3px stroke) from the feedback patterns design. The spinner provides clear activity indication at a size appropriate for the canvas area. Below the spinner, a text message describes the current activity.

**Contextual Messages**: The message adapts to the loading scenario to set accurate expectations:

| Scenario | Message |
|----------|---------|
| Fetching existing test | "Loading test…" |
| Restoring draft | "Preparing workbench…" |
| Initializing blank canvas | "Preparing workbench…" |

**Overlay Visual Treatment**: The overlay uses design system tokens for consistent theming:

| Element | Property | Token/Value |
|---------|----------|-------------|
| Backdrop | Background | `--color-bg-card` at high opacity |
| Backdrop | Position | Absolute, fills canvas panel |
| Spinner | Size | Large (40px) |
| Spinner | Color | `--color-primary` |
| Message text | Font size | `text-sm` (14px) |
| Message text | Color | `--color-text-muted` |
| Message text | Spacing above spinner | `space-3` (12px) |

**Overlay Timing**: The overlay appears immediately when the canvas page mounts, before any asynchronous work begins. It remains visible throughout the entire preparation phase - data fetching, block creation, layout calculation, and viewport adjustment. The overlay only dismisses after the canvas is fully ready for interaction.

##### REQ-3.1.1.12.2 Canvas centered on Start node for blank/draft workflows

When the canvas opens with no existing test data, the viewport centers on the Start node and onboarding area to provide immediate visual orientation. Users should see the workflow entry point and instructional content without needing to pan or zoom.

**Blank Canvas Centering**: For a fresh workbench with no draft, the viewport positions to show the Start node prominently with the onboarding placeholder visible below it. The view should encompass the Start node, ghost connector, and placeholder block within the visible canvas area with comfortable padding around the group.

**Draft Canvas Viewport**: For draft restoration, the saved viewport position (pan offset and zoom level) is restored exactly as the user left it. No automatic centering or fit-view adjustment occurs, preserving the user's spatial context.

**Existing Test Centering**: For loaded tests, a fit-view operation adjusts the viewport to show all workflow blocks with appropriate padding. This ensures the user sees the complete workflow structure upon opening, regardless of how many blocks exist or their spread.

**Centering Priorities**: The centering strategy follows a clear priority order:

| Priority | Condition | Action |
|----------|-----------|--------|
| 1 | Draft exists with saved viewport | Restore exact viewport |
| 2 | Test loaded from API | Fit view on all blocks |
| 3 | Blank canvas (no draft, no test) | Center on Start node area |

##### REQ-3.1.1.12.3 Smooth transition from loading to ready state

The transition from loading overlay to revealed canvas uses a coordinated animation that feels natural and polished. The overlay fades out while the canvas content becomes visible, following the crossfade timing established in the feedback patterns design.

**Transition Timing**: The crossfade uses a 200 millisecond duration with ease-out timing. The overlay opacity decreases from full to zero while the canvas content opacity increases from zero to full simultaneously. This duration matches the skeleton-to-content transition specified in the loading indicators pattern, maintaining consistency across the application.

**Transition Trigger**: The transition initiates only after all preparation steps complete:

1. Data fetching complete (API response received or draft loaded)
2. Nodes and edges created in the canvas store
3. Viewport positioned (fit-view calculated or saved viewport restored)
4. React Flow has painted the nodes (layout stable)

Only when all four conditions are satisfied does the overlay begin its fade-out transition. This prevents any partial or intermediate state from becoming visible.

**Reduced Motion Support**: When the user has indicated a preference for reduced motion, the crossfade transition is replaced with an instant swap - overlay disappears and canvas appears without animation. Functional behavior remains identical; only the visual transition changes.

##### REQ-3.1.1.12.4 No visual flicker of intermediate canvas states

The canvas content remains completely hidden during the preparation phase to prevent any flicker of unpositioned blocks, blank canvas, or mid-layout states. Users should never see:

- An empty canvas flash before blocks appear
- Blocks at default positions before fit-view repositions them
- A brief frame of the wrong zoom level
- Partially loaded workflow structures

**Visibility Control**: The canvas content (React Flow container with nodes, edges, and background) starts in a hidden state when the page mounts. The loading overlay provides the only visual content during preparation. The canvas becomes visible only as part of the coordinated transition.

**Layout Stability**: The canvas area maintains consistent dimensions throughout the loading-to-ready transition. The overlay occupies the exact same space as the canvas content, preventing any layout shift (CLS) during the reveal. Panel boundaries (palette on left, variable manager on right, console below) remain stable and interactive during canvas loading.

**Preparation Completeness**: The readiness determination accounts for the asynchronous nature of React Flow's internal layout. After nodes are added to the store, the rendering engine requires time to measure and position DOM elements. The readiness signal fires only after the rendering engine has completed its layout pass, ensuring that fit-view calculations operate on accurate node dimensions and positions.

#### 15.4.0.5 Block Palette

The block palette serves as the primary source for workflow elements. Positioned on the left edge of the canvas, it provides organized access to plugins and control elements that users drag onto the canvas to construct workflows.

**Dual-State Design**: The palette operates in two visual modes to balance workspace efficiency with discoverability:

| State | Width | Purpose |
|-------|-------|---------|
| Collapsed | 64px | Maximizes canvas space while maintaining quick access |
| Expanded | 260px | Full navigation and search capabilities |

**Collapsed State Structure**: When collapsed, the palette presents a vertical icon strip providing rapid category access:

| Icon Position | Icon | Category | Color | Function |
|---------------|------|----------|-------|----------|
| 1 (Top) | Magnifying glass | Search | Cyan | Opens search within flyout |
| 2 | Star | Most Used | Yellow | Shows most frequently used elements |
| 3 | Clock | Recent | Cyan | Shows recently used elements |
| 4 | Git-branch | Controls | Orange | Flow control elements |
| 5 | Puzzle piece | Plugins | Violet (#A78BFA) | Execution plugins |
| 6 | Plus | New Function | Cyan | Drag onto the canvas to create a new Function |
| 7 (Bottom) | FunctionSquare | Function Lib | Indigo (#818CF8) | Opens the Functions Library modal |

**Icon Consistency Rule**: The Functions icon and color MUST match their counterpart in the navigation sidebar (LIBRARY section). This ensures users recognize the same concept across different UI contexts. Specifically:
- **Functions**: Uses `FunctionSquare` icon from lucide-react with indigo color (#818CF8 / text-indigo-400)

**Flyout Interaction**: Hovering over category icons in collapsed state triggers a popover flyout displaying that category's contents. Users can drag elements directly from the flyout without expanding the full palette.

**Expanded State Sections**: When expanded, the palette organizes content into collapsible sections:

| Section | Icon | Content | Behavior |
|---------|------|---------|----------|
| Search | - | Text input | Placeholder: "Search controls & plugins", filters all sections |
| Recently Used | Clock | Last N dragged elements | Collapsible, auto-populated from usage history |
| Most Used | Star | Frequently used elements | Collapsible, ranked by usage count |
| CONTROLS | - | Flow control elements | Collapsible, contains control items |
| PLUGINS | - | Execution plugins | Collapsible, nested subcategories |
| MISC | - | Utility/annotation elements | Collapsible, non-executable canvas tools |

**CONTROLS Section Items**: The CONTROLS section contains flow control elements:

| Item | Icon | Purpose |
|------|------|---------|
| If / Else | Branch icon | Conditional branching |
| Switch | Multi-branch icon | Multi-way branching |
| Assert | Check icon | Validation checkpoint |
| For Loop | Loop icon | Count-based iteration |
| For Each | List-loop icon | Collection iteration |
| While Loop | Repeat icon | Condition-based iteration |
| Parallel | Fork icon | Concurrent execution |
| Try / Catch | Shield icon | Error handling |
| Delay | Timer icon | Fixed time pause |
| Wait Until | Hourglass icon | Condition-based pause |
| Halt | Stop icon | Execution termination |
| Var Setter | Variable icon | Set variables in execution context |

**PLUGINS Section Structure**: The PLUGINS section organizes execution plugins into nested subcategories with item counts:

| Subcategory | Count | Items |
|-------------|-------|-------|
| Network | 3 | HTTP, Ping, Netcat |
| Remote | 8 | SSH Session Open, SSH Session Exec, SSH Session Script, SSH Session Close, SSH Tunnel Open, SSH Tunnel Close, SFTP, Remote Execution |
| Database | 2 | SQL, OpenSearch |
| System | 5 | Shell Script, Shell Exec, Python, File I/O, Log |
| Kubernetes | 12 | K8s Connect, K8s Get, K8s Events, K8s Logs, K8s Top, K8s Apply, K8s Delete, K8s Scale, K8s Rollout, K8s Exec, K8s Script, K8s Copy |
| Helm | 6 | Helm Install, Helm Upgrade, Helm Rollback, Helm Uninstall, Helm Test, Helm Status |
| Data | 4 | CSV, Excel, XML, JSON |

A subcategory with no plugins in it is not rendered - not as a collapsed header, not as an empty state. A group that can only ever say "nothing here" is a promise the palette cannot keep, and a reader who opens one stops trusting the groups that do carry blocks. The rule is general rather than a named exclusion, so a category fills in and appears by itself the moment plugins declare it.

**MISC Section Items**: The MISC section contains non-executable utility elements:

| Item | Icon | Purpose |
|------|------|---------|
| Note | StickyNote icon | Canvas annotation - sticky note with user text and color presets |

**Action Buttons**: The expanded palette includes action triggers at the bottom:

| Button | Style | Action | Purpose |
|--------|-------|--------|---------|
| + New Function | Cyan, drag handle | Dragging it onto the canvas creates an empty local Function block; clicking it shows a hint | Create local Functions |
| Function Lib | Indigo (#818CF8), text button | Opens the Functions Library modal | Browse and add published Functions |

Note: The Function Lib button uses the same indigo color as the Functions nav item and collapsed icon strip, per the Icon Consistency Rule above.

**Plugin Categorization**: Elements in the palette derive from the backend plugin registry. The palette automatically categorizes plugins by their declared type and subcategory.

**Usage Tracking**: The palette maintains user-specific usage data to populate Recent and Most Used sections. This data persists across sessions to provide personalized quick access to frequently used elements.

#### 15.4.1 React Flow Integration

##### REQ-3.1.1.1 Drag-and-drop interface for block arrangement

The drag-and-drop interface enables visual workflow construction through direct manipulation, allowing test engineers to build test sequences by dragging elements from a palette onto a canvas. This interaction model makes test creation accessible to users of all technical levels while providing the precision power users expect.

**Drag-and-Drop Model**: The interface implements a source-to-target drag model with two scenarios: palette-to-canvas drags that create new element instances, and canvas-to-canvas drags that reposition existing elements. Both share consistent visual feedback while applying context-appropriate behavior.

**Element Palette**: The palette contains plugins and control elements organized by category, displaying each item's icon and name. Dragging creates a visual clone that follows the cursor while the original remains in place. The palette supports scrolling. Plugins are the leaf executors that perform actions; control elements manage flow logic.

**Adding to a Canvas**: The controls at the bottom of the palette add Functions to the canvas; suites and regressions are ordered lists, not canvases, and add their members from their detail pages:

| Editing Context | Available Actions |
|-----------------|-------------------|
| Inside Test | Palette (plugins, control elements), "+ New Function" and "Function Lib" |
| Inside Function | Palette (plugins, control elements), "+ New Function" and "Function Lib" |
| Suite detail | "+ Add Tests" button |
| Regression detail | "Add Suites" button |

**Start Point**: Each canvas (a Test's flow, a Function's flow) has an intrinsic start point that appears automatically when the canvas is created. The start point is not draggable - it represents the entry into execution flow and must connect to the first child element.

**Canvas Drop Zone**: The workflow canvas accepts elements from the palette or repositioned from other locations. A dot grid background aids spatial orientation. Drop operations position elements at the cursor release point.

**Canvas Navigation**: The canvas supports pan and zoom for navigating larger workflows:

| Interaction | Behavior |
|-------------|----------|
| Mouse wheel | Zoom in/out centered on cursor |
| Click-drag on background | Pan canvas |
| Fit-to-screen button | Adjusts zoom to show all elements |

Pan and zoom operate smoothly during all canvas interactions including drag operations.

**Focus a Block**: The canvas exposes a single focus-a-block capability that pans and zooms to center one block in the viewport. This capability is available whether the canvas is being edited or only watched, since panning and zooming remain live even when editing is locked.

**Focus Current**: A Focus Current control on the run toolbar centers the block that is "current" right now - the running or paused block during a run. If that block lives inside a function the viewer is not currently looking at, Focus Current first navigates to that function's level, drilling in or popping out as needed, and then centers it, so the current block is always reachable in a single action regardless of nesting depth. The control appears only while a run is active and is disabled when there is no current block. Because it only moves the viewport, it stays available to a watcher mirroring another holder's session.

**Follow Changes**: While mirroring another holder's session, a viewer can turn on Follow Changes from the watch banner. The viewport then re-centers automatically on whatever block just changed - the block the assistant places during a live build, or any block the holder moves or edits - re-firing on each change, including a repeated change to the same block, and drilling into a function's interior when the changed block is nested there. The block that changed carries a one-shot highlight ring on each change. Panning does not turn following off; only toggling it off does. Following only moves the viewport and never takes control, so a watcher can track a live build without interrupting it.

**Drag Visual Feedback**: The dragged element follows the cursor with slight offset. Opacity reduction (50%) and subtle scale increase (1.02×) distinguish the preview from placed elements. The cursor changes to grabbing during drags and indicates not-allowed over invalid targets.

**Drop Target Indication**: Drop target feedback includes a highlight zone at the drop position and insertion indicators when dragging near existing elements, showing where new elements will connect.

**Placement Behavior**: Elements dropped near connection points (including the start point's output handle) automatically establish connections, supporting fluid workflow construction.

**Workflow Element Node**: Each plugin, control, or Function placed on the canvas renders as a single shared element-node component whose full visual contract lives in § 15.4.8 (Element Node Visual Structure). The card has a body region (icon + entity-type title + optional multiline description) and a persistent status bar at the bottom (badges left, action cluster right). Notes are the single explicit exception (§ 15.4.7).

**Block Selection Interaction**: Clicking a block selects it, making it the active element for editing operations. Clicking the canvas background deselects all blocks. Selection is a prerequisite for operations like delete, configure, and copy.

**Canvas Drop Handler Flow**: When a user drops an element from the palette onto the canvas:

1. Canvas receives drop event with `application/wave-block` data containing element type and metadata
2. Drop position is calculated relative to canvas viewport
3. New block is created with unique ID, position, and element type data
4. Block appears on canvas at drop location with default label
5. If dropped near start node output or another block's output handle, edge auto-connects

The drop handler validates that the drop occurred within the canvas bounds and that the element type is permitted in the current editing context.

**Element Repositioning**: Existing elements support drag-to-reposition while maintaining connections - edge paths update dynamically during movement. Multi-select enables moving groups as a unit, preserving relative positions.

**Visual Integration**: The interface derives visual properties from the design system:

| Element | Property | Token/Value |
|---------|----------|-------------|
| Canvas background | Color (light) | `#FAFAFA` (`--color-bg-card`) |
| Canvas background | Color (dark) | `#09090B` (`--color-bg-page`) |
| Grid dots | Color (light) | `#E4E4E7` (`--color-border-default`) |
| Grid dots | Color (dark) | `#27272A` (`--color-border-default`) |
| Grid | Spacing | 20px |
| Grid | Dot size | 1px |
| Drag preview | Opacity | 50% |
| Drag preview | Scale | 1.02 |
| Drop target highlight | Border style | dashed |
| Drop target highlight | Border color | `--color-primary` |
| Drop target highlight | Background | `--color-primary-muted` at 30% opacity |
| Insertion indicator | Color | `--color-primary` |
| Insertion indicator | Width | 2px |
| Invalid drop cursor | Style | not-allowed |
| Workflow block | Background | `--color-bg-card` |
| Workflow block | Border | 1px solid `--color-border-default` |
| Workflow block | Border radius | `rounded-lg` (8px) |
| Workflow block | Padding | 12px |
| Workflow block | Min width | 120px |
| Workflow block | Icon size | 20px |
| Workflow block | Label size | `text-sm` (14px) |

##### REQ-3.1.1.2 Sequential flow connections between blocks

Sequential flow connections define execution order within the WAVE hierarchy. Every canvas (a Test's flow, a Function's flow) has a start point that must connect to child elements to establish the execution path. Elements without incoming connections are inactive and ignored during execution.

**Execution Hierarchy**: WAVE uses a nested container model. Suites and regressions are ordered lists, and connections determine flow on a canvas:

| Container | Contains | Order Set By |
|-----------|----------|-------------------|
| Regression | Suites | List order |
| Suite | Tests | List order |
| Test | Blocks | Canvas connections |
| Function | Blocks | Canvas connections, within the Function's scope |

Plugins are leaf elements - the actual executors that perform actions. Functions serve as scope managers that can contain other Functions or plugins, providing variable isolation and logical grouping.

**Start Point Model**: Each canvas displays a start point - a distinct visual element representing the entry into that canvas's execution flow. The start point must connect to the first child element to activate execution. Without this connection, no child elements execute.

**Connection Semantics**: Connections define what executes and in what order:
- Connected elements execute in the sequence defined by the flow
- Elements with no incoming connection are orphaned and completely ignored
- The flow `A→B→C` means A executes first, then B, then C

**Edge Creation Interaction**: Users create connections by dragging from an output handle (bottom of element or start point) to an input handle (top of target element). A temporary edge follows the cursor during the drag. Valid drop targets highlight when the cursor approaches. Releasing over a valid input completes the connection; releasing elsewhere cancels it.

**Connection Feedback**: During edge creation, visual feedback guides the user:
- Temporary edge renders as a dashed line following the cursor
- Valid target handles display a highlight glow effect
- Invalid targets (self-connection, duplicate edge) show not-allowed cursor
- The start point pulses subtly when no connections exist (prompting user action)

**Auto-Connection Proximity Feedback**: When dragging a new element from the palette or repositioning an existing element that has no incoming connection, the system provides real-time visual feedback indicating that releasing will auto-connect to a nearby element:

| Condition | Visual Feedback |
|-----------|----------------|
| Drag within proximity threshold of a free output | Ghost edge appears from source output to dragged element's input position |
| Ghost edge | Dashed stroke, neutral gray at 60% opacity |
| Source output handle | Glow ring in `--color-success` with pulse animation |
| Drag leaves proximity threshold | Ghost edge and handle glow disappear immediately |
| Drop within threshold | Ghost edge transitions to solid real edge |

Proximity detection uses a distance threshold from the source element's output handle center to the dragged element's position. Only elements with a free output handle (no existing outgoing edge) participate as proximity sources. Elements that already have an incoming connection do not trigger proximity feedback when repositioned.

**Edge-Scroll During a Connection Drag**: Holding a connection drag at the canvas edge scrolls the canvas, so a target outside the visible area can be reached without releasing. The scroll rate is deliberately low: a block the user is aiming at must stay inside the ingress trigger range long enough to correct, otherwise its ingress affordance disappears mid-gesture and the release silently does nothing. This makes blocks sitting near the canvas edge targetable at the cost of a longer hold to reach genuinely off-screen ones.

**Edge Rendering**: Established edges render as smooth bezier curves. The path calculation creates natural-looking curves that flow top-to-bottom, matching the execution direction. An arrow marker at the target end indicates flow direction.

**Edge States**: Edges reflect workflow and execution status:

| State | Visual Treatment | Usage |
|-------|-----------------|-------|
| Default | Solid stroke, muted color | Normal connection |
| Selected | Primary color, thicker stroke | User-selected edge |
| Hover | Lightened stroke | Cursor over edge |
| Executing | Animated dash pattern | Active flow during execution |

**Orphan Indication**: Elements without incoming connections display a visual indicator showing they are inactive. This helps users identify incomplete workflow definitions where elements exist but won't execute.

**Edge Updates During Drag**: When elements are repositioned, connected edges update in real-time. The bezier path recalculates smoothly, maintaining visual continuity during canvas interactions.

**Visual Integration**: Edge and handle styling derives from the design system:

| Element | Property | Token/Value |
|---------|----------|-------------|
| Edge stroke | Color (default) | `--color-border-strong` |
| Edge stroke | Color (selected) | `--color-primary` |
| Edge stroke | Width | 2px |
| Edge path | Style | Bezier curves |
| Arrow marker | Size | 8px filled triangle |
| Executing animation | Pattern | Dashed, moving offset |
| Executing animation | Duration | 1000ms |
| Temporary edge | Style | Dashed stroke |
| Valid target highlight | Color | `--color-primary` at 50% opacity |
| Valid target highlight | Border | 2px solid `--color-primary` |

**Connection Handles**: Input and output handles use a pill/capsule shape for improved click targets and visual clarity:

| Element | Property | Token/Value |
|---------|----------|-------------|
| Handle | Shape | Rounded pill (capsule) |
| Handle | Size | 16px wide × 8px tall |
| Handle (regular) | Color | Solid neutral gray (no alpha - dark theme compatibility) |
| Handle (start node) | Color | `--color-success` (green, matches start point color scheme) |
| Handle | Border | None |
| Handle (hover) | Ring | 2px ring outline in neutral gray (regular) or green (start node) |
| Output handle | Icon | Downward chevron (▾) indicating flow direction |
| Input handle | Icon | None (plain pill) |
| Input handle | Position | Top center, 50% translated up |
| Output handle | Position | Bottom center, 50% translated down |

**Start Point Visual**: The start point is visually distinct from regular elements:

| Element | Property | Token/Value |
|---------|----------|-------------|
| Start point | Shape | Rounded rectangle with play icon |
| Start point | Background | `--color-success-muted` |
| Start point | Border | 2px solid `--color-success` |
| Start point | Label | "Start" text |
| Start point (no connection) | Animation | Subtle pulse |

**Orphan Element Visual**: Inactive elements display their orphan state:

| Element | Property | Token/Value |
|---------|----------|-------------|
| Orphan indicator | Opacity | 50% |
| Orphan indicator | Border style | Dashed |
| Orphan indicator | Badge | "Inactive" label |

**Validation Rules**: The connection system enforces sequential flow constraints at connection time. The connection validator rejects invalid connections before they are created:

| Rule | Validation | Rejection Behavior |
|------|-----------|-------------------|
| Multiple ingress | Each input can receive multiple incoming connections | Enables branch convergence (e.g., If/Else branches merging downstream) |
| Single egress per handle | Each output handle sends exactly one outgoing connection | Source handle does not highlight; edge preview snaps back |
| No self-connections | Element cannot connect to itself | Not-allowed cursor shown |
| No duplicates | Same source-target pair cannot have multiple edges | Not-allowed cursor shown |

- Start point has one output that connects to the first element
- Branching controls (If/Else, Assert split mode) have multiple named output handles, each allowing one outgoing edge
- Non-branching elements have a single output handle and follow the single-egress constraint
- Branch convergence is achieved naturally via multiple incoming edges on a downstream block

##### REQ-3.1.1.3 Parallel branch creation and merging

Parallel branches are created on the Parallel control, which carries one lane per branch plus an always-present exit lane. The lanes are the fork; the exit lane is the merge. Because the control owns both, a concurrent region is visible on the canvas as a bounded shape rather than inferred from where edges happen to converge.

**Creating a branch**: Branches are an ordered list on the control's configuration sheet - a row per branch carrying its name, its optional time limit, reorder controls and delete. Adding a row adds a lane; removing one removes its lane and the edge that used it. Branch order in the list is lane order on the block, so the canvas and the list never disagree. A section holds at least two branches, so the last two rows cannot be deleted.

**Wiring a branch**: Each branch lane accepts one outgoing edge, the head of that branch's sub-flow, following the same single-egress-per-handle rule as every other lane in the builder. The sub-flow is authored exactly like any other stretch of canvas and dead-ends at its last block - branch bodies do not connect onward to blocks after the section.

**Merging**: The exit lane carries the flow out of the section. It is present whether or not the branches are wired, so a Parallel can never dead-end a workflow. Convergence by pointing several branch tails at one downstream block is not how a parallel section merges; the exit lane is.

**Block presentation**: Lane color distinguishes roles: branch lanes take a color reserved for concurrent sub-flows, the exit keeps the container-exit neutral. Lanes carry no text labels - each lane's name is its hover tooltip - and the block widens by one lane width per branch, as the other multi-lane controls do. During a run, several branch lanes and several blocks inside them carry live status at once; the block itself shows no branch counter or countdown.

**Disabled**: Disabling a Parallel cascades skipped through its branch bodies and passes the flow through the exit lane, matching the pass-through behavior of the other container controls.

##### REQ-3.1.1.4 Nested block composition (blocks containing blocks)

##### REQ-3.1.1.5 Block parameter configuration forms

Block parameter configuration provides a floating-sheet interface for editing a Block's fields - a plugin's or control's settings, a Function Block's declared inputs - and its output variable assignments. This capability enables users to customize plugin behavior and establish data flow between workflow elements.

**Configuration Sheet Purpose**: The configuration surface is a non-blocking draggable sheet - the shared floating-panel primitive also used by the Functions Library - rather than a screen-blocking modal. There is no backdrop and no focus trap: the canvas behind stays fully interactive while a sheet is open, so users can pan, select, and compare against the workflow mid-edit. The sheet serves as the primary interface for configuring any workflow element (plugins, controls, and Function instances).

**Sheet Activation**: A configuration sheet opens when users click the gear icon (⚙) in the action row of a selected element or double-click the element. If the element's sheet is already open, activation focuses the existing sheet and brings it to front instead of opening a duplicate.

**Multiple Open Sheets**: Every element opens its own sheet, and several sheets can be open simultaneously - editing two elements side by side is a supported workflow. The first sheet opens viewport-centered; each additional sheet cascades 28px right and down from the previous so headers stay reachable. Clicking anywhere on a sheet brings it to front. At most 10 sheets can be open; attempting to open an 11th is refused with an explanatory toast. The Functions Library sheet can be open alongside config sheets - the "From library" link inside a Function instance sheet opens or focuses the library, pre-filtered to the source entry, without closing the config sheet.

**Sheet Lifecycle**: Because the canvas stays interactive, sheets follow the visibility of their element. A sheet counts the editors open from it as part of itself: unsaved text in an open value editor makes the sheet unsaved for every prompt below, its Save writes those editors' values into the sheet before saving, and closing the sheet closes them with it (see Editing surfaces and the floating editors opened from them). Deleting an element whose sheet has unsaved edits asks first, with Keep Editing to cancel the delete and Close to continue it; elements with clean sheets close silently. Element removal caused by undo/redo closes the sheet without confirmation - history operations are never vetoed. Sheets survive drill navigation within the same test (an outer element's configuration can remain open while the user is drilled inside a Function); a sheet whose element is outside the active scope renders read-only - an indigo READ-ONLY pill in the header, frozen fields, and a Close-only footer, with the fields of any sheet opened from it frozen as well - and re-enables when the user returns to its scope. A scope change resets a sheet's in-progress form edits, so any user-initiated drill navigation (dig-in, the kebab Open action, breadcrumb pops) while a sheet holds unsaved edits asks for confirmation first; canceling keeps the current scope with the edits intact. Execution-driven scope changes (such as the debugger stepping into a Function) never prompt. Switching to a different test or opening a library Function view closes all sheets, prompting once per sheet that holds unsaved edits.

**Modal Structure**: The configuration interface organizes information into distinct regions:

| Region | Position | Content | Purpose |
|--------|----------|---------|---------|
| Sheet header | Top | Element icon + "&lt;Element name&gt; - Edit Config" title (left), close button (right) | Drag handle for repositioning; identifies which open sheet belongs to which element; X closes through the unsaved-changes guard |
| Name/version row | Below header | Element name input (left), version dropdown (right) | Identification and version selection |
| Input section | Middle | Parameter table | Configure input values |
| Output section | Below inputs | Variable assignment table | Define output bindings |
| Footer | Bottom | Changelog/Details (left), Cancel/Save (right) | Documentation and confirmation |

**Version Management**: The header displays a version dropdown (format: "2.18 ▼") allowing users to select which version to use. Footer buttons provide access to changelog and detailed documentation.

**Parameter Configuration**: Input fields display in a structured table. Each row shows the field name (with required indicator), its type, and its value. Every Block keeps, edits, checks and runs its fields one way: a Function Block's declared inputs are its fields exactly as a plugin's settings are. Plugin and Function Block rows share one value field, the variable window's value field, followed by two fixed icon columns: expand (two diagonal arrows) and the expression toggle {x} (FIELD-1). Expand opens the field's full editor, or the expression editor in {x} mode. Icons that belong to the value sit inside the field: a clock (a time value) or a lock (a secret) at the left, the reset or the null icon at the right. A masked secret shows dots and is replaced rather than shown. What the field holds and what expand opens follow the field's type:

| Field Type | In the Field | Expand Opens |
|------------|--------------|--------------|
| String | The text on one line; a long value, or one with line breaks, is cut with "…" | The shared "Edit value" sheet's string body: a Kind control (Literal / Template), format auto-detection (JSON/YAML/XML/Plain), syntax highlighting, formatting/pretty-print, YAML and XML lint, and cross-format conversion. The Template kind renders `{{expression}}` variable references at runtime, with autocomplete and validation; literal, template, and expression are mutually exclusive per field |
| Integer | Numeric input that keeps every digit; accepts whole numbers (including negatives), no spinner controls | Nothing: no expand |
| Float | Numeric input that keeps every digit; accepts integers and decimals (including negatives), no spinner controls | Nothing: no expand |
| Boolean | Checkbox that always holds true or false | Nothing: no expand |
| Choice list | Static dropdown of the declared allowed values that always holds one of them | Nothing: no expand |
| Array | The list written out as one line of text that can be typed into | The shared "Edit value" sheet with add/remove elements, a Type picker per item, and visual/code mode toggle |
| Map | The map written out as one line of text that can be typed into | The shared "Edit value" sheet with add/remove rows, duplicate key detection, and visual/code mode toggle |
| Table | The table written out as one line | Structured tabular data editor via the shared "Edit value" sheet (no table parameters currently declared) |

Fields with their own editors are the same one-line field, and expand opens their own editor: a time value opens its picker, which ends with Cancel and Save (FIELD-9); Cipher Suites, Multipart Parts, the Shell and Python Arguments, the SQL Out arguments, and OpenSearch Fields and Stats are one-line list fields (FIELD-10).

**Field States**: A field the user has not set is not set. A free-entry field (text, number, list, map, table, code) that is not set shows its default in gray, as real text that can be selected and copied; typing or pasting replaces the whole default, Delete or Backspace shows a tooltip saying the default is used, and select all + Delete takes a field back to not set. Expand opens the editor with the default filled in, and only a change there makes it the user's value. Save stores only what the user set, and the run fills every field that is not set from its default as it stands when the run starts, so a default is never written into a Block. A reset icon inside the field takes a field that has a default and holds the user's value back to not set (FIELD-2, FIELD-3). Empty text shows `<empty string>` and a null shows `null`, both in italic gray; the null icon inside the field sets a whole null only where the field allows one (FIELD-4, FIELD-5). A list or map holding {x} entries draws each entry in its own look, an {x} entry as `{{ expr }}` in the expression color, and is changed only through expand (FIELD-6). Choice lists and true / false are different: they always hold a real value, written when the Block is created, in normal color with no reset; a Settings change reaches only Blocks created after it, and duplicate and paste copy the Block's own value (FIELD-8).

Parameter types align with WAVE's canonical 7-type system (string, integer, float, boolean, array, map, table) used across the variable manager, profile schemas, and backend plugin definitions. Complex types like `map` and `array` reuse the shared "Edit value" sheet (the type-aware value bodies described in component-specification Variable Type Editors), which provides both a structured visual mode and a raw code mode with bidirectional synchronization. Map parameters store values as string-keyed dictionaries, matching backend expectations for parameters like HTTP headers and query parameters.

**Parameter Grouping**: Parameters may declare a `group` field to organize related settings into collapsible sections. Ungrouped parameters render at the top of the table without a section header. Named groups render as collapsible sections with a header row containing the group name and a chevron indicator; a group is collapsed unless the block declares it open, which a block does for the groups holding its primary settings. This keeps the primary configuration surface clean while providing access to advanced settings (timeouts, retries, security, connection options) on demand.

**Conditional Parameter Visibility**: Parameters may declare visibility conditions based on other parameter values. When a parameter specifies a dependency, the parameter table evaluates the condition against the current form state and hides or shows the row accordingly. A hidden field stores nothing at Save, so a stale value never reaches the run; when it shows again it starts not set, and a hidden choice list starts on its default, written at that moment (SHELL-10, FIELD-8).

This enables context-sensitive configuration forms where parameters appear only when relevant - for example, a request body field that is visible only when the HTTP method supports a body (POST, PUT, PATCH, DELETE), or a destination path field that appears only when the file operation is copy or move.

**Expression Mode**: Every field WAVE evaluates at run has the expression toggle {x}, which switches between literal input and expression mode. In expression mode, the field value is a pure Python expression evaluated at runtime, and a multi-line expression keeps its line breaks. The expression editor (CodeMirror-based) provides syntax highlighting for the full expression and autocomplete suggestions for available variables and functions. For string composition, users write Python f-strings (e.g., `f"http://{host}:{port}/api"`). Enumeration fields support the same toggle to switch between fixed dropdown selection and expression mode. A field in expression mode that holds nothing is not set: the box shows the default as a gray hint written as a Python literal, and the run uses the default. Switching modes never carries a value across. When the field holds a value, switching asks first, in a dialog that never names the field, with Cancel, Clear & Default and, for text, list, map and table, Clear & Empty, which in expression mode puts in the empty value as an expression (FIELD-7).

**Output Variable Binding**: The output section allows users to assign element outputs to workflow variables. Each output can be bound to a variable name, making that value available to downstream elements; a binding that names a variable that does not exist yet creates it with the output's type, holding null until the block runs. Output bindings establish the data flow connections between elements in the workflow. Duplicate and copy/paste operations intentionally clear output bindings on the new element - each block instance must have its bindings explicitly configured to avoid multiple blocks silently writing to the same variables.

**Description Toggle**: A toggle control shows or hides parameter descriptions. When shown, description text appears below each parameter row. This allows users to balance information density with the need for documentation.

**Validation Feedback**: Required parameters display validation errors inline when empty or invalid. The save action is disabled until all required fields have valid values. Type mismatches and constraint violations show contextual error messages.

**Sheet Positioning and Resize**: The sheet opens viewport-centered (cascading when other sheets are already open) and is draggable by its header anywhere on screen; dragging clamps to the viewport so the header and close button always stay reachable. A thin right-edge resize handle adjusts the sheet width between 520 and 1200 pixels - useful when editing wide parameter tables. Position and width are per-sheet.

**Sheets Opened from a Sheet**: Every floating sheet is its own window. A sheet opened from inside a configuration sheet - the "Edit value" sheet, the multipart Parts editor, the TLS cipher picker, the file picker, the Profiles binding sheet - opens, drags, resizes, maximizes and clamps against the viewport, never against the sheet it came from. Dragging the configuration sheet leaves those sheets where they are, and each one can be dragged anywhere on screen, including clear of the sheet that opened it. They join the same front-most ordering as every other sheet: clicking one brings it to the front, and Escape closes the front-most sheet only. Pressing the control that opened one while it is still open - a field's edit control, Browse, Profiles - brings that sheet back to the front and pulses it once, as reopening a block's configuration sheet does; it never opens a second copy. Keys and clicks inside such a sheet act on that sheet alone; they never save, close or drag the configuration sheet behind it. A sheet belongs to the page it was opened on, so when that page is blocked by a failed load the sheet is blocked with it; a sheet opened from inside a modal dialog stays within that dialog.

**Accessibility**: The sheet is a non-modal panel: there is no focus trap, and clicking outside does not dismiss it (preventing accidental loss of in-progress edits). Escape closes the front-most sheet, routing through the unsaved-changes guard. The header exposes a descriptive ARIA label, tab navigation moves through form fields in logical order, and screen readers announce parameter names, types, and current values. Keyboard users reach the close button in the header tab order.

**List and Map Entries**: List items and map entries are edited in the shared editor, every row at one height; text, List, Map and {x} entries open their own editor, a List or Map entry the same editor one level deeper, up to 10 levels. A change that would drop an entry's value (its type or its {x} mode) asks first, and every {x} button in WAVE is one shared toggle.

#### 15.4.1.5 Element Selection Visual Feedback

Element selection provides visual feedback to identify which workflow element is currently active for editing operations. The selection model supports single-element selection with clear visual distinction from other elements.

**Selection Activation**: Clicking on any workflow element (a plugin, control or Function block) selects it, making it the active element for editing operations. Clicking the canvas background deselects all elements. The selection state is mutually exclusive - selecting one element deselects any previously selected element.

**Multi-Selection**: Ctrl-click (Cmd-click on macOS) adds elements to the current selection without deselecting others. Multi-selection enables bulk operations like delete or move on groups of elements.

**Selection Visual Treatment**: Selected elements display a distinctive halo effect to differentiate them from unselected elements:

| Visual Property | Value | Purpose |
|-----------------|-------|---------|
| Outline style | Dashed | Distinguishes from execution state borders |
| Outline width | 2px | Visible without overwhelming |
| Outline offset | 4px | Creates halo effect around element |
| Outline color | White (dark theme), border-strong (light) | Maintains visibility on both themes |

**Persistent status bar (not a selection affordance)**: Every block (plugin, control, Function, Note) carries a persistent status bar at the bottom of the card. The bar is always visible and never appears or disappears on selection. The bar's badge cluster (left) and action cluster (right) - including the cog (Configure), kebab (context menu), version-update arrow, and Function dig-in - are described in § 15.4.8. Selection does not change the bar's contents or its visibility.

**Connection Handle Visibility**: Selection affects connection handle display:
- Unselected elements: Handles hidden or minimized to reduce visual noise
- Selected elements: Input and output handles prominently displayed
- Hover state: Handles become visible on mouse hover for edge creation

**Context Menus**: Right-click context menus are available on three canvas targets:

| Target | Menu Content | Trigger |
|--------|-------------|---------|
| Element (block) | Same actions as kebab (⋮) menu - Edit, State, Delete groups | Right-click on block |
| Canvas (empty area) | Paste, Select All, Fit View, Reset | Right-click on background |
| Edge (connection) | Delete | Right-click on edge |

The element context menu mirrors the kebab menu structure. Actions that have corresponding keyboard shortcuts display the shortcut label right-aligned in muted text, adapting to the user's platform (Ctrl on Windows/Linux, ⌘ on macOS):

| Group | Actions | Shortcut Label | Purpose |
|-------|---------|---------------|---------|
| Execution | Run, Debug, Run from Here, Debug from Here | - | Direct execution controls |
| Edit | Duplicate, Copy, Cut | ⌘D/Ctrl+D, ⌘C/Ctrl+C, ⌘X/Ctrl+X | Clipboard operations |
| State | Toggle Breakpoint, Disable, Skip | - | Execution modifiers |
| Delete | Delete (destructive) | Del | Element removal |

The canvas context menu also displays shortcut labels:

| Action | Shortcut Label | Purpose |
|--------|---------------|---------|
| Paste | ⌘V / Ctrl+V | Clipboard paste |
| Select All | ⌘A / Ctrl+A | Select all elements |
| Fit View | - | Fit canvas to viewport |
| Unskip All | - | Remove skip from all blocks |
| Enable All | - | Enable all disabled blocks |
| Reset | - | Clear execution state |

The edge context menu provides edge type selection and Delete:

| Action | Indicator | Shortcut Label |
|--------|-----------|---------------|
| Bezier | Check icon if active | - |
| Smoothstep | Check icon if active | - |
| Step | Check icon if active | - |
| Straight | Check icon if active | - |
| *(separator)* | - | - |
| Delete | - | Del |

The default edge type is Bezier (maps to React Flow's `default` type). Edges with no type or `"default"` type show Bezier as active.

**Element Disable State**: Disabling is a structural, design-time decision. Disabled elements remain on the canvas but are excluded from execution. Disabled state persists with the workflow across saves.

| Visual Property | Value |
|-----------------|-------|
| Opacity | 50% |
| Border | Dashed, muted/neutral color |

**Element Skip State**: Skipping is a runtime, transient modifier. Skipped elements are bypassed during execution but remain structurally active. Skip state does not persist - it clears on save or reload.

| Visual Property | Value |
|-----------------|-------|
| Opacity | 100% |
| Border | Dashed, muted/neutral color |

Both states share dashed borders but differ in permanence and opacity. Disable signals structural exclusion; Skip signals temporary bypass.

**Selection During Execution**: During workflow execution (running or paused states), selection remains functional but with restricted capabilities. Users can select elements to view their state but cannot modify them. The action row adapts to show only applicable actions for the current execution state.

**Keyboard Selection**: Keyboard users navigate between elements using Tab key. Enter key selects the focused element. Escape key deselects all elements.

**Selection State Persistence**: Selection state is ephemeral - it does not persist across page navigation or browser refresh. Selection is purely for in-session editing operations.

**Edge Selection**: Edges (connections between elements) support selection independently from block selection. Clicking an edge selects it; clicking the canvas background deselects. Ctrl-click adds edges to the selection. Edge selection and block selection are mutually exclusive - selecting an edge deselects all blocks and vice versa. This prevents ambiguous behavior when pressing Delete.

| Edge State | Visual Treatment |
|------------|-----------------|
| Default | Solid stroke, `--color-border-strong`, 2px width |
| Selected | Neutral gray (zinc-400) stroke, 3px width |
| Hover | Slightly lighter stroke, pointer cursor |
| Executing | Animated dash pattern |

**Edge Keyboard Interaction**: Canvas-level keyboard shortcuts enable efficient edge and element management. Keyboard events only fire when the canvas area has focus (not during modal, input, or search interactions):

| Key | Selection State | Behavior |
|-----|----------------|----------|
| Delete | Edge(s) selected | Remove selected edges |
| Delete | Block(s) selected | Remove selected blocks and their connected edges |
| Delete | Mixed selection | Remove all selected edges and blocks |
| Backspace | Edge(s) selected | Remove selected edges (same as Delete for edges) |
| Backspace | Block(s) selected | Disconnect: remove all edges connected to selected blocks without deleting the blocks |

**Canvas Context Menu Reset**: The canvas right-click context menu includes a Reset option that clears execution state, returning the workflow to idle status without modifying workflow structure. Reset follows the same availability rules as the breadcrumb bar Reset button:

| Execution Status | Reset Availability |
|-----------------|-------------------|
| Idle | Disabled |
| Running | Disabled |
| Paused | Disabled |
| Passed | Enabled |
| Failed | Enabled |
| Stopped | Enabled |

Activating Reset from the context menu shows the same confirmation dialog as the breadcrumb bar Reset. After confirmation, all element execution states clear and the workflow returns to idle status.

#### 15.4.1.6 Canvas Keyboard Shortcuts and Mouse Interactions

##### REQ-4.2.5 Keyboard shortcuts for common operations

The canvas provides keyboard shortcuts for all common workflow editing operations, enabling power users to work efficiently without relying on context menus or toolbar buttons. Shortcuts follow platform conventions (Ctrl on Windows/Linux, Cmd on macOS) and are discoverable through an expandable hint bar.

**Keyboard Shortcut Registry**: The canvas responds to the following keyboard shortcuts when the canvas area has focus. Shortcuts are suppressed when focus is inside text input fields (INPUT, TEXTAREA, SELECT, contenteditable) or when a modal dialog is open.

| Shortcut | Action | Behavior |
|----------|--------|----------|
| Delete | Delete selected | Remove selected blocks (with connected edges) and selected edges |
| Backspace | Disconnect selected | If blocks selected: remove their edges without deleting blocks. If edges selected: remove edges |
| Ctrl+C | Copy | Copy selected blocks and their internal edges to clipboard |
| Ctrl+X | Cut | Copy selected to clipboard, then remove from canvas (single-use paste) |
| Ctrl+V | Paste | Place clipboard contents on canvas with positional offset |
| Ctrl+D | Duplicate | Duplicate selected blocks in place with offset (does not use clipboard) |
| Ctrl+A | Select all | Select all blocks and edges on canvas |
| Ctrl+Z | Undo | Revert last canvas operation from history |
| Ctrl+Shift+Z | Redo | Re-apply last undone operation |
| Ctrl+Y | Redo | Alternative redo shortcut (Windows convention) |
| Ctrl+Space | Quick add | Open the Quick Insert window (see "Quick Insert" below) |
| Enter | Configure selected | Open the config sheet of the single selected block - the same surface double-click opens |
| Ctrl+Enter | Open selected Function | Drill into the inner canvas of the single selected Function block, Local or Shared alike |

**Selected-Block Keys**: Enter and Ctrl+Enter act on a selection of exactly one block; with no selection or a multi-selection they do nothing, so a key press can never fan out across blocks. Enter reaches the same config surface as double-click, including the open-sheet cap and the raise-and-pulse behavior when that block's sheet is already open. Ctrl+Enter drills into any Function block: a Shared instance already offers Open in its context menu, so the keyboard grants no capability the menu withholds. The Start block and Note blocks have no config surface and ignore both keys.

**Quick Insert**: Ctrl+Space opens a keyboard-driven insert surface over the canvas area - a movable, non-blocking window with no backdrop, opening top-center so it never covers the viewport center where centered insertion lands. Its header is a drag handle carrying the title and a close control; ESC and the close control dismiss it, and a position the author drags it to is remembered for the session. The window consists of a search input, a grouped result list (PLUGINS / CONTROLS / FUNCTIONS), and a footer of key hints. The catalog has full palette parity: every plugin, every control, every published library Function (rows carry the entry's version badge; inserting materializes a Shared instance pinned to that version), and a "New Function" creator row that inserts an empty Local Function.

Search matches the typed text as a substring of a block's name, the same rule the Tools Palette search applies, so the two surfaces cannot disagree about what a query means. Ranking beyond that is not applied: results keep catalog order within their group.

The window owns the keyboard for as long as it is open. The text caret stays in the search input no matter where inside the window the author clicks, and returns there when the author clicks away to the canvas and back, so navigation keys and typing are never silently dead. The first result is highlighted on open and after every query change.

| Key | Effect |
|-----|--------|
| ArrowDown / ArrowUp | Move the highlight one row |
| Tab / Shift+Tab | Move the highlight one row - Tab is a synonym for ArrowDown, not a focus change |
| PageDown / PageUp | Move the highlight one visible page |
| Enter | Insert where the flow is (see below) |
| Ctrl+Enter | Insert at the viewport center, unwired |
| ESC | Close |

Enter resolves its anchor from the canvas rather than from a separate key: the single selected block when there is one, otherwise the last block the author interacted with, otherwise the end of the flow - which on a clean root canvas or an empty Function body is the Start block. The new block is linked from the anchor's first free egress; when the anchor has no free egress it is placed without a link. The footer hint names the anchor in force, so the key's promise stays literally true as the selection changes.

"Last interacted" is the most recently selected or inserted block within the active (drilled) scope. The placed element always becomes the selection - replacing any prior selection - and the new "last interacted" block, so repeated Ctrl+Space, Enter sequences chain a workflow downward without touching the mouse. Result rows are also drag sources: dragging a row onto the canvas follows the standard palette drop path, including proximity auto-connect; the footer names that affordance with a pointer badge rather than a key badge.

Quick Insert reuses the drop path's behavior wholesale - identical auto-naming, default configurations, published-Function self-nesting rejection (same toast), and palette usage tracking for plugins and controls. The shortcut is unavailable in read-only canvases, while a workflow is executing, and while focus is inside a text input.

**Focus Context Guards**: Keyboard shortcuts activate only when the canvas container has focus and no overlay is capturing input. The guard logic checks the active element's tag name and contenteditable attribute. During execution (running or paused state), destructive and history shortcuts (Delete, Cut, Undo, Redo) are suppressed to prevent accidental modification of an active workflow; the suppression is enforced at the state layer so shortcuts and toolbar buttons always agree.

**Clipboard Behavior**: Copy creates a reusable clipboard snapshot - paste can be repeated multiple times. Cut creates a single-use clipboard - after one paste, the clipboard clears. Each paste offsets the pasted elements from the previous paste position to avoid exact overlap. The clipboard is internal to the canvas session and does not interact with the system clipboard. Both copy/paste and duplicate operations clear output bindings on the resulting elements - each block instance must have its output bindings explicitly configured to prevent multiple blocks writing to the same variables.

**Rubber-Band Box Selection**: Holding Shift while dragging on the canvas background activates a selection rectangle (rubber-band). All blocks whose bounding boxes intersect the rectangle are added to the current selection when the drag completes. Rubber-band selection is always additive - previously selected blocks remain selected. This leverages the canvas library's native selection behavior with Shift as the activation modifier. Regular drag (without Shift) continues to pan the canvas.

**Shift+Click Chain Selection**: When blocks are already selected, Shift+clicking on a connected block selects the shortest path (by edge hops) between the clicked block and the nearest already-selected block. The algorithm performs BFS on the undirected edge graph starting from the clicked block, finds the closest selected block, and reconstructs the path - all intermediate blocks along that path are added to the selection. Chain selection is additive: previously selected blocks remain selected. Shift+click is a no-op when: (a) no blocks are currently selected, (b) the clicked block has no path to any selected block through the edge graph, or (c) clicking the empty canvas background. The Start node is excluded from chain selection targeting.

| Interaction | Modifier | Behavior |
|-------------|----------|----------|
| Drag on canvas | None | Pan canvas |
| Shift+drag on canvas | Shift | Draw selection rectangle, add intersecting blocks to selection |
| Shift+click block (has selection) | Shift | Select shortest chain from clicked block to nearest selected block |
| Shift+click block (no selection) | Shift | No-op (no selected block to chain from) |
| Shift+click canvas | Shift | No-op |
| Click block | None | Select block (deselect others) |
| Ctrl+click block | Ctrl/Cmd | Toggle block in selection (additive) |
| Click canvas background | None | Deselect all |

**Interactions Hint Popover**: A floating popover triggered by a prominent circle button showing all available keyboard shortcuts and mouse interactions. The popover presents shortcuts as a vertical list with section dividers, making it easy to scan without consuming canvas space.

| Property | Value |
|----------|-------|
| Trigger | Circle button with `?` icon, bottom-right of canvas (near React Flow zoom controls) |
| Behavior | Click to open popover, click outside or click `?` again to close |
| Layout | Vertical list - label left-aligned, key badge(s) right-aligned |
| Sections | Divided by horizontal rules with uppercase section headers |
| Platform | macOS shows ⌘ instead of Ctrl |

The popover organizes shortcuts into six sections:

| Section | Shortcut | Keys |
|---------|----------|------|
| **Editing** | Copy | Ctrl+C |
| | Cut | Ctrl+X |
| | Paste | Ctrl+V |
| | Duplicate | Ctrl+D |
| | Delete | Del |
| | Unlink | Backspace |
| **Selection** | Select | Click |
| | Multi-select | Ctrl+Click |
| | Box Select | Shift+Drag |
| | Chain Select | Shift+Click |
| | Select All | Ctrl+A |
| **Navigation** | Zoom | Scroll |
| | Pan | Drag |
| **History** | Undo | Ctrl+Z |
| | Redo | Ctrl+Shift+Z |
| **Blocks** | Configure | Enter |
| | Open Function | Ctrl+Enter |
| **Insertion** | Quick Insert | Ctrl+Space |
| | Add where the flow is | Enter |
| | Add at center | Ctrl+Enter |

#### 15.4.3 Test Persistence

##### REQ-3.2.1.1 Save workflow as test

Test persistence enables users to save constructed workflows as named test entities. The save operation transforms the visual canvas representation into the backend Test model and persists it through the Tests API.

**Save Dialog Trigger**: The save action initiates from the toolbar Save button. When the canvas has unsaved changes (dirty state), clicking Save opens the Save Test Dialog. For existing tests, Save performs an update operation without showing the dialog.

**Save Test Dialog**: The dialog collects metadata required for test creation:

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| Test name | Text input | Yes | 1-255 characters, alphanumeric with spaces/hyphens/underscores |
| Description | Textarea | No | Free-form text, optional |
| Tags | Free-form chip input | No | User types tags, displayed as removable chips |
| Suite assignment | Searchable dropdown | No | Searchable popup with suite list, fetches via GET /api/v1/suites |

**Tags Input**: The tags field uses a free-form chip input pattern:

- User types tag text and presses Enter/comma to add
- Each tag displays as a removable chip
- Clicking chip × removes the tag
- Tags are accepted as typed: no server-side validation and no autocomplete

**Suite Assignment Selector**: The suite assignment field uses the same searchable popup pattern:

- Displays currently selected suite name (or "No suite" placeholder)
- Opens searchable popup on click
- Popup shows all available suites with search filtering
- Selection is per-test only (no persistence across sessions)
- Selecting "None" or clearing removes suite assignment

**Dialog Actions**: Two action buttons control the dialog:

| Button | Position | Behavior |
|--------|----------|----------|
| Cancel | Left | Closes dialog without saving, preserves canvas state |
| Save | Right (primary) | Validates inputs, initiates save operation |

**Loading States**: The dialog provides visual feedback during asynchronous operations following WAVE design patterns (see 11-feedback-patterns.md):

| Operation | Loading Indicator | Behavior |
|-----------|-------------------|----------|
| Opening dialog | Suites/tags loading | Skeleton placeholders in dropdown fields until data loads |
| Saving test | Save button spinner | Small spinner replaces button text, button disabled, maintains width |
| Save error | Button returns to normal | Error message appears in dialog footer |

The Save button uses the small spinner variant (16px) that replaces the button text while maintaining button dimensions to prevent layout shift.

**Model Transformation**: The save operation converts the React Flow canvas state to the backend Test model:

| Canvas Concept | Test Model Field | Transformation |
|----------------|------------------|----------------|
| Nodes array | workflow_blocks | Each node becomes a WorkflowBlock with type, parameters, position |
| Edges array | block connections | Edge graph preserves control flow structure (branching and convergence) |
| Node data | block configuration | Plugin type, parameters, variable bindings preserved |
| Node positions | layout metadata | X/Y coordinates stored for canvas restoration on load |

**Control Flow Graph Structure**: The edge graph forms a directed acyclic graph (DAG) supporting control flow patterns:

| Pattern | Structure | Example |
|---------|-----------|---------|
| Sequential | One-to-one | Start → Plugin A → Plugin B → End |
| Branching | One-to-many via control | If/Else → (Branch 1: Script A) or (Branch 2: Script B) |
| Convergence | Many-to-one | Both branches → single Log plugin → continue |
| Parallel | Explicit Parallel control | Parallel control manages concurrent execution |

**Branch Convergence**: Control elements create conditional branches that converge on a common successor element (many-to-one), so execution continues regardless of which branch was taken. The transformation preserves these convergence points in the model. No explicit merge block is required - downstream blocks simply accept multiple incoming edges. If/Else takes exactly one branch and hands it to the main flow. Switch/Case can take several branches in one pass and returns from each, converging instead on its own `then` egress.

##### REQ-3.5.1.1 Conditional execution block (if/else)
##### REQ-3.5.1.5 Custom condition evaluation (expression-based)
##### REQ-3.5.1.6 Visual conditional branching (if/elseif/else)

**If/Else Block Rendering**: The If/Else block supports N output handles (one per branch). The block width expands dynamically as branches are added, maintaining comfortable spacing between handles. Each handle is labeled with the user-defined branch label, or falls back to the branch type ("if", "else if", "else"). Handle IDs are stable monotonic identifiers (e.g., `branch_0`, `branch_1`) that survive reordering and label changes. At design time, all handles render in neutral color. After execution, the taken branch handle is highlighted and untaken branches are dimmed.

**If/Else Configuration**: The configuration modal renders a stacked collapsible section editor. Each branch appears as a card with a type badge (IF/ELSE IF/ELSE), expand/collapse toggle, user-defined label input, and for else-if branches, up/down reorder arrows. Each if/else-if branch body embeds a full assertion builder (literal comparisons, expression rows, AND/OR combinators). The else branch has no conditions. Buttons below the branch list add new else-if or else branches, with the else button disabled when one already exists. A condition row on a null value follows the check rule of REQ-3.1.3.25.

##### REQ-3.5.1.20 Switch/Case multi-branch decisions

**Switch/Case Block Rendering**: The Switch block renders one output handle per case, an optional `default` handle, and an always-present `then` handle, in that order. Block width expands by one handle slot per lane, reusing the branching-block width rule. Handle IDs are stable identifiers that survive reordering and renaming. Lane color encodes role: case lanes take the conditional category color, `default` a muted neutral, and `then` the same neutral the other container controls use for their egress, so branch, fallback and exit are distinguishable at rest. Handles carry no rendered text; the branch name is the handle's accessible name and surfaces in a hover tooltip, matching If/Else and Try/Catch. After execution, every lane that ran is highlighted and the untaken lanes are dimmed - unlike If/Else, more than one case lane can be highlighted in a single run.

**Switch/Case Configuration**: The configuration modal renders the same stacked collapsible branch-card editor as If/Else, sharing its card shell. Each case card carries a CASE badge, expand/collapse toggle, branch name input, a Break switch, up/down reorder arrows, and delete - every case is equal, so none of these are withheld from the first card. The card body embeds the full assertion builder. The Break switch sits in the card header so a collapsed card still reveals whether that case falls through; no further annotation marks fall-through. An optional DEFAULT card sits last with a neutral badge, no conditions, no Break switch, and a body line stating it runs when no case matched or when a case above falls through. Buttons below the list add a case or the default, with the default button disabled when one already exists.

**Variable Selection**: Every surface that picks an existing variable - the assertion builder inside Switch, If/Else, Assert and While, the table transform's filters and sorts, output bindings and variable-reference parameters - uses one searchable variable picker. Long names truncate with the full name available on hover while the type label stays pinned and legible, and typing narrows the list. Namespace roots (`system`, `suite`, `regression`) are not variables and are never offered for selection; only their members appear, each with its real type. Expression autocomplete keeps offering the bare roots, since there the root is the prefix the author types before dotting into it.

**Execution Ordering**: The transformation traverses the edge graph starting from the Start node. For control elements, child branches are captured within the control block's structure. Convergence edges (multiple sources to single target) indicate where branches rejoin the main flow.

**Save Operation Flow**:

1. User clicks Save button (toolbar)
2. For new tests: Save dialog opens for metadata entry
3. For existing tests: Skip dialog, proceed to step 5
4. User enters name, optional description, optional suite selection, tags
5. Frontend validates required fields
6. Canvas state converts to Test model payload
7. API call: POST /api/v1/tests (new) or PUT /api/v1/tests/{id} (update)
8. On success: Dialog closes, breadcrumb updates, dirty state clears
9. On error: Validation errors display inline in dialog

**State Synchronization**: After successful save:

- Canvas dirty state clears (Save button becomes disabled)
- Breadcrumb path updates from "Draft Workflow" to "{Test Name}"
- If suite assigned, breadcrumb shows "{Suite Name} / {Test Name}"
- Test ID stored in canvas context for subsequent updates

**Error Handling**: The dialog displays validation errors inline:

| Error Source | Display Location | Example |
|--------------|------------------|---------|
| Name empty | Below name field | "Test name is required" |
| Name invalid | Below name field | "Name contains invalid characters" |
| Name duplicate | Below name field | "A test with this name already exists" |
| API error | Dialog footer | "Failed to save test. Please try again." |

**Draft Persistence**: The Workbench automatically persists the canvas of a test that has not been saved yet to localStorage:

- Canvas state (nodes, edges, positions) saves to localStorage on every change
- Returning to Workbench restores the exact state as left
- Draft persists across browser refresh, tab close, and navigation away
- Draft includes: workflow structure, element configurations, variable bindings, watched variables list
- localStorage key pattern: `wave-workbench-draft` (single active draft)

**Close Button Behavior**: The toolbar Close button (×) behavior depends on canvas state:

| Canvas State | Close Action | Dialog |
|--------------|--------------|--------|
| No changes (clean) | Immediate close | None - navigates to Tests list |
| Unsaved changes (dirty) | Shows confirmation dialog | "Cancel" / "Discard and Stay" / "Discard and Close" / "Save and Close" |

**Close Confirmation Dialog**: When unsaved changes exist, the Close button presents four actions:

| Button | Behavior |
|--------|----------|
| Cancel | Keeps editing |
| Discard and Stay | Reverts the canvas to the last-saved state and keeps the workflow open |
| Discard and Close | Clears localStorage draft, navigates to Tests list |
| Save and Close | Opens Save Test Dialog, then closes on success |

**Closing Result**: After closing (by either method), the Workbench returns to a clean/draft state ready for new work. The previous draft is cleared from localStorage.

**Navigation Protection**: Additional protection triggers for:

- Browser back/forward navigation
- Clicking navigation links while dirty
- Browser tab/window close (beforeunload event)

##### REQ-3.5.4.3 Wait until condition with timeout

**Wait Until Block Rendering**: The Wait Until block renders its output handles according to
the wait mode. In Retry mode it carries a retry lane followed by the always-present `then`
lane; in Watch mode it carries `then` alone and reads as a sequential block. Block width
follows the same handle-slot rule the other container controls use, so it widens only in
Retry mode. Lane color encodes role: the retry lane takes the repeated-sub-flow color the
loop bodies use, and `then` takes the container-exit neutral. Handles carry no rendered
text; the lane name is the handle's accessible name and surfaces in a hover tooltip. After
execution, the lanes that ran are highlighted and untaken lanes are dimmed.

**Wait Until Configuration**: The configuration modal renders the standard input-parameter
table above the shared assertion builder, the same composition the While loop uses. The
table leads with the required timeout and the check interval, and carries an optional
message describing what the wait was for. Output variables offer the two values the control
produces - whether the condition came true and how long the wait took. A wait-mode radio
section sits where Assert's branching-mode section sits, offering Retry and Watch with a
one-line description each, and below its divider the same fail-current-scope checkbox Assert
offers, which decides whether reaching the time limit fails the block and stops the flow or is
informational only. Assert and Wait Until render this radio-plus-checkbox section through one
shared component rather than each hand-rolling it.

**Switching modes**: Changing from Retry to Watch removes the retry lane, so any wiring on
that lane is reported by the same stale-handle validation that covers Assert's split-to-single
transition. Nothing is silently discarded.

##### REQ-3.5.4.4 Poll-based condition checking

**Progress while waiting**: A Wait Until legitimately occupies the canvas for a long time. The
block carries the ordinary running badge for the whole wait, and the run log carries the
per-round trace, so the reader can tell a live wait from a hung one without the canvas
putting per-round traffic on the wire.

#### 15.4.4 Variable Setter Control Editor

##### REQ-3.4.1.14 Variable setter control

The Variable Setter is a flow control that provides declarative variable assignment within workflows. Its config modal uses a purpose-built inline editor for first-class variable manipulation.

**Inline Variable Rows (VARSET-1)**: The config modal displays variables as editable rows directly in the modal body. Each row contains: Name field, Type selector, Value editor, Expression toggle, and Delete button. A "+ Add" button appears at the top-right of the variables section. All variables are always visible.

**Adding Variables (VARSET-2)**: The "+ Add" button inserts a new empty row inline at the bottom of the list. The user fills in name, picks type, and enters a value directly in the row. No dialog or confirmation step.

**Name Field with Autocomplete (VARSET-3)**: The name field supports both defining new variable names and searching existing test-scope variables via autocomplete. Only test-scope variables appear in the autocomplete suggestions. Typing filters the suggestion list; selecting an existing variable populates the name and locks the type (per VARSET-5).

**No Shadowing (VARSET-4)**: A name that matches a suite-scoped or regression-scoped variable draws no warning, because a setter's variables never shadow those levels: suite and regression variables are read only through their namespaces (`suite.x`, `regression.x`). A bare namespace keyword (`system`, `suite`, `regression`) is rejected as a name with a validation error.

**Type Behavior (VARSET-5)**: The type selector behaves differently based on whether the variable is new or existing:

| Scenario | Type Selector State | Default |
|----------|-------------------|---------|
| New variable (name not in store) | Active dropdown | "string" |
| Existing variable (from autocomplete) | Disabled, shows declared type | N/A |

**Expression Mode Per Variable (VARSET-6)**: Each variable row has its own expression toggle. Activating expression mode replaces the value field with an ExpressionField. The type selector follows VARSET-5 behavior regardless of expression mode. Standard literal-to-expression and expression-to-literal conversion rules apply.

**Complex Type Value Editors (VARSET-7)**: For array, map, and table types, the value cell shows a compact summary with an Edit button:

| Type | Summary Format | Edit Action |
|------|---------------|-------------|
| Array | "3 items" | Opens existing array modal editor |
| Map | "2 pairs" | Opens existing map modal editor |
| Table | "5 rows x 3 col" | Opens existing table modal editor |

These reuse the standard modal editors for complex types.

**Inline Per-Row Validation (VARSET-8)**: Each row validates independently with visual feedback:

| Severity | Border Color | Conditions |
|----------|-------------|------------|
| Error (red) | Red border + message | Invalid Python identifier, duplicate name in this setter, Python keyword, namespace keyword, a name a profile alias uses, a Function input name |

**Value Starts on the Type Default (VARSET-9)**: When an existing variable is selected from autocomplete, the value field starts on its type's default (empty text, `0`, `false`, an empty list, map or table), never on the variable's current value. The setter's purpose is to SET new values, not display current ones.

**Empty State (VARSET-10)**: When the modal opens with no variables configured, it displays one empty row ready for input with the name field auto-focused. This makes the most common action (adding the first variable) immediate.

#### 15.4.5 XML Parser Query Editor

##### REQ-3.4.1.11 XML processing with XPath support

The XML Parser is a data plugin that parses XML documents and executes XPath queries to extract values for workflow variables. Its config modal replaces the generic parameter table for queries with a purpose-built inline query editor, and provides a client-side XPath testing feature.

**Config Modal Layout**: The config modal organizes into four sections:

| Section | Content |
|---------|---------|
| Input Parameters | XML Content (Edit button opens TextEditorModal), Output Mode (dropdown) |
| Queries | Custom inline query editor with add/delete rows |
| Output Variables | Standard output binding table (success, errors, results) |
| Footer | Changelog, Details, Profiles (left), Cancel/Save (right) |

The `queries` and `query_output_types` parameters are hidden from the generic parameter table and rendered by the custom query editor instead.

**Inline Query Rows**: Each query row uses a two-line layout:

| Line | Content |
|------|---------|
| Line 1 | XPath expression input (full width) + expression toggle button |
| Line 2 | Alias/variable name field + type dropdown (individual mode only) + test button + delete button |

The XPath input provides a placeholder hint (`XPath expression (e.g. //item/text())`). The alias field label shows "Assign to" with placeholder "Result alias" in combined mode or "Variable name" in individual mode.

**Output Mode Switching**: The Output Mode dropdown controls query row layout and output variable visibility:

| Mode | Line 2 Fields | Output Variables |
|------|---------------|------------------|
| Combined (default) | Alias + test + delete | success, errors, results |
| Individual | Variable name + type dropdown + test + delete | success, errors (no results - queries are outputs) |

In individual mode, the queries section moves under the "Output Variables" heading and each query row has a type dropdown (String, Integer, Float, Boolean) that controls backend type coercion. Query aliases in individual mode become output variable names with self-bound output bindings.

**Expression Toggle Per Query**: Each query's XPath field has its own expression toggle. When activated, the text input is replaced with a CodeMirror expression editor supporting variable autocomplete and Python expression syntax. Expression-mode queries cannot be tested client-side (test button is disabled).

**XPath Expression Fields and Per-Entry Expression Tracking**: Individual XPath entries within the queries map can be toggled between literal and expression mode independently. The expression tracking system uses dotted notation (`queries.alias_name`) to record which map entries are expressions. This enables mixing literal XPath queries (testable client-side) with dynamic expression-based queries (evaluated at runtime) in the same plugin instance.

**Add and Delete**: The "+ Add Query" button inserts a new empty row. The delete button removes a row. When only one query remains, its delete button is disabled (at least one query is required). An empty state starts with one empty row.

**Duplicate Alias Validation**: When multiple rows share the same alias/variable name, an inline alert appears below the affected rows with "Duplicate query name". This is a blocking validation error - the save action is disabled until duplicates are resolved.

**Client-Side XPath Testing**: Each query row has a test button (play icon) that evaluates the XPath expression against the current XML content entirely in the browser. The test button is disabled when: (a) the XML content is empty, (b) the XPath field is in expression mode, or (c) the XML content is in template mode (contains runtime expressions that cannot be evaluated client-side).

**XPath Test Evaluation**: The client-side evaluator uses the browser's native `DOMParser` and `document.evaluate` API. It parses the XML content, auto-extracts namespace declarations from root element attributes, builds a namespace resolver, and evaluates the XPath expression. Results are classified by XPath result type:

| XPath Result Type | WAVE Type | Display |
|-------------------|-----------|---------|
| Number (integer) | integer | Numeric value |
| Number (float) | float | Decimal value |
| String | string | Quoted string |
| Boolean | boolean | true/false |
| Node set (single) | string | Text content |
| Node set (multiple) | array | Indexed items [0], [1], ... |
| Empty node set | array | Empty array [] |

**Test Result Dialog**: Clicking the test button opens a dialog showing the evaluation result. Successful evaluations display the query alias, type, and value. Failed evaluations display the query alias, an "XPath evaluation failed" description, the error category (XML Parse Error or Invalid XPath), and the error detail message.

**XPath Syntax Validation**: The query editor validates XPath syntax on change using a lightweight validation function. When XML content is available as a literal (not template/expression), the validator parses the XML to extract namespace declarations and builds a resolver, preventing false "unresolvable namespace" errors for prefixed queries. When XML is in template mode, namespace-dependent validation is deferred to the test feature or runtime.

#### 15.4.6 Validation Surface

The validation surface presents structural and configuration issues consistently across every interaction that triggers validation. It has two parts: a persistent per-element alert cue on the canvas, and a single unified validation modal shown when an action is blocked by validation. Toasts are reserved for transient errors and never used for validation findings.

##### Error Class Taxonomy

Three classes drive every routing decision:

| Class | Definition | Examples | User visibility on canvas | Auto-fix behavior |
|-------|-----------|----------|---------------------------|-------------------|
| Fatal | Test cannot be parsed or loaded. | Missing workflow, invalid node array, cycle detected | None - test cannot open | Not auto-fixable |
| Invisible (structural) | The workflow graph or its references are malformed in ways the user cannot detect by inspection. | Missing node IDs, missing positions, dangling edges, undeclared variable bindings, cross-scope edges, namespace-targeting setters | Hidden - the canvas would render normally despite the issue | Auto-fixed by the backend write-path pipeline and by the fix endpoint; an undeclared variable is only fixed by the fix endpoint or the user's Fix All |
| Visible (incomplete) | An element is on the canvas but a required parameter is unfilled, or an output binding points at a wrong-type variable. | Required parameter missing, required parameter empty, output binding type mismatch | Visible via the alert icon on the affected block | Not auto-fixable by the backend - the user must address them; an output binding type mismatch in a stored test is repaired on the canvas when the test opens (Load-Time Routing) |

##### Persistent Alert Badge

Every block whose configuration is currently incomplete (any visible-class issue) carries a small alert badge as the left-most entry in its persistent status bar (§ 15.4.8). The badge is the canonical "warning triangle" from the icon set, sized to fit the 32-pixel bar, rendered in destructive (error) red. Hover surfaces a tooltip naming the offending parameter(s). The cue is reactive: it appears as soon as the user adds an element with mandatory parameters and clears as soon as those parameters become valid.

The alert badge has two states. In its static state, no animation is applied; the badge simply marks the block. In its pulse state, an outline ring around the badge expands and fades repeatedly, drawing attention. Pulse is transient: it activates only after the user attempts an action (Save, Run, or Debug) that the unified modal blocks and the user closes the modal. Pulse clears when the user begins editing the affected block, when the test saves successfully, or when a fresh attempt re-opens the modal.

The alert never sits outside the status bar, never replaces the run-state badge (the two coexist, alert first), and never animates the block's outer border.

##### Unified Validation Modal

When validation needs to surface findings to the user, a single non-dismissible modal appears (no ESC, no backdrop dismiss). The modal's body is the same regardless of trigger: an issue table listing each error that blocks the action with the affected element label, a short description, and (where applicable) a fix action description. A warning never appears in the modal; it stays on its block and, for a Function's unused input or unwritten output, in the run log. Only the title and the action buttons differ by trigger:

| Trigger | Title | Buttons |
|---------|-------|---------|
| Save attempt blocked | Cannot Save Test | Close, and Fix All when an issue is an undeclared variable |
| Run attempt blocked | Cannot Run Test | Close, and Fix All when an issue is an undeclared variable |
| Debug attempt blocked | Cannot Debug Test | Close, and Fix All when an issue is an undeclared variable |
| Load with fatal error | Cannot Load Test | Close (navigates to the tests list), Delete Test |
| Load with invisible (auto-fixable) errors | Test Issues Found | Close (keeps the test open), Fix All |

Closing the modal after a Save, Run, or Debug attempt leaves the user on the canvas and activates the pulse state on every block referenced in the modal's issue list. Closing the modal after a fatal Load attempt navigates the user back to the tests list, because the test cannot be safely worked with while structural issues remain; closing it after a non-fatal Load keeps the test open. Fix All on the Save, Run or Debug modal declares each undeclared variable on the canvas with the type the backend inferred, holding null, keeps every other unsaved edit and dismisses the modal; the user then repeats the action. Fix All on the Load modal calls the fix endpoint, replaces the canvas with the cleaned workflow, and dismisses the modal; if visible warnings remain after the fix, the alert icons render statically on those blocks (no pulse).

##### Load-Time Routing

The frontend calls the validate endpoint with `context=load` in parallel with the data fetch and routes the response by class:

- Fatal: show the unified modal in fatal mode.
- Invisible errors present (with or without visible warnings): show the unified modal in load-non-fatal mode.
- Visible warnings only, no errors: do not show the modal. Render the test on the canvas with static alert icons on the affected blocks. The test is "valid but incomplete" - it persisted in this state because a prior fix run handled the structural cleanup but does not auto-resolve incomplete configuration.
- Clean: render the test with no validation cues.

Before routing, the load matches each output-bound variable's type to the type its block now declares, and declares a bound target the validation does not report, holding null; a variable the validation reports undeclared waits for Fix All.

##### Save, Run, and Debug Routing

The Save flow runs through the backend's unified write-path pipeline. The backend repairs invisible-class issues silently, except an undeclared variable: it never re-creates one on its own and refuses the save, naming the inferred type in the fix action. It also rejects the request when visible-class or fatal issues remain. On rejection (422), the frontend opens the unified modal in save mode.

Run and Debug call the validate endpoint with `context=run` ahead of execution; when the canvas has unsaved edits they send the workflow and variables the run will execute, so the check sees the same body the run start checks. Any error or warning that would block the run surfaces in the unified modal in run or debug mode.

##### Inline Output-Binding Gate

Output binding type compatibility is enforced inline inside the configuration modal. As the user picks a variable in an output binding row, the row checks the variable type against the parameter's declared output type. A mismatch displays an inline error directly under the row and disables the configuration modal's Save button until the user picks a compatible variable. This prevents the test-level Save flow from ever seeing a binding-type mismatch produced by interactive authoring; the corresponding backend validation rule is therefore a safety net for legacy or imported data.

##### Dialog Design

The unified modal uses the standard non-dismissible AlertDialog primitive. Maximum widths are 640 pixels for the load-non-fatal, save, run, and debug variants, and 480 pixels for the load-fatal variant. The issue table shows the error code, message, block context, and (load-non-fatal only) the fix action description. Reduced-motion users see the alert icon in its static form even after Save, Run, or Debug close - the pulse animation falls back to a steady ring under `prefers-reduced-motion: reduce`.

#### 15.4.7 Note Control

The Note is a free-form annotation card on the canvas, visually distinct from execution-bearing blocks. It serves as a sticky-note style element for documenting workflows, marking TODOs, or labeling sections of complex flows. The Note is the single explicit exception to the canonical element-node visual structure (§ 15.4.8): it has no flow handles, no cog button, no dig-in glyph, and its persistent status bar morphs between an idle layout and an edit layout. All other rules from § 15.4.8 (persistent status bar always visible, badges left / actions right, selection halo only) apply unchanged.

**Visual Appearance**: The Note appears as a colored card with a dashed muted border and tinted background in idle mode. Five color presets are available (Yellow, Pink, Blue, Green, Slate). A new Note starts Slate. Each preset uses a light tint for the card fill with a matching muted border, adapting automatically to dark mode. When the Note has no text, the body shows the placeholder "Add your note here..." in muted italic.

**No flow handles**: Notes do not have input or output handles. They sit on the canvas as annotations only; they are not part of the execution graph and cannot be connected by edges. Removing the handles is the simplest contract that matches their annotation-only role and prevents the user from wiring a Note into a flow that the runtime would ignore.

**Modes and Transitions**: A Note has two modes; selection is a state inside either mode and only changes the halo.

| Mode | Border | Persistent status bar contents | Trigger to enter |
|------|--------|--------------------------------|------------------|
| Idle | Dashed, color preset's muted border | Kebab only | Default; also entered when leaving Edit |
| Edit | Solid 2 px blue | Color picker (left); accept (green check) + cancel (red X) (right) | Double-click body, kebab > Edit, or keyboard activation |

Selection in either mode adds the dashed halo from § 15.4.1.5; it does not change the bar's contents or the body border. Idle mode and Edit mode differ in both bar contents and body border simultaneously - this is the single allowed chrome morph and applies only to Notes.

**Status bar (idle)**: A slim 32-pixel bar with the color preset's tint as background. The right cluster contains a kebab button only. The left cluster is empty. The kebab menu provides: Duplicate, Copy, Cut, Unlink, Delete, and Edit (which enters Edit mode). Actions specific to execution-bearing blocks (Configure, Profiles, Skip, Disable) are not available.

**Status bar (edit)**: Same 32-pixel slim bar, same color tint. The left cluster shows a single color-dot button - clicking opens a vertical dropdown of all five color presets as labeled swatches. The right cluster shows an accept button (green check) and a cancel button (red X). Accept commits the textarea contents and returns to Idle; Cancel discards changes and returns to Idle. Escape on the textarea is equivalent to Cancel; nothing on the textarea is equivalent to Accept until the user activates the accept button.

**Inline Text Editing**: Entering Edit mode focuses an in-place textarea over the body. The textarea expands vertically as the user types, growing the card up to the maximum total card height; beyond that, the textarea scrolls internally. Scrolling and panning the canvas do not interfere with text editing. The body border switches to solid blue during Edit, providing an unambiguous "you are editing" affordance distinct from the dashed selection halo.

**Color Picker**: The color picker lives in the status bar's left cluster but only in Edit mode. To recolor an existing Note without changing its text, the user enters Edit mode, picks a color (the swatch dropdown closes immediately on selection and the card color updates live), and Accepts to keep both the (unchanged) text and the new color. There is no canvas-visible affordance to change color outside Edit mode - this keeps the idle status bar to its single kebab and matches the "Notes morph only on Edit" rule.

**Resizing**: The Note is resizable in all directions (vertical, horizontal, diagonal). Resize affordance is cursor-only - no visible grab handles. Minimum size is 120 x 60 pixels. The status bar height contributes to the visual bottom of the card, so dragging the bottom edge resizes from the bar's bottom.

#### 15.4.8 Element Node Visual Structure

Every plugin, control, and Function on the canvas renders through a single shared element-node component. There are no per-plugin or per-control body variations beyond the data displayed; the card shape, the persistent status bar, the body layout, and the resize behavior are uniform across the entire palette. Plugins, controls, and Functions differ only in icon, title source, configuration content, and a small set of variant slots in the status bar - never in the visual contract of the card itself. The Note variant is the single explicit exception and is described in § 15.4.7.

The card is a three-region structure stacked top to bottom: input handle row, body (icon + title + optional description), and a persistent status bar at the bottom. The status bar is always visible; selection does not morph it.

##### Sizing

Element blocks have a width on creation that depends on how they were created, a user-controlled resize range, and a body that grows vertically only when the description is set. The status bar height is fixed.

| Aspect | Value |
|--------|-------|
| Width on creation, block placed by hand | 240 pixels, the canvas default |
| Width on creation, block created from a recipe | Sized to fit its name, within the resize bounds |
| Minimum width | 160 pixels (192 pixels for a Function block) |
| Maximum width | 360 pixels |
| Body height, title only | 44 pixels |
| Body height, with description | 72 pixels (title row + 2-line description clamp) |
| Status bar height | 32 pixels |
| Maximum total card height (auto-grow ceiling) | 120 pixels |

A block placed by hand from the palette starts at the canvas default width, which fits its auto-generated name. A block created from a recipe carries the author's descriptive name from the start, so it is sized on creation to fit that name within the resize bounds. A control whose handle row needs more room than its name is stored at the width its handles draw, which is the width the canvas shows for it either way. The fitted width is stored with the block like a manual resize, and the auto-layout spaces columns by the stored widths so wide blocks never overlap side by side.

A selected card is resizable from every edge and corner, within the width bounds above and a height range of 44 to 240 pixels; the grab areas carry no visible chrome, so the card's outline stays clean. A card with no user-applied height sizes itself from the description state. User-applied width and height both persist across reloads as part of the block's storage payload and survive a recipe amend, the same way a dragged position does.

##### Title and description

The body holds a title row (icon + title) and, when set, a description row beneath it. The title source depends on the block kind; the description is a single optional field shared by all kinds.

| Field | Plugin / Control | Function | Note |
|-------|------------------|----------|------|
| Title source | Human-readable entity name from the registry (e.g. "HTTP", "If / Else", "Var Setter") - immutable per instance | Function entity name (set when the Function is created) - immutable per instance | The note text itself (rendered in the body, not in a separate title row) |
| Description | Optional, multiline, 64-character max, 2-line clamp in display | Same as plugin/control | Not applicable - Notes have no separate description field |

Titles are rendered as human-readable display strings supplied by the registry, not as identifiers. Snake_case (`http_client`), camelCase (`httpClient`), or kebab-case (`http-client`) forms are reserved for the entity's internal id; they never appear in the title row on the canvas. The registry is responsible for supplying a display string for every entity kind.

The description is a per-instance freeform string the test author uses to annotate what THIS instance does in THIS test. It is displayed below the title in muted text and clamps to two lines; long descriptions end with an ellipsis. Clearing the description hides the row and reverts the body to title-only height.

The title is bold, the description is muted; the two never compete visually. There is no user-editable per-instance title for plugins or controls - the title is locked to the registry-supplied display string so the user can always tell at a glance what kind of block it is.

##### Persistent status bar

The status bar sits at the bottom edge of the card and is always visible. Selection adds a dashed halo around the card but does not change the bar's contents or visibility. The bar has two clusters: badges on the left, action cluster on the right. The right edge of the action cluster is anchored to the card's right edge regardless of how many badges sit on the left.

**Left cluster (badges).** Render order, left to right:

1. **Validation alert** - shown when the block has live validation issues (required parameter missing, invalid output-binding type, etc.). Tooltip lists the offending parameter names. Color: destructive (red).
2. **Local / Shared (Function only)** - single-letter `L` (Local, slate) or `S` (Shared, indigo) indicating the Function's lifecycle state. Hover tooltip names the state. Hidden for plugins, controls, and Notes.
3. **Run state** - mutually exclusive: PASS (green), FAIL (red), SKIP (amber), WARN (amber, the block hit a failure it was configured to record and continue past, so the run went on), RUNNING (cyan), PAUSED (amber, debug breakpoint hit), STOPPED (orange, the run ended before this block finished - stopped by the user, or cut off when the section around it ended), DISABLED (neutral). Only one run-state badge is visible at a time. While RUNNING is active, the card border animates with a slow cyan pulse; while DISABLED is active, the entire card renders in grayscale at opacity 0.55. Idle state shows no run-state badge.

**Right cluster (action cluster).** Render order, left to right:

1. **Version-update arrow** - shown only when a newer version of this entity is published. Click target opens the version upgrade flow (out of scope for the visual structure itself).
2. **Cog** - opens the element configuration modal.
3. **Kebab** - opens the per-block context menu (Run, Debug, Edit/Duplicate/Copy/Cut/Unlink, Disable/Skip, Toggle Breakpoint, Delete). The same menu is also surfaced via right-click on the card body.
4. **Dig-in (Function only)** - opens the Function's inner canvas in the same Workbench page. Hidden for plugins, controls, and Notes.

Slots present in the layout but not applicable to the current block kind (`L`/`S` and dig-in for non-Function kinds, version-update arrow when no newer version exists) collapse to zero width so the visible clusters compact tightly without leaving holes.

##### Breakpoint marker

The breakpoint marker is a 12-pixel filled-red circle positioned at the top-right corner of the body, intentionally outside the status bar. It is a status indicator, not an interactive control - setting and clearing the breakpoint happens through the kebab menu's "Toggle Breakpoint" entry or the corresponding keyboard shortcut. The marker's position never overlaps the input handle row above the body because the body's TR corner sits below the handle layer.

##### Selection feedback

Selecting a block adds a dashed halo around the card (2 pixels, white in dark theme, border-strong in light, 4-pixel offset). The halo is the only chrome change; the status bar, badges, action cluster, breakpoint marker, and description visibility are unaffected by selection. Multi-select uses the same halo on each selected card. Click-drag of the canvas background clears the selection.

##### Hover and version surfacing

The version string does not appear on the canvas. The configuration modal header is the single surface that names the version; the ver-update arrow in the action cluster is the only canvas hint that a newer version is available. Hovering individual badges or action-cluster buttons summons each control's own tooltip (`"Configuration incomplete: ..."`, `"Local Function"`, etc.).

Each action-cluster button's tooltip names the action and, beside it, the input that reaches it - so the canvas teaches its own shortcuts instead of hiding them in the hint popover. The input is rendered as a key badge, or as a pointer badge where the affordance is a mouse gesture rather than a key.

| Button | Tooltip | Input badge |
|--------|---------|-------------|
| Cog | Configure | `Enter` |
| Kebab | More options | pointer badge - right-click |
| Dig-in | Open | `Ctrl+Enter` |

The kebab carries no key of its own because it opens the same menu right-clicking the card body opens; naming the gesture is truthful where inventing a shortcut would not be. Action-cluster tooltips are anchored below their button so they never cover the block's own title, which sits directly above the status bar.

##### Handles

Input and output handles attach to the top and bottom edges of the body. Their counts and positions are dictated by the element's parameter declarations and branching mode (where applicable). Handles are independent of the status bar; the status bar sits below the body and never overlaps a handle. Notes are the explicit exception: they have no input or output handles (§ 15.4.7).

##### Accessibility

Every interactive element in the status bar exposes an `aria-label` describing its action (`"Configure"`, `"More actions"`, `"Toggle breakpoint"`, etc.). The badges expose `aria-label` describing their state (`"Configuration incomplete"`, `"Local Function"`, `"Running"`, `"Failed"`, ...). Keyboard focus traverses the card body, then each badge that carries a tooltip, then each button in the action cluster. The breakpoint marker is decorative and `aria-hidden`; the breakpoint state is announced through the kebab menu entry that toggles it.

##### Persistence

The user's resized width and the description text are part of the block's persisted shape, so reopening a test reproduces the visual layout exactly. Run state, breakpoint state, selection, and ver-update arrow visibility are derived from the workflow store and never persisted on the block itself. The runtime engine ignores all of these visual fields - they are authoring-time only.

#### 15.4.9 Function Lifecycle

A Function on a Test canvas is either **Local** (inline definition stored on the Test) or **Published** (a reference to an immutable library entry, identified by UUID). The on-canvas lifecycle badge (`L` slate / `S` indigo) is the single visual cue distinguishing the two; the body chrome is identical across both states. The library is a flat collection of Function definitions. Multi-version history and edit-mode drafts are described with REQ-3.1.2.2 in the Function management design.

##### REQ-3.1.2.1 Create custom Functions with input parameters

The create-and-share loop is the union of three surfaces: the canvas (where Local Functions are built and parameter schemas are declared - input and output parameters are declared through the Variable Manager's `+ Add` menu while drilled into the Function and edited from the row pencil, both in the one variable window, see REQ-3.1.3.11), the publish dialog (which freezes the Local definition into a library entry), and the library modal (which surfaces published entries to other Tests). A user can build a Function once, declare its inputs (and outputs) via the schema editor, then publish to make it reusable. Inputs, outputs (with their Kind) and local variables declared on a Local Function carry through verbatim into the published entry; the library entry stores them as part of its immutable definition.

The publish dialog collects the library-entry metadata (display name, version, description, tags, changelog, details) but never re-collects the parameter schema - that has already been declared on the canvas. Publishing replaces the Test's local content with a published reference; future opens of the same Test render the Function as a Published instance.

##### REQ-3.1.2.3 Function dependency specification

Each Function definition carries a `dependencies` record listing the plugin ids and Function ids referenced inside its workflow. The publish flow does not collect this from the user - it is inferred from the workflow body at publish time. Dependencies are stored on the library entry alongside the workflow so a consumer reading a published Function can determine, without recursive expansion, which plugins and which other published Functions it transitively requires.

Dependency resolution at the library level is what enables the circular-dependency detection described in REQ-3.1.2.4: a publish-time scan of the dependency graph rejects a publish that would close a cycle.

##### REQ-3.1.2.4 Function validation before save

The library entry is validated atomically as part of publish. Two validation passes run before the entry is written: a structural pass (inputs and outputs have names and types, names are unique within the Function's parameter namespace, the workflow is well-formed) and a graph pass (the new dependency graph remains acyclic across the entire library). Either pass returning errors aborts the publish; the publish dialog surfaces the errors inline.

The structural validator is the same one used by the Function-canvas save flow (the REQ-3.1.1.5 path) so a Function that passes Test-level validation does not need to re-prove its body shape at publish time - only the library-level invariants (name uniqueness case-insensitive, version well-formed, dependency graph acyclic across the library) need fresh checks.

##### REQ-5.2.1.38 Function library browser (extension of § 15.4.0.5 Block Palette)

The palette's "Function Lib" affordance opens a non-blocking, draggable sheet that browses the published library, built on the same shared floating-panel primitive as the element configuration sheets: header-as-drag-handle with viewport clamping, right-edge width resize, ESC closing the front-most sheet, click-outside never dismissing, and pointer-down bringing the sheet to front. The library sheet can be open side by side with configuration sheets - the "From library" link inside a Function instance's configuration sheet opens or focuses the library pre-filtered to the source entry without closing anything. The sheet's chrome (search input, single tag-popover button, card grid, drag-source cards, filtered-empty + Clear filters affordance) reuses the WAVE `EntityFilters` pattern verbatim so consistency with the entity list pages is automatic. Tag filtering uses multi-select AND semantics: a card matches only if every selected tag is present on its entry.

Cards are drag sources. Dragging a card and releasing it on the canvas materializes a published Function reference at the drop position; this is how a Test gains a Published Function instance without ever entering the publish flow on the consuming side. The card body deliberately omits the lifecycle badge - the whole modal is the library, so every entry is Shared by definition and a per-card badge would be redundant. The card grid is responsive: columns are added as the modal is resized (minimum one column, card minimum width 220px).

##### 15.4.9.4 Make Local (architectural)

A Published Function instance on a canvas can be converted back into a Local instance via the "Make Local" affordance (kebab on the block and right-click context menu, top of the menu as a lifecycle group with "Open"). This operation deep-copies the published library entry's definition into the Test's storage as the block's inline content. The source library entry is not modified. The local copy has no upstream link - subsequent updates to the published Function do not flow into it; conversely, edits to the local copy do not propagate back. To re-attach, the user drops the Function from the library modal again.

The action opens a confirmation dialog (not a silent fork) so the consequence is explicit. On confirm the workflow block atomically swaps from Shared to Local: `functionLifecycle` becomes `local`, the `functionId` reference is cleared, and the deep-copied definition lands as `localDefinition`. The canvas block visual swaps `S` → `L`; a toast confirms.

##### 15.4.9.5 Read-only inspect view (architectural)

Drilling into a Published Function instance opens its inner canvas in soft-read-only mode. The user can select blocks, toggle breakpoints, and Run / Debug, but cannot add, remove, move, configure, or delete any block inside the Function body. Breakpoints set inside a Published Function live on the calling instance, not on the library entry - two different Tests using the same Published Function can carry independent breakpoint sets, and the library record stores none of them.

The read-only state surfaces in the workbench top action bar (not the breadcrumb bar): immediately to the right of the Env selector, bracketed by vertical splitters, the bar renders a "Read-only" pill in the S-badge indigo palette and an Edit button. The Edit button opens edit mode.

Run / Debug operate against the session's runtime state per the Function canvas integration rules (Run Scope while drilled into a Function, and Run Current or Run from Here on a block in its body, start from the instance's inputs and outputs as its most recent run left them and from its local variables as the session gave them values - a local variable no block wrote follows its definition as it stands - or from declared defaults after Reset or when nothing has run in the session). A Current value the author set before the run is the value the run uses, also right after Reset, and a Required input with no value stops the run and names it (REQ-3.1.2.10). Right-clicking a block in the inspect view opens a menu with only its run and debug items. The read-only inspect offers no paused-state Variable Manager editing.

##### 15.4.9.6 Profile binding on Function instances (architectural)

A Function instance on a canvas - Local or Published - may carry profile bindings the same way a plugin instance does. The Profile binding modal (the existing `ProfileBindingModal` used by plugin and control config) is reachable from a Profiles button in the Function config modal's footer-left group. The library entry itself does not carry profile bindings: the same Published Function dropped into two Tests can carry different profiles without the library being aware. Profile bindings propagate to nested plugin invocations inside the Function body using the same propagation rules in force for plugin-on-canvas bindings.

##### 15.4.9.7 Function lifecycle (architectural)

Versioning - multiple versions, the publish modes including "Major of existing", the per-Test draft edit mode, semver-restricted edits, "Drop changes" and missing-version validation - is designed with REQ-3.1.2.2 in the Function management design.
