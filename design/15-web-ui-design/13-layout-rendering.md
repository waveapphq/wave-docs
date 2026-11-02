### 15.13 Layout & Rendering

#### 15.13.1 Responsive Layout

##### REQ-5.2.1.17 Responsive layout with desktop and tablet breakpoints

WAVE runs in desktop browsers only. Responsive layout enables the application to adapt its presentation across desktop viewport sizes, ensuring usability on workstations and laptops.

**Breakpoint Scale**: Five breakpoints define meaningful shifts in layout strategy. The `sm` breakpoint at 640px and the `md` breakpoint at 768px sit below the supported desktop range. The `lg` breakpoint at 1024px serves small laptops. The `xl` breakpoint at 1280px targets standard laptops and desktops with full feature density. The `2xl` breakpoint at 1536px accommodates large desktops and ultrawide displays with maximum content width and enhanced spacing.

**Container Behavior**: Content containers constrain maximum width and apply consistent horizontal padding. At 1024px and above, containers constrain to their breakpoint width (1024px, 1280px, or 1440px maximum) with 32px padding, preventing content from stretching uncomfortably wide on large displays.

**Sidebar Adaptation**: The sidebar serves as the primary navigation mechanism. It remains fixed and always visible at 200px width expanded and 56px collapsed, with the main content area offset accordingly.

**Grid Columns**: Multi-column layouts adjust based on available width. Card grids display three columns from 1024px to 1279px and four columns at 1280px and above. Form layouts use two columns. Data tables display fully.

**Visual Integration**: The responsive system derives its values from the design system tokens. The following table maps breakpoint boundaries to design system definitions:

| Breakpoint | Min Width | CSS Variable | Container Max | Padding |
|------------|-----------|--------------|---------------|---------|
| Base | 0 | - | 100% | `space-4` (16px) |
| `sm` | 640px | `--breakpoint-sm` | 100% | `space-6` (24px) |
| `md` | 768px | `--breakpoint-md` | 100% | `space-6` (24px) |
| `lg` | 1024px | `--breakpoint-lg` | 1024px | `space-8` (32px) |
| `xl` | 1280px | `--breakpoint-xl` | 1280px | `space-8` (32px) |
| `2xl` | 1536px | `--breakpoint-2xl` | 1440px | `space-8` (32px) |

**Viewport State Management**: The application tracks viewport dimensions through a centralized mechanism, debouncing resize events at 100ms intervals. Components subscribe to breakpoint changes rather than raw dimensions, receiving notifications when crossing thresholds.

**Performance Considerations**: Layout recalculations during resize must remain efficient. CSS-based breakpoints handle presentation changes where possible. Components memoize breakpoint-dependent renders. Transitions disable during active resize to prevent animation overhead.

#### 15.13.2 List Virtualization

##### REQ-5.2.1.18 Virtualized lists for large datasets

List virtualization renders only visible items plus a small buffer, enabling smooth 60fps scrolling for collections exceeding 1,000 items. The system operates transparently - scrolling, keyboard navigation, and visual presentation remain indistinguishable from fully-rendered lists.

**Windowed Rendering**: A rendering window tracks the visible viewport region, shifting as the user scrolls. Spacer elements above and below maintain accurate scrollbar proportions while the total height reflects the full collection size. During rapid scrolling, placeholder content may appear briefly before resolving within a single frame.

**Item Size Strategy**: Fixed-height mode enables direct position calculation and offers superior performance. Variable-height mode accommodates dynamic content but requires item measurement and cumulative indexing.

| List Context | Size Mode | Item Height |
|--------------|-----------|-------------|
| Execution log | Fixed | 40px (compact row) |
| Variable inspector | Variable | Dynamic |

**Overscan Buffer**: Items render beyond the viewport to prevent blank space during scrolling. Buffer size varies by context: 3-5 items for standard scrolling and 1-2 for keyboard navigation.

**Keyboard and Focus**: Arrow keys, Page Up/Down, and Home/End navigate sequentially with the window shifting to keep focused items rendered. Focus persists across render cycles and transfers to the nearest item when the focused item is removed. Screen readers announce position within the full collection.

**System Integration**: Collections below 50-100 items bypass virtualization entirely.

**Performance Targets**:

| Metric | Target | Measured Against |
|--------|--------|------------------|
| Initial render | < 50ms | 10,000 item collection |
| Scroll frame time | < 16ms | Continuous scrolling |
| Memory per 1,000 items | < 1MB overhead | Fixed-height items |
| Keyboard navigation | < 100ms response | Focus shift latency |

**Accessibility**: All items keyboard-reachable, focus clearly visible, screen reader position announcements accurate, no content flashing, reduced motion preferences honored.
