### 15.10 Navigation Components

#### 15.10.1 Breadcrumb Navigation

##### REQ-5.2.1.25 Breadcrumb navigation

Breadcrumbs provide hierarchical context showing the user's current location within the application structure. They enable quick navigation to ancestor pages without requiring the browser back button, giving users confidence about where they are and how to return to previous levels.

**Component Anatomy**: A breadcrumb trail consists of four distinct elements working together. The root item represents the application home and always appears first. Path items are intermediate navigation levels rendered as clickable links. The current item shows the active page as non-interactive text, visually distinguished from clickable ancestors. Separators divide items using chevron icons that indicate the directional flow from general to specific.

**Visual Integration**: The breadcrumb component derives its visual properties from the design system to ensure consistency across the application. Text uses the secondary size for information density without competing with page headings. Links appear in subdued colors that darken on hover, while the current page displays in muted tones to indicate its non-interactive nature. The following table maps component elements to design system tokens:

| Element | Property | Token |
|---------|----------|-------|
| Container | Vertical padding | `space-2` (8px) |
| Link text | Font size | `text-sm` (14px) |
| Link text | Font weight | `font-medium` (500) |
| Link text | Color | `--color-text-secondary` |
| Link hover | Color | `--color-text-primary` |
| Current text | Font weight | `font-normal` (400) |
| Current text | Color | `--color-text-muted` |
| Separator icon | Size | `icon-sm` (16px) |
| Separator icon | Color | `--color-text-muted` |
| Item gap | Spacing | `space-1` (4px) |
| Separator margins | Spacing | `space-2` (8px) each side |

**Interaction Behavior**: Path items respond to user interaction with clear visual feedback. On hover, link text transitions to primary color and gains an underline. Focus states display visible rings following accessibility guidelines for keyboard users. The current item remains visually static as it represents the active location rather than a navigation target.

**Hierarchy Representation**: The breadcrumb maps directly to WAVE's entity hierarchy. A typical path progresses from Home through entity type (Regressions, Suites, Tests, Functions), to specific entity instances, and finally to detail views or nested entities. This consistent mapping helps users build mental models of the application structure and navigate efficiently between related items.

**Overflow Handling**: Deep hierarchies may exceed available horizontal space. When this occurs, intermediate items collapse into an ellipsis menu while preserving the root, immediate parent, and current item. Clicking the ellipsis reveals a dropdown containing the hidden path segments, each functioning as a navigation link. This approach maintains usability without sacrificing the full navigation context.

**Canvas-specific application**: The workbench surface has its own breadcrumb bar with constraints that differ from list-page breadcrumbs (fixed height, must coexist with execution status indicators, must handle both navigation-origin and execution-context modes). The canvas-specific contract, including entity icons, the collapse-then-truncate overflow cascade, and the back-arrow target matrix, is documented in section 15.4.0.2 of the Visual Workflow Builder.

#### 15.10.2 Sidebar Navigation

##### REQ-5.2.1.29 Sidebar navigation with collapsible sections

The sidebar provides primary application navigation through a persistent vertical panel positioned along the left edge of the viewport. It organizes navigation items into logical sections, maintaining quick access to all application areas. The sidebar itself supports two display modes: an expanded state showing icons with labels, and a collapsed state displaying icons only for users who prefer maximum content area.

**Component Anatomy**: The sidebar comprises several distinct regions working together. The header area contains the application logo and branding, serving as a home navigation anchor. The navigation region occupies the main body, containing grouped navigation sections. Each section includes a header and a list of navigation items beneath it. The footer area houses persistent controls such as the theme switcher and sidebar collapse toggle. A resize handle along the right edge enables the collapse interaction.

**Section Structure**: Navigation sections group related destinations under descriptive headers. Section headers display in uppercase with reduced emphasis to distinguish them from actionable items.

**Navigation Items**: Each item within a section consists of an icon and label pair. Icons provide quick visual recognition while labels offer explicit identification. In expanded mode, items display horizontally with icon leading the label. In collapsed sidebar mode, only icons appear, with labels available via tooltip on hover. Active items receive visual distinction through background highlighting to indicate the current location within the application.

**Visual Integration**: The sidebar derives its visual properties from the design system to ensure consistency. The following table maps component elements to design system tokens:

| Element | Property | Token |
|---------|----------|-------|
| Container (expanded) | Width | 200px |
| Container (collapsed) | Width | 56px |
| Container | Background | `--color-bg-card` |
| Container | Border right | 1px `--color-border-default` |
| Section header | Font size | `text-xs` (12px) |
| Section header | Text transform | uppercase |
| Section header | Letter spacing | 0.08em |
| Section header | Color | `--color-text-muted` |
| Section header | Font weight | `font-semibold` (600) |
| Section header | Margin bottom | `space-3` (12px) |
| Navigation item | Height | 40px |
| Navigation item | Padding horizontal | `space-3` (12px) |
| Navigation item | Border radius | `rounded-md` (6px) |
| Navigation item | Font size | `text-sm` (14px) |
| Navigation item | Font weight | `font-medium` (500) |
| Navigation item | Color | `--color-text-secondary` |
| Item hover | Background | `--color-bg-subtle` |
| Item hover | Color | `--color-text-primary` |
| Item active | Background | `--color-primary-muted` |
| Item active | Color | `--color-primary` |
| Icon | Size | 20px |
| Icon | Gap from label | `space-3` (12px) |
| Collapse transition | Duration | 300ms |
| Collapse transition | Easing | ease-in-out |

**Interaction Behavior**: The sidebar responds to user actions with clear visual feedback. Navigation items transition background color on hover, indicating interactivity. Clicking an item navigates to the corresponding application area and updates the active state. The sidebar collapse toggle switches between expanded and collapsed modes with a smooth width transition. During collapse animation, labels fade out before the width reduces; during expansion, width increases before labels fade in.

**Collapsed Mode Behavior**: When the sidebar collapses to icon-only mode, navigation remains fully functional. Icons provide visual recognition for frequent users familiar with the application structure. Hovering over an icon displays a tooltip containing the item label for users needing confirmation. Section groupings flatten in collapsed mode, with all items visible as individual icons. Active state indication remains visible through icon background highlighting.

**Desktop Viewport Adaptation**: The sidebar adapts to the full range of desktop browser viewports, from compact laptop displays to ultra-wide monitors. On smaller desktop screens where horizontal space is limited, users may prefer the collapsed sidebar mode to maximize content area. On larger displays, the expanded sidebar provides comfortable navigation without constraining the main content region. A minimum supported viewport width of 1024 pixels ensures the application remains usable on all modern desktop browsers. Below this threshold, the layout degrades gracefully but is not optimized. At a viewport width of 1200 pixels or less the sidebar auto-collapses, and the user can expand it manually; when the window grows back above 1200 pixels it expands again unless the user has toggled it.

**State Persistence**: User preferences for sidebar state persist across sessions. Collapse state of the overall sidebar saves to local storage, restoring on subsequent visits. State restoration occurs during initial render to prevent visual jumps.

##### REQ-5.2.1.50 Layout choices remembered for the browser session

A layout choice is a choice about how a page is arranged or which part of it is shown: a section, a navigation group, a card or a tree node opened or closed, a panel shown, hidden or resized, a view switched. WAVE remembers each such choice for the browser session. The choice comes back the same when the user returns to the page, and after a page load in the same tab. A new tab, and a browser opened again, start from the defaults.

**What a choice belongs to**: A choice is remembered for the control it was made on. A control that exists once (a Settings navigation group, a page section) has one remembered choice. A control that exists once per item keeps a choice per item.

**Defaults and automatic changes**: Until the user makes a choice, a control shows its default, and a rule that opens it on its own (a navigation group opens when one of its pages is shown) still applies. Once the user has chosen, their choice holds.

**Choices kept longer**: A choice WAVE already keeps in the browser beyond the session - the sidebar and its sections, the theme, the time zone, table layouts, console display settings - stays kept that way. The browser session is the least any layout choice is remembered for.

**Not a layout choice**: Which dialog is open, what a form field holds, what is selected, where a list is scrolled, and list filters are not remembered by this rule.
