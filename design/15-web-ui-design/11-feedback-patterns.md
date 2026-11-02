### 15.11 Feedback Patterns

#### 15.11.1 Feedback Components

##### REQ-5.2.1.19 Toast notifications for user feedback

Toast notifications provide transient, non-blocking feedback messages that inform users about the outcome of their actions without interrupting workflow. They appear temporarily to confirm successful operations, warn about potential issues, report errors, or convey informational messages. The toast system enables the application to communicate with users in a consistent, unobtrusive manner across all features.

**Component Anatomy**: A toast notification consists of four primary elements working together. The icon provides immediate visual recognition of the message type through semantic color and shape. The message area contains the primary text conveying the notification content, kept concise for quick scanning. An optional description provides additional context when needed. The dismiss button allows users to close the toast before auto-dismissal, appearing as a subtle close icon that becomes more prominent on hover.

**Positioning and Stacking**: Toasts appear in the bottom-right corner of the viewport, positioned 16 pixels from both edges to maintain visual breathing room. When multiple toasts appear simultaneously, they stack vertically with newer toasts appearing below existing ones. An 8-pixel gap separates stacked toasts, preventing visual crowding while maintaining clear association. The stack grows upward from the bottom, ensuring the newest toast remains closest to typical cursor position and primary content area.

**Visual Integration**: The toast component derives its visual properties from the design system to ensure consistency across the application. Each toast variant uses semantic colors appropriate to its message type. The following table maps component elements to design system tokens:

| Element | Property | Token |
|---------|----------|-------|
| Container | Max width | 360px |
| Container | Padding | `space-4` (16px) |
| Container | Border radius | `rounded-lg` (8px) |
| Container | Shadow | `shadow-lg` |
| Container | Border | 1px semantic border color |
| Message text | Font size | `text-sm` (14px) |
| Message text | Font weight | `font-medium` (500) |
| Message text | Color | semantic text color |
| Description text | Font size | `text-sm` (14px) |
| Description text | Font weight | `font-normal` (400) |
| Description text | Color | semantic text color (80% opacity) |
| Icon | Size | `icon-md` (18px) |
| Icon | Color | semantic text color |
| Dismiss button | Size | 20px target |
| Dismiss button | Color | `--color-text-muted` |
| Dismiss button hover | Color | semantic text color |
| Stack gap | Spacing | `space-2` (8px) |

**Toast Variants**: Four semantic variants communicate different message types through distinct color schemes. Success toasts use green tones to confirm completed operations like saves, deletions, or successful connections. Warning toasts employ amber colors to alert users about potential issues that may require attention but do not prevent continued use. Error toasts display in red to indicate failures requiring user awareness or action. Info toasts use cyan tones for neutral informational messages that neither confirm success nor indicate problems.

| Variant | Background | Text | Border |
|---------|------------|------|--------|
| Success | `--color-success-bg` | `--color-success-text` | `--color-success-border` |
| Warning | `--color-warning-bg` | `--color-warning-text` | `--color-warning-border` |
| Error | `--color-error-bg` | `--color-error-text` | `--color-error-border` |
| Info | `--color-info-bg` | `--color-info-text` | `--color-info-border` |

**Animation Behavior**: Toasts animate into view with a combined slide and fade effect. On appearance, the toast slides upward from below its final position while simultaneously fading from transparent to opaque over 300 milliseconds. The easing function follows the standard ease-out curve for natural deceleration. On dismissal, the toast fades out while sliding downward over 200 milliseconds using ease-in for smooth exit. These animations provide clear visual feedback about toast lifecycle without distracting from primary content.

**Auto-Dismissal**: Toasts automatically dismiss after a configurable duration, defaulting to 5000 milliseconds. This duration provides sufficient time for users to read the message while preventing notification accumulation. Different contexts may warrant adjusted durations: error messages may persist longer to ensure users notice problems, while simple confirmations may dismiss more quickly. Hovering over a toast pauses its dismissal timer, allowing users to read longer messages or interact with the dismiss button without racing against auto-dismissal.

**Interaction Behavior**: Users can dismiss toasts manually by clicking the dismiss button or allowing auto-dismissal. The dismiss button appears at reduced opacity initially, increasing opacity on hover to indicate interactivity without creating visual clutter. When a toast is dismissed, remaining toasts in the stack reposition smoothly to fill the gap, maintaining visual continuity.

**Reduced Motion Support**: When the user has indicated a preference for reduced motion through their operating system settings, toast animations simplify to instant appearance and disappearance. The slide component of animations is removed entirely, leaving only a brief opacity transition. This respects user preferences while maintaining functional feedback about toast lifecycle.

##### REQ-5.2.1.20 Loading skeletons and spinners

Loading indicators provide visual feedback during asynchronous operations, maintaining user engagement while content loads. Two complementary patterns address different loading contexts: spinners indicate indeterminate progress for actions with unknown duration, while skeletons preview content structure during data fetching. Together, these components ensure the interface feels responsive and communicates system state clearly.

**Spinner Component**: Spinners display a rotating circular indicator that communicates ongoing activity. The spinner consists of a circular track with a partial arc that rotates continuously, creating the perception of motion and progress. This pattern applies to button loading states, form submissions, action confirmations, and any operation where the user initiated an action and awaits completion. Spinners draw attention to the loading state, making them appropriate when the user should wait for completion before proceeding.

**Spinner Anatomy**: The spinner comprises two visual elements. The track forms a complete circle rendered at reduced opacity, providing a reference shape for the motion. The indicator arc spans approximately 270 degrees of the circle, rendered at full opacity with the primary accent color. The arc rotates clockwise continuously, creating the loading animation. This two-element approach provides clear motion perception while maintaining visual elegance.

**Spinner Sizes**: Three size variants accommodate different contexts. The small variant fits within compact elements like buttons and inline indicators. The medium variant serves as the default for standalone loading states and overlay centers. The large variant provides emphasis for page-level loading states and empty content areas.

| Size | Diameter | Stroke Width | Usage |
|------|----------|--------------|-------|
| Small | 16px | 2px | Buttons, inline text, table cells |
| Medium | 24px | 2.5px | Cards, form sections, default standalone |
| Large | 40px | 3px | Page loading, empty states, overlays |

**Spinner Visual Integration**: The spinner uses design system tokens for consistent appearance across themes.

| Element | Property | Token |
|---------|----------|-------|
| Track | Color | `--color-border-default` at 30% opacity |
| Indicator arc | Color | `--color-primary` |
| Animation | Duration | 1000ms |
| Animation | Timing function | `ease-linear` |
| Animation | Iteration | Infinite |

**Skeleton Component**: Skeletons render placeholder shapes that mirror the structure of content being loaded. Unlike spinners, skeletons preview the layout users will see once loading completes, reducing perceived wait time and preventing layout shift. This pattern applies to lists, cards, tables, and any content area where the structure is known before data arrives. Skeletons maintain layout stability and set expectations about incoming content.

**Skeleton Anatomy**: A skeleton placeholder consists of a rectangular or circular shape with a subtle shimmer animation. The base color uses a muted background tone, while the shimmer effect creates a highlight that travels across the shape from left to right. This movement indicates activity without demanding attention, allowing users to anticipate the content structure while understanding that loading is in progress.

**Skeleton Variants**: Four shape variants address common content patterns. Text skeletons represent text lines at varying widths to simulate paragraph structure. Circular skeletons serve as avatar and icon placeholders. Rectangular skeletons fill card bodies, images, and block content areas. Custom skeletons combine these primitives into composite loading states matching specific component layouts.

| Variant | Shape | Typical Height | Usage |
|---------|-------|----------------|-------|
| Text | Rounded rectangle | 14-16px | Text lines, labels, descriptions |
| Circular | Circle | Variable diameter | Avatars, icons, status indicators |
| Rectangular | Rounded rectangle | Variable | Cards, images, content blocks |
| Custom | Composite | Variable | Specific component layouts |

**Skeleton Visual Integration**: Skeleton placeholders derive their appearance from design system tokens.

| Element | Property | Token |
|---------|----------|-------|
| Base background | Color | `--color-bg-muted` |
| Shimmer highlight | Color | `--color-bg-subtle` |
| Border radius (text) | Radius | `rounded` (4px) |
| Border radius (card) | Radius | `rounded-lg` (8px) |
| Border radius (circular) | Radius | `rounded-full` |
| Shimmer animation | Duration | 1500ms |
| Shimmer animation | Timing function | `ease-in-out` |
| Shimmer animation | Iteration | Infinite |

**Shimmer Animation**: The shimmer effect creates a subtle wave of brightness traveling across the skeleton surface. A linear gradient with a semi-transparent highlight sweeps from left to right, spanning the full width of the skeleton. The highlight band occupies approximately 40% of the skeleton width at any moment, creating a smooth wave rather than an abrupt flash. This animation suggests loading activity while remaining unobtrusive.

**Composition Guidelines**: Skeleton layouts should accurately reflect the content they represent. Text skeletons should vary in width to simulate natural text line lengths, with the final line typically shorter than preceding lines. When representing lists, skeleton items should maintain consistent spacing matching the actual content grid. Card skeletons should include placeholder elements for all visible card sections, including headers, images, and action areas. This fidelity reduces visual jarring when real content replaces the skeleton.

**Loading State Transitions**: When content loads successfully, skeletons should transition smoothly to the real content. The skeleton fades out while the actual content fades in simultaneously over 150 milliseconds. This crossfade prevents abrupt visual changes and maintains the sense of continuous interface behavior. If loading fails, the skeleton should transition to an appropriate error state rather than remaining indefinitely.

**Reduced Motion Support**: When users prefer reduced motion, loading indicators adapt to respect this preference. Spinners continue rotating but at reduced visual complexity, using a simpler opacity pulse rather than rotation if needed. Skeleton shimmer animations pause entirely, displaying the static placeholder color without movement. The static state still communicates loading through context and position, maintaining functional feedback without motion that may cause discomfort.

##### REQ-5.2.1.21 Empty state illustrations and guidance

Empty states communicate when a view contains no data to display, transforming potentially confusing blank screens into helpful guidance opportunities. These states appear when users first encounter a feature before creating content, after filtering or searching yields no results, or when data has been deleted or archived. Effective empty states reduce user confusion, provide clear direction for next steps, and maintain engagement during the onboarding experience.

**Component Anatomy**: An empty state consists of four hierarchical elements that work together to communicate context and guide action. The illustration provides immediate visual recognition that the empty condition is intentional rather than an error. The title states the current situation concisely in user-friendly language. The description elaborates on why the state is empty and what the user can do about it. The optional call-to-action button provides a direct path to populate the view with content.

**Visual Hierarchy**: The empty state centers vertically and horizontally within its container, drawing focus to the message without competing with surrounding interface elements. The illustration appears topmost, followed by the title, description, and action button in descending order. This arrangement guides the eye naturally from visual recognition through understanding to action.

**Illustration Guidelines**: Empty state illustrations use simple, abstract iconography rather than detailed artwork. Icons from the Lucide library at the `icon-2xl` size (32px) serve as the primary visual element, rendered in muted colors to avoid overwhelming the message. The illustration should metaphorically represent the content type that would normally appear, helping users understand what belongs in the empty space. For example, a folder icon represents missing files, a list icon represents missing items, and a search icon represents no search results.

**Visual Integration**: Empty state components derive their appearance from the design system to maintain consistency across the application.

| Element | Property | Token |
|---------|----------|-------|
| Container | Padding | `space-8` (32px) |
| Container | Max width | 400px |
| Container | Text alignment | Center |
| Illustration | Size | `icon-2xl` (32px) |
| Illustration | Color | `--color-text-disabled` |
| Illustration | Margin bottom | `space-4` (16px) |
| Title | Font size | `text-lg` (18px) |
| Title | Font weight | `font-semibold` (600) |
| Title | Color | `--color-text-primary` |
| Title | Margin bottom | `space-2` (8px) |
| Description | Font size | `text-sm` (14px) |
| Description | Font weight | `font-normal` (400) |
| Description | Color | `--color-text-muted` |
| Description | Line height | 1.5 |
| Description | Margin bottom | `space-6` (24px) |
| Action button | Variant | Primary or Secondary |
| Action button | Size | Medium (40px height) |

**Empty State Variants**: Different contexts require tailored empty state messaging to provide relevant guidance.

| Context | Illustration | Title Example | Description Focus |
|---------|--------------|---------------|-------------------|
| First use | Feature-specific icon | "No tests yet" | Explain the feature's purpose and how to get started |
| No search results | Search icon | "No results found" | Suggest refining search terms or clearing filters |
| Filtered empty | Filter icon | "No matching items" | Explain active filters and offer to clear them |
| Deleted content | Trash icon | "Nothing here" | Confirm deletion context, offer undo if applicable |
| Error-induced empty | Alert icon | "Unable to load" | Explain the problem and provide retry option |

**Content Guidelines**: Empty state copy should be concise, helpful, and action-oriented. Titles use sentence case and avoid technical jargon, stating the situation in three to five words. Descriptions expand on the title with one to two sentences explaining context and suggesting next steps. The tone remains encouraging rather than apologetic, treating the empty state as a starting point rather than a failure condition. Avoid phrases like "Oops!" or "Sorry!" that imply system fault when the state is normal.

**Action Button Behavior**: When an empty state includes a call-to-action, the button should lead directly to the most logical next step. For first-use empty states, this typically opens a creation flow or wizard. For no-results states, the button might clear filters or broaden search scope. The button label uses active verbs describing the outcome, such as "Create Test" or "Clear Filters" rather than vague labels like "Get Started" or "Continue."

**Contextual Placement**: Empty states occupy the content area where data would normally appear, inheriting the layout context of their container. In card-based layouts, the empty state appears within the card boundaries. In full-page views, the empty state centers in the main content region. The empty state should not overlap or displace navigation, headers, or other persistent interface elements, maintaining spatial consistency with the loaded state.

**Transition Behavior**: When content becomes available, the empty state transitions smoothly to the content view. The empty state fades out while the content fades in over 200 milliseconds, preventing jarring visual changes.

##### REQ-5.2.1.22 Error boundaries with recovery options

Error boundaries provide graceful degradation when JavaScript errors occur within the component tree, preventing a single component failure from crashing the entire application. These boundaries catch errors during rendering, in lifecycle methods, and in constructors of child components, displaying a fallback interface that communicates the problem and offers recovery paths. Error boundaries transform potentially catastrophic failures into recoverable situations, maintaining user trust and application stability.

**Component Architecture**: The error boundary system comprises two complementary components working together. The boundary component wraps sections of the application and catches errors that propagate from descendants. The fallback component renders when an error occurs, providing visual feedback and recovery actions. This separation allows the boundary logic to remain focused on error catching while the fallback presentation can be customized for different contexts.

**Boundary Placement Strategy**: Error boundaries should wrap logical application sections rather than individual components. Placing boundaries around major features, routes, and independent widgets prevents error propagation while avoiding excessive granularity that would fragment the user experience. A typical application includes a root-level boundary catching catastrophic failures, route-level boundaries isolating page failures, and feature-level boundaries protecting complex interactive widgets like the workflow builder.

**Fallback Component Anatomy**: The error fallback presents four elements that together communicate the situation and guide recovery. The icon provides immediate visual recognition of an error condition using the error semantic color. The title states that something went wrong in clear, non-technical language. The description offers context about what happened and reassurance that the error has been contained. The action group presents recovery options as clearly labeled buttons.

**Visual Integration**: Error fallback components derive their appearance from the design system, using error semantic colors to communicate severity while maintaining visual consistency.

| Element | Property | Token |
|---------|----------|-------|
| Container | Background | `--color-error-bg` |
| Container | Border | 1px solid `--color-error-border` |
| Container | Border radius | `rounded-lg` (8px) |
| Container | Padding | `space-6` (24px) |
| Container | Max width | 480px |
| Container | Text alignment | Center |
| Icon | Size | `icon-xl` (24px) |
| Icon | Color | `--color-error-text` |
| Icon | Margin bottom | `space-4` (16px) |
| Title | Font size | `text-lg` (18px) |
| Title | Font weight | `font-semibold` (600) |
| Title | Color | `--color-error-text` |
| Title | Margin bottom | `space-2` (8px) |
| Description | Font size | `text-sm` (14px) |
| Description | Font weight | `font-normal` (400) |
| Description | Color | `--color-text-secondary` |
| Description | Line height | 1.5 |
| Description | Margin bottom | `space-6` (24px) |
| Action group | Gap | `space-3` (12px) |
| Primary action | Variant | Primary button |
| Secondary action | Variant | Ghost button |

**Recovery Actions**: Error fallbacks present contextually appropriate recovery options that help users continue their work.

| Action | Label | Behavior | When to Show |
|--------|-------|----------|--------------|
| Retry | "Try Again" | Reset error state and re-render children | Always (primary action) |
| Navigate Home | "Go to Dashboard" | Navigate to application root | Route-level and root boundaries |
| Reload Page | "Reload Page" | Trigger full page refresh | Persistent errors after retry |
| Report Issue | "Report Problem" | Open feedback mechanism | When error reporting is enabled |

**Error Information Handling**: When an error is caught, the boundary captures both the error object and the component stack trace. This information enables appropriate fallback rendering and supports error reporting. In development environments, detailed error information may display to aid debugging. In production, error details are logged for monitoring while the user-facing message remains friendly and non-technical.

**Reset Behavior**: The retry action resets the error boundary state, allowing the failed component tree to attempt re-rendering. This recovery path succeeds when errors result from transient conditions like race conditions, temporary network issues, or stale state. If the same error recurs immediately after reset, the boundary should track retry attempts and suggest alternative actions like page reload after repeated failures.

**Content Guidelines**: Error fallback messaging follows specific patterns to maintain user confidence without over-explaining technical details.

| Element | Pattern | Example |
|---------|---------|---------|
| Title | Brief acknowledgment | "Something went wrong" |
| Description | Context + reassurance | "This section encountered an error and couldn't load. Your other work is safe." |
| Primary action | Active verb + outcome | "Try Again" |
| Secondary action | Clear destination | "Go to Dashboard" |

**Animation Behavior**: When an error boundary catches an error, the fallback fades in over 200 milliseconds to replace the failed content smoothly. If the user triggers a retry and the component renders successfully, the fallback fades out while the recovered content fades in over 150 milliseconds. These transitions maintain visual continuity during the error and recovery cycle.

**Integration with Toast Notifications**: Error boundaries may optionally trigger toast notifications when errors occur, providing additional feedback without requiring users to focus on the error fallback. The toast serves as a supplementary alert, particularly useful when errors occur in peripheral interface areas that may not have immediate user attention. Toast notifications for boundary-caught errors use the error variant and include a brief message referencing the affected area.

**Reduced Motion Support**: When users prefer reduced motion, error fallback transitions simplify to immediate visibility changes without fade animations. The static presentation maintains all functional elements while respecting accessibility preferences.

**Nested Boundary Behavior**: When error boundaries nest within each other, only the nearest ancestor boundary catches a given error. This isolation ensures that feature-level errors display feature-specific fallbacks rather than triggering the root-level fallback. If a nested boundary itself fails, the error propagates to the parent boundary, providing layered protection against cascading failures.

**Thrown Errors Versus Failed Reads**: An error boundary answers code that threw while rendering. A request that comes back as a failure is not that: the component is healthy, it simply has no data. Boundaries therefore never see failed reads, and a failed read never produces the boundary fallback. A page that cannot show its records falls to the full-page error state, and a page that renders but cannot be trusted because a supporting read failed is blocked as described in 15.11.3.

##### REQ-5.2.1.23 Confirmation dialogs for destructive actions

Confirmation dialogs protect users from unintended consequences of destructive actions by requiring explicit acknowledgment before proceeding. These modal interruptions appear when users initiate operations that cannot be easily undone, such as deletions, overwrites, or data resets. By introducing a deliberate pause and requiring conscious confirmation, the dialog pattern prevents accidental data loss while respecting user intent when actions are deliberate.

**Session artifact protection**: Workbench session teardown is one such destructive family (ARTIF-17). Gestures that destroy the live session - resetting it, closing the test, or switching to another test - also destroy the session's artifact files, so whenever the session actually holds artifacts these gestures confirm first. Resetting raises a dedicated destructive confirmation; the existing close and switch dialogs gain a shared warning callout stating how many session artifacts will be permanently deleted and pointing at the Artifacts panel for last-chance downloads. The warning appears only when files would really be lost: a session without artifacts resets instantly, a re-run replacing the previous run's outputs proceeds silently as the expected refresh of results, and sessions tracking a stored execution never warn because their files persist in the permanent artifact zone.

**Component Architecture**: The confirmation dialog system consists of a modal overlay component that presents a focused decision point. The dialog captures user attention by dimming the background content and centering the confirmation prompt. This modal pattern blocks interaction with the underlying interface until the user makes an explicit choice, ensuring the destructive action receives appropriate consideration.

**Component Anatomy**: A confirmation dialog comprises five coordinated elements that together communicate risk and facilitate informed decision-making. The icon provides immediate visual recognition of the action's severity through shape and color. The title states the action requiring confirmation in clear, direct language. The description elaborates on the consequences and permanence of the action, helping users understand what will happen. The confirmation button executes the destructive action when clicked. The cancel button dismisses the dialog without taking action, returning the user to their previous state.

**Visual Hierarchy**: The confirmation dialog centers both vertically and horizontally over the dimmed background, demanding attention without disorienting the user. The icon appears topmost as an immediate severity indicator, followed by the title, description, and action buttons in descending order. The destructive action button appears on the right following the platform convention for primary actions, while the cancel button appears on the left as the safe default escape path.

**Visual Integration**: Confirmation dialog components derive their appearance from the design system, using the small modal size to maintain focus on the decision at hand.

| Element | Property | Token |
|---------|----------|-------|
| Backdrop | Background | `--color-bg-overlay` |
| Container | Background | `--color-bg-elevated` |
| Container | Width | 400px (`sm` modal size) |
| Container | Max height | 90vh |
| Container | Padding | `space-6` (24px) |
| Container | Border radius | `rounded-xl` (12px) |
| Container | Shadow | `shadow-lg` |
| Icon | Size | `icon-xl` (24px) |
| Icon | Color | `--color-error-text` |
| Icon | Margin bottom | `space-4` (16px) |
| Title | Font size | `text-lg` (18px) |
| Title | Font weight | `font-semibold` (600) |
| Title | Color | `--color-text-primary` |
| Title | Margin bottom | `space-2` (8px) |
| Description | Font size | `text-sm` (14px) |
| Description | Font weight | `font-normal` (400) |
| Description | Color | `--color-text-secondary` |
| Description | Line height | 1.5 |
| Description | Margin bottom | `space-6` (24px) |
| Action group | Layout | Flex, justify end |
| Action group | Gap | `space-3` (12px) |
| Cancel button | Variant | Secondary |
| Cancel button | Size | Medium (40px height) |
| Confirm button | Variant | Danger |
| Confirm button | Size | Medium (40px height) |

**Dialog Variants**: Different destructive actions require appropriately tailored confirmation messaging to communicate specific consequences.

| Action Type | Icon | Title Pattern | Description Focus | Confirm Label |
|-------------|------|---------------|-------------------|---------------|
| Delete item | Trash | "Delete {item}?" | Permanence, what will be lost | "Delete" |
| Delete multiple | Trash | "Delete {count} items?" | Scope of deletion, recovery options | "Delete All" |
| Reset/Clear | RefreshCw | "Reset {scope}?" | What will be reset, default state | "Reset" |
| Overwrite | AlertTriangle | "Overwrite {item}?" | What will be replaced, existing data | "Overwrite" |
| Discard changes | X | "Discard changes?" | Unsaved work that will be lost | "Discard" |
| Remove connection | Unlink | "Disconnect {target}?" | Impact on dependent features | "Disconnect" |

**Content Guidelines**: Confirmation dialog copy should be clear, specific, and honest about consequences. Titles use a question format ending with a question mark, naming the specific item or items affected. Descriptions provide concrete information about what will happen, avoiding vague warnings. The tone remains neutral and informative rather than alarming or apologetic. Technical identifiers like test names or file paths should appear in the description when they help users confirm they selected the correct item.

| Element | Pattern | Example |
|---------|---------|---------|
| Title | Action + specific target | "Delete 'Authentication Test'?" |
| Description | Consequence + permanence | "This test and its 12 blocks will be permanently removed. This action cannot be undone." |
| Cancel button | Neutral escape | "Cancel" |
| Confirm button | Active verb matching action | "Delete" |

**Animation Behavior**: Confirmation dialogs animate into view to signal the modal interruption without jarring transitions. On appearance, the backdrop fades from transparent to the overlay color over 200 milliseconds while the dialog container scales from 95% to 100% and fades in over 300 milliseconds. The easing follows the standard ease-out curve for natural deceleration into the final state. On dismissal, these animations reverse using ease-in for smooth exit, with the container animating over 200 milliseconds. Focus transfers to the dialog on open and returns to the triggering element on close.

**Confirmation Policy**: Not all destructive actions require confirmation dialogs. The decision depends on whether the operation supports undo functionality and the significance of the data being affected.

| Action Type | Undo Support | Confirmation Required | Rationale |
|-------------|--------------|----------------------|-----------|
| Simple/leaf deletion (plugin, list item, variable) | Yes | No | Undo provides recovery path; confirmation adds friction without safety benefit |
| Simple/leaf deletion | No | Yes | No recovery path; confirmation prevents accidental loss |
| Container deletion (test, suite, regression) | Yes | Yes | Scope of impact warrants explicit acknowledgment despite undo |
| Container deletion | No | Yes, with type-to-confirm | Permanent loss of multiple items requires highest protection |
| High-value data deletion (artifacts, reports) | N/A | Yes, with type-to-confirm | Industry-standard protection for significant stored data |
| Reset/Clear form settings | Yes | No | Undo restores previous state |
| Reset/Clear form settings | No | Yes | Standard confirmation sufficient for restorable settings |
| Overwrite existing data | Yes | Yes | User should acknowledge replacement even with undo |
| Discard unsaved changes | N/A | Yes | Work in progress cannot be recovered |

Undo functionality must be supported for all operations where technically feasible. The undo system should maintain a history stack allowing users to reverse recent actions.

**Type-to-Confirm Protection**: Type-to-confirm is reserved for operations with significant, permanent impact that align with industry-standard dangerous actions. This additional protection layer applies to deletion of high-value data entities that represent substantial user investment or cannot be recreated.

| Requires Type-to-Confirm | Standard Confirmation Only |
|--------------------------|---------------------------|
| Delete artifact (test execution results) | Clear form settings |
| Delete report | Reset profile preferences |
| Delete regression (without undo) | Clear filters |
| Delete suite (without undo) | Remove single list item |
| Bulk delete operations | Discard unsaved changes |

The dialog includes a text input field where users must type a specific confirmation phrase before the confirm button becomes enabled. This friction ensures users consciously acknowledge the permanence of their action for significant data deletions.

| Element | Property | Token |
|---------|----------|-------|
| Input field | Height | 40px (medium) |
| Input field | Margin top | `space-4` (16px) |
| Input field | Border | 1px solid `--color-border-default` |
| Input field | Border (invalid) | 1px solid `--color-error-border` |
| Input field | Border radius | `rounded` (4px) |
| Instruction text | Font size | `text-sm` (14px) |
| Instruction text | Color | `--color-text-secondary` |
| Instruction text | Margin bottom | `space-2` (8px) |
| Confirm phrase | Font weight | `font-semibold` (600) |
| Confirm phrase | Font family | Monospace (15.9.3) |
| Confirm button | State | Disabled until input matches |

The confirmation phrase should be contextually appropriate: "DELETE" for deletions, "RESET" for resets, or the item name for high-value targets. The instruction text clearly states what the user must type, displaying the required phrase in monospace font for clarity. The confirm button remains disabled with reduced opacity until the input exactly matches the required phrase, case-sensitive.

**Interaction Behavior**: The confirmation dialog supports deliberate dismissal paths. Clicking the cancel button dismisses the dialog without action. Clicking the confirm button executes the destructive action and closes the dialog. Pressing the Escape key does not dismiss the dialog. The backdrop does not dismiss the dialog when clicked, ensuring users make an explicit choice through the provided buttons rather than accidentally dismissing by misclick.

**Keyboard Navigation**: The confirmation dialog supports full keyboard operation for accessibility. When the dialog opens, focus moves to the cancel button as the safe default, ensuring accidental Enter presses do not trigger the destructive action. For type-to-confirm dialogs, focus moves to the confirmation input field instead, allowing immediate typing. Tab cycles focus between interactive elements within the dialog, trapping focus until the dialog closes. Pressing Escape leaves the dialog open. Pressing Enter activates the currently focused button, or submits the form if focus is on the input field and the confirm button is enabled. This focus management ensures keyboard users can navigate the dialog efficiently and safely.

**Focus Trap Behavior**: While the confirmation dialog is open, focus remains trapped within the dialog boundaries. Tabbing from the last focusable element cycles back to the first, and shift-tabbing from the first element cycles to the last. This focus containment prevents users from interacting with the obscured background content and maintains the modal nature of the interruption. When the dialog closes, focus returns to the element that triggered the dialog, maintaining orientation within the page.

**Item Identification**: When confirming deletion of a specific item, the dialog should display identifying information that helps users verify they selected the correct target. For named items, the title includes the item name in quotes. For items with additional context like parent containers or types, the description may include this information. Visual indicators like icons matching the item type reinforce recognition. This specificity prevents the common error of confirming deletion of the wrong item in a list.

**Bulk Action Handling**: When confirming destructive actions on multiple items, the dialog communicates the scope clearly. The title includes the count of affected items. The description may list specific items if the count is small, or summarize the selection criteria for larger sets. A scrollable list within the dialog body may display affected items when verification of specific targets is important. The confirm button label reflects the plural nature of the action.

**Reduced Motion Support**: When users prefer reduced motion, confirmation dialog animations simplify to respect accessibility preferences. The backdrop applies instantly without fade animation. The dialog container appears immediately without scale transformation, using only a brief opacity fade over 150 milliseconds. Focus management and interaction behavior remain unchanged, ensuring full functionality without motion that may cause discomfort.

##### REQ-5.2.1.24 Undo/redo support for critical operations

Undo and redo functionality enables users to reverse and restore editing actions within a session, providing confidence to experiment freely without fear of permanent mistakes. This capability operates entirely on the frontend, managing draft state changes between explicit save operations. The undo system transforms the editing experience from a series of committed decisions into a fluid exploration where users can safely try approaches and backtrack as needed.

**Architectural Model**: The undo/redo system follows a session-scoped, save-bounded architecture. When a user opens an entity for editing, the history initializes with a single baseline entry representing the loaded state - the saved Test, or the loaded published body when a Function is opened standalone from the library. All editing operations within that session accumulate as entries after the baseline. When the user saves, the history floors to a fresh baseline and the saved state becomes the new starting point. This model treats save as a commit point, creating a clear mental model where undo operates only on uncommitted draft changes and can never reach further back than the loaded state.

| Concept | Behavior |
|---------|----------|
| Session start | History holds one baseline entry: the loaded saved/published state |
| Edit operation | Appends a post-change snapshot; any redoable future is discarded |
| Undo action | Moves the pointer one entry back and restores that state |
| Redo action | Moves the pointer one entry forward and restores that state |
| Save operation | Floors history to a single baseline (commit point) |
| Discard changes | Restores last saved state, floors history to the baseline |
| Close without save | Prompts for save/discard, history discarded on close |

**History Timeline Implementation**: The history is a single timeline of complete canvas snapshots (workflow tree, connections, variables) with a pointer; the canvas always equals the entry at the pointer. Full snapshots make restore trivially correct for arbitrarily nested workflow trees and keep equality checks structural rather than operation-dependent.

| Timeline Property | Value | Rationale |
|-------------------|-------|-----------|
| Maximum depth | 50 entries | Balance between utility and memory |
| Storage format | Full state snapshots | Restore correctness for nested trees |
| Baseline retention | The baseline entry is never evicted by the depth cap | The clean (saved) state must always stay reachable |
| Scope | Per editing session | Isolated between different open editors |
| Persistence | None (memory only) | Clears on save, close, or page refresh |

**Tracked Operations**: The undo system tracks editing operations that modify entity state within the current editing context. Operations are categorized by their scope and reversibility.

| Operation Category | Examples | Tracked |
|--------------------|----------|---------|
| Block operations | Add block, delete block, reorder blocks, duplicate block | Yes |
| Block configuration | Update plugin parameters, change block name, modify conditions | Yes |
| Connection operations | Add edge, remove edge, change connection target | Yes |
| Variable operations | Add variable, delete variable, update variable value (root and Function-scoped) | Yes |
| Function contract | Add, edit, delete input/output parameters | Yes |
| Block enable/disable | Toggle disabled, enable all (persisted state) | Yes |
| Function lifecycle entry | Enter edit mode, Make Local | Yes (one entry each) |
| Version pin change | Re-pin a published Function instance to another version | Yes (with binding reconciliation) |
| Layout changes | Move block position | Yes |
| Publish / Drop changes | First publish, edit-mode publish, drop edit-mode changes | No (session collapse, see Function Lifecycle and History) |
| Runtime skip marks | Skip block, unskip all (transient run-time state, never persisted) | No (not content changes) |
| Test/Suite metadata | Update name, description, tags | No (standard form editing, outside canvas history) |
| Permanent deletions | Delete artifact, delete saved test | No (uses confirmation dialog) |
| Navigation | Switch between tabs, open different entity, drill into or out of a Function body | No (not destructive) |
| View preferences | Zoom level, collapsed sections, panel sizes | No (not content changes) |

**Dirty State Tracking**: The unsaved-changes state is derived, not imperatively set: the canvas is dirty exactly when its canonical form differs from the canonical form of the saved baseline. The canonical form is the persistence shape of the canvas - the same transform used to build save and run payloads - so transient view state (drill-materialized published bodies, selection and drag residue) never influences dirtiness. The comparison is re-evaluated at every mutation, undo, redo, and lifecycle-surgery boundary.

| Situation | Dirty State |
|-----------|-------------|
| Entity opened | Clean (canvas equals baseline) |
| Any tracked operation that changes canonical state | Dirty |
| Runtime-transient operations (skip marks, selection, drill) | Unchanged |
| Save completed | Clean (canvas becomes the new baseline) |
| Discard changes | Clean |
| Undo/redo to a state canonically equal to the baseline | Clean |

**Scope and Navigation Reconciliation**: The history timeline is canvas-global: one timeline spans the root workflow and every drilled-into Function body, and entries snapshot the whole tree regardless of where the change was made. Undo and redo never move the user while their current drill location remains valid in the restored state. When a restored state invalidates the location - the drilled Function no longer exists there, or its body is no longer materialized - the drill location automatically pops to the nearest still-valid ancestor scope, and the breadcrumb, canvas chrome, palette scope, and variable panels all follow the reconciled location atomically. Resolution of the active subtree never fails: an invalid location degrades to the nearest valid one rather than producing an error state. Drill navigation itself is never an undoable step, and restoring a snapshot never re-enters a scope on the user's behalf.

**Function Lifecycle and History**: Lifecycle transitions of Function instances interact with history in two distinct ways.

Entering edit mode on a published Function instance and converting an instance with Make Local each record exactly one undoable entry covering their canvas-side effect. Undo reverts only the canvas: a library entry created or referenced by the operation is never altered by undo.

Publishing (the first publish of a local Function, an edit-mode publish that re-pins to the new version, and an edit-mode publish that keeps the old pin) and dropping edit-mode changes are not undoable actions. Each performs a session collapse on the timeline: every retained entry is normalized so that the affected Function (including changes nested anywhere inside its body) appears in its resulting pinned form, entries that become no-ops are removed, and unrelated changes - root workflow edits and edits to other Functions - remain individually undoable. Instance-level changes made in the parent scope during the session (moving the node, renaming the instance, editing bindings) also remain. A publish that re-pins to the new version appends exactly one undoable version-change entry at the tip of the timeline; a keep-old publish and a drop append nothing. A session collapse discards any redoable future, since the collapsed timeline is a new branch point.

Every pin transition - a version-change entry being undone or redone, a keep-old normalization, or a restored manual pin change - reconciles the instance's input and output bindings against the contract of the version being pinned; bindings that no longer match the contract are dropped.

**Keyboard Shortcuts**: Standard platform-specific keyboard shortcuts provide quick access to undo and redo functionality.

| Action | Windows/Linux | macOS |
|--------|---------------|-------|
| Undo | Ctrl+Z | Cmd+Z |
| Redo | Ctrl+Shift+Z or Ctrl+Y | Cmd+Shift+Z |

These shortcuts function globally within the editing context, regardless of which element has focus. The shortcuts are disabled when focus is within a text input field that has its own undo behavior, preventing conflicts with native input undo functionality. Availability is enforced at the state layer, so keyboard shortcuts, toolbar buttons, and any programmatic trigger always agree: while an execution is running or paused, undo and redo are unavailable on every input path. A read-only drilled scope (viewing a published Function body) does not block undo/redo, because the timeline is canvas-global; scope reconciliation keeps the view coherent.

**Toolbar Integration**: The editor toolbar displays undo and redo buttons providing mouse-accessible alternatives to keyboard shortcuts. Button states reflect the availability of undo and redo actions.

| Element | Property | Token |
|---------|----------|-------|
| Button container | Layout | Horizontal button group |
| Button | Variant | Ghost |
| Button | Size | Small (32px height) |
| Icon | Size | `icon-sm` (16px) |
| Icon (enabled) | Color | `--color-text-secondary` |
| Icon (disabled) | Color | `--color-text-disabled` |
| Button (disabled) | Opacity | 0.5 |
| Button (disabled) | Cursor | not-allowed |
| Tooltip | Content | "Undo (Ctrl+Z)" / "Redo (Ctrl+Shift+Z)" |
| Tooltip | Position | Below button |

**Button States**: The undo and redo buttons reflect stack availability through visual states.

| Condition | Undo Button | Redo Button |
|-----------|-------------|-------------|
| Pointer at baseline (nothing to undo) | Disabled | - |
| Entries behind the pointer | Enabled | - |
| No entries ahead of the pointer | - | Disabled |
| Entries ahead of the pointer | - | Enabled |
| During save operation | Disabled | Disabled |
| Execution running or paused | Disabled | Disabled |

**Running with unsaved changes**: Triggering Run or Debug while the canvas has unsaved changes does not open a dialog. The platform executes the exact in-memory working state - the unsaved Test, a drilled-in Function body, or an active edit-mode draft - so unsaved edits are never silently excluded from a run. To run the last-saved version instead, the author first discards the unsaved changes (see Close with Unsaved Changes below), which reverts the canvas to the saved state. There is no save-and-run or discard-and-run prompt; saving is an explicit, separate action.

| Aspect | Behavior |
|--------|----------|
| Dirty Run/Debug | Executes the in-memory working state directly, validated by the same run gate a saved run uses |
| Nesting depth | Unsaved edits execute regardless of where they were made: the root workflow, a drilled-in local Function body, a nested Function inside another Function, or an edit-mode draft at any depth - across full runs, ad-hoc runs, scoped Function runs, and standalone library runs |
| Run the saved version | Discard unsaved changes first (Discard and Stay), then Run |
| Persistence | The run is ephemeral; it never saves the working state |

**Discard Changes Action**: An explicit discard action allows users to abandon all uncommitted changes and return to the last saved state. This action appears in the editor toolbar or menu and requires confirmation when changes exist.

| Trigger Condition | Behavior |
|-------------------|----------|
| No unsaved changes | Action disabled or hidden |
| Unsaved changes exist | Action enabled, shows confirmation |

**Discard Confirmation Dialog**: When discarding changes, a confirmation dialog prevents accidental loss of work.

| Element | Property | Token |
|---------|----------|-------|
| Icon | Icon | AlertTriangle (Lucide) |
| Icon | Color | `--color-warning-text` |
| Title | Text | "Discard Changes?" |
| Description | Text | "All changes since your last save will be lost. This cannot be undone." |
| Cancel button | Variant | Secondary |
| Cancel button | Label | "Keep Editing" |
| Confirm button | Variant | Danger |
| Confirm button | Label | "Discard Changes" |

**Close with Unsaved Changes**: When a user attempts to close a workflow while unsaved changes exist, a dialog prompts for how to proceed. It offers four actions: Cancel (keep editing), Discard and Stay (revert the canvas to the last-saved state but keep the workflow open), Discard and Close (revert and close), and Save and Close. Discard and Stay is the path to running the last-saved version - it returns the canvas to the saved state without leaving the editor.

| Element | Property | Value |
|---------|----------|-------|
| Title | Text | "Unsaved Changes" |
| Cancel button | Label | "Cancel" |
| Discard-and-stay button | Label | "Discard and Stay" |
| Discard-and-close button | Label | "Discard and Close" |
| Save button | Label | "Save and Close" |

**Editing surfaces and the floating editors opened from them**: A surface that opens floating editors and has its own Save - the block configuration sheet, the Function instance sheet, the profile form, and the suite, regression and system variable grids - owns everything open beneath it. Its Save stays disabled while nothing has changed in it or in any editor open from it. Once anything has changed, its Save writes every open editor's unsaved value into its own draft and saves once, as a single undo step, and closing it - by its X, its Close button or Escape - asks the same question first: Keep Editing returns, Close discards the surface's draft and every open editor's, and Save is the surface's own Save. The question offers Keep Editing and Close only where the surface cannot save, as on a read-only canvas.

While a run holds the canvas, an editing surface and everything open beneath it are read-only: the surface carries its READ-ONLY badge, offers no Save, and every field of it and of every editor open from it is frozen. One editor is exempt. When a debug run is paused at a block, that block's script editor, if it is already open, stays live: the author edits the script and runs it, and the run repeats that block with the edited script. Nothing else thaws - not the row the editor was opened from, not another editor of the same block, not another block's configuration, and not that same editor while the run is merely in flight or paused somewhere else. Because the surface still offers no Save, the edit stays a draft until the run ends.

When something cannot be saved - the surface's own fields are invalid, or an open editor that has changes cannot be saved - both Saves are disabled and each carries a red note naming what blocks it, by the row the editor was opened from ("Save is disabled: the Input Variables editor has errors") or by the surface's own fields ("Save is disabled: the settings have errors"). An editor showing errors it has made no change to blocks nothing, and Close still discards everything.

Every other way out follows the same rule and never drops an editor's unsaved text without asking: closing the test, switching to another test, and deleting the element whose configuration is open all count an open editor's changes as unsaved. Where the action removes the surface's subject, as deleting the element does, the question offers Keep Editing (which cancels the action) and Close (which continues it) and never a Save, and only one question is asked for one action.

| Element | Property | Value |
|---------|----------|-------|
| Title | Text | "Unsaved changes" |
| Description | Text | "You have unsaved changes. What would you like to do?" |
| Keep-editing button | Label | "Keep Editing" |
| Close button | Label | "Close" |
| Save button | Label | "Save" |
| Blocked-Save note | Text | "Save is disabled: the &lt;row&gt; editor has errors" / "Save is disabled: the &lt;fields&gt; have errors" |

**A row the surface takes away**: An action that would take away a row whose editor is open asks the same question, with Keep Editing and Close only, when that editor holds unsaved changes. Keep Editing cancels the action and the row stays exactly as it was - a mode switch is not applied, a row is not deleted; Close carries the action out and the row, its editor and its unsaved text all go. This covers a row hidden by a mode switch (a Script Type switched from Source to File), a field switched from a literal value to an expression (which takes away its Edit value sheet and its file picker) and a row the author deletes (an ordered-list row, a parameter row, a variable row). An editor with nothing unsaved in it never stops the action; it simply closes with its row.

**Edit-mode reminder on close**: When a workflow being closed still contains a Shared Function in edit mode, the platform shows an acknowledgement-only warning that the function carries an unpublished draft to publish or discard. The warning is non-blocking - acknowledging it proceeds with the close - so an author can always switch to another workflow; it exists to keep an unlinked edit-mode draft from being forgotten inside a saved Test.

**Browser Navigation Protection**: When unsaved changes exist, the system registers a beforeunload handler that triggers the browser's native confirmation dialog if the user attempts to close the tab, refresh the page, or navigate away using browser controls. This provides a final safety net against accidental data loss.

**Operation Grouping**: Related operations that occur in rapid succession may group into a single undo unit to prevent tedious reversal of granular changes. For example, typing multiple characters in a text field groups into a single undo operation rather than requiring one undo per keystroke.

| Grouping Rule | Behavior |
|---------------|----------|
| Text input | Characters typed within 500ms group together |
| Drag operations | Drag start to drop is single operation |
| Batch selections | Multi-select followed by action is single operation |
| Explicit group | Operations within programmatic transaction group together |

**Memory Management**: The history stack implements size limits to prevent unbounded memory growth during long editing sessions.

| Limit Type | Threshold | Behavior |
|------------|-----------|----------|
| Entry count | 50 entries | Oldest non-baseline entries discarded when exceeded; the baseline entry is always retained |
| Session duration | None | Timeline persists for entire session |

**State Consistency**: When undo or redo executes, the system ensures visual state updates atomically. The workflow canvas, breadcrumb, chrome, property panels, selection, and any dependent views update simultaneously to prevent inconsistent intermediate states; references to elements absent from the restored state (selection, open configuration dialogs, breakpoints) are released as part of the same update. Grouped text edits that are still pending their grouping window are finalized before any undo, redo, save, or lifecycle collapse executes, so no late grouping commit can alter the timeline afterwards. If an undo operation fails due to state conflict, an error toast notifies the user and the history resets to prevent corruption.

**Session collapse boundaries**: Publishing a Function or dropping edit-mode changes collapses that Function's editing session out of the timeline (see Function Lifecycle and History) while preserving unrelated history; a later redo cannot restore a state the collapse removed because the collapse discards the redoable future. Drill locations always reconcile to the nearest valid scope when the structure they referenced is gone (see Scope and Navigation Reconciliation).

**Accessibility Considerations**: The undo/redo system supports full keyboard operation and screen reader compatibility.

| Requirement | Implementation |
|-------------|----------------|
| Keyboard access | Standard shortcuts plus toolbar buttons |
| Screen reader | Announce "Undone: [operation]" / "Redone: [operation]" |
| Focus management | Focus remains stable during undo/redo |
| Visual feedback | Button states clearly indicate availability |

**Reduced Motion Support**: Undo and redo operations execute state changes immediately without animation when users prefer reduced motion. Visual updates apply instantly rather than transitioning, while maintaining all functional behavior.

#### 15.11.2 Delete Blocked by Running Execution

Delete attempts against a test, suite, or regression currently referenced by a running or paused tracked execution are rejected at the server with `409 Conflict`. The response payload carries the identifiers of the blocking execution or executions so the UI can route the user to the right place to resolve the block.

**Toast Surface**: The UI does not open a modal for this case. The user initiated the delete, so the inline toast channel described in REQ-5.2.1.19 is the appropriate acknowledgment. The toast uses the destructive severity and includes a link action navigating to the execution detail view where the user can stop the blocking execution and retry the delete.

| Element | Property | Value |
|---------|----------|-------|
| Severity | Variant | Destructive |
| Title | Pattern | "Cannot delete `<entity type>`" |
| Description | Pattern | "Execution `<id>` is still running. Stop it and try again." |
| Action link | Label | "View execution" |
| Action link | Target | `/executions/<id>` |
| Dismissal | Behavior | Standard toast auto-dismiss plus manual close |

**No Pre-Check**: The UI does not pre-flight a "can I delete?" request before opening its confirmation dialog. The user confirms the delete in the usual destructive-action dialog, the delete request fires, and if the server returns `409` the destructive toast replaces the success feedback. Avoiding the pre-check keeps the interaction symmetric with every other destructive action in the product and sidesteps the race where an execution starts between the pre-check and the delete.

#### 15.11.3 Page Blocked by a Failed Load Step

Loading a page is rarely one request. Alongside the records a page exists to show, most pages read a second, smaller set of data before the user can act on what they see: the environments a profile can hold values for, the tags a list can be filtered by, the suites a test can be filed under. When one of those supporting reads fails while the page itself renders perfectly, the page is not degraded, it is wrong. It presents a complete, operable interface built on data it never received.

**Load Step Classification**: Every read a surface performs while loading is one of two kinds, and the kind decides what its failure looks like. A surface declares the kind for each of its own reads; the kind is a property of what the surface does with the data, not of the endpoint.

| Kind | Definition | Failure Behavior |
|------|------------|------------------|
| Primary | The records the surface exists to show | The error state the surface already owns replaces its content; for a page that is the full-page error state (LIST-5), because there is nothing left to render |
| Supporting | Every other read the surface performs | The surface is blocked, as described below |

**Primary Failure Wins**: When the read a surface exists to show fails, that surface shows the error state it already owns and nothing is blocked, whatever else failed in the same load. The two states never appear together, and a block already raised for a supporting read is cleared as soon as the primary read is known to have failed. The reason is recovery: the full-page error state carries its own Try Again, and a card sitting over it would leave that control inert underneath, so the user would have to get past the block before reaching the one thing that reloads the page, and a one-click recovery would cost two. There is also nothing left to guard, because the records the supporting data was going to serve never arrived either.

**There Is No Third Kind**: No read is optional and no failure is absorbed. A read whose result merely decorates something already on screen is still a read the surface goes on to make claims about: it says these are the tags the user has, these are the types the server offers. If the request never arrived, every one of those claims is unfounded, and a surface that presents them anyway is stating something it does not know. So a failed supporting read blocks whatever asked for it, whether that data was going to fill an editor or paint a filter menu.

**Empty Is Not Failed**: A read that fails is reported as a failure. It is never substituted with an empty collection, a zero, or any other default a caller could mistake for an answer. The two facts are not interchangeable: emptiness is something the server said, and failure is the absence of anything said at all. A page that conflates them tells the user they have no environments when they have several, opens an editor with every field blank, and offers a Save assembled from data it never received.

**A Configuration Is Not a Failure**: One reply leaves a caller without data and is still not a failure, because it is an answer. A capability can be legitimately switched off in a working install, and asking for one that is off gets a supported reply naming that condition: the capability is not enabled here. That is information, not a lost request, so the caller takes the documented default for that capability being off, nothing is blocked and nothing is reported, because nothing went wrong. The line is drawn at what came back and never at how badly the caller wanted the data: a request refused for any other reason, timed out, never delivered, or answered with something unreadable is a failure, and it is reported like every other failure in this section. The distinction is never inferred. Only an explicit answer naming the condition counts, so no failure may be read as "probably switched off". Where the default that stands in the absence of data exists for safety rather than convenience, such as keeping secrets masked until the server says they may be shown, that default holds in both cases: it is what the capability being off means, and it is also what is held while a genuine failure is reported, so a failed read can never expose what a working install would have kept hidden. What the user is told is the only difference between the two. The supported answer passes in silence, because the safe default is what that answer means and nothing went wrong. A genuine failure is reported once, for the attempt that failed, where the user can see it: flags read once for the whole application as it starts are reported in a toast naming what did not load and carrying the server's own reason, since no one surface asked for them. The settings section that edits those flags reads them again when it is opened and, whatever came back, puts in place of its controls the reason they are unavailable and an offer to load them again; that is the state of the capability the section exists to edit, not a second report of a failure. This is the only case in the product where a caller takes a default and reports nothing, and it is deliberately narrow: it covers a supported answer that names its own condition, and nothing else.

**What the Surface Is**: The blocked region is whatever the failed read serves. For a page it is the route content. For a modal that performs its own reads, it is the modal, and a read the modal needs before it can show anything lands in the error state that modal already owns, behind the single Try Again that state carries. A read that only adds to what the modal is already showing, such as the next page of a list the user is scrolling, is not one of those: it has content behind it that the error state would discard, so what is on screen stays and the retry sits beside it. The page behind an open modal is never blurred: the user is inside the modal, and putting a second competing recovery gesture on a surface they did not leave answers the wrong question.

**Which Surface Answers Which Failure**: A failed request is reported where the user is already looking, and what they were doing decides where that is.

| The request | Where its failure is reported |
|-------------|-------------------------------|
| A read a surface performs in order to become usable | The surface is blocked, as described here |
| An action the user just triggered, such as an export, a delete or a backup | Where the gesture was made: the toast that would have confirmed it, or the error line of the dialog it was started from |
| A read repeating on a timer to keep already-rendered data current | In place, beside the data it refreshes, which stays on screen as the last good copy |

An action reports back to the gesture that started it, because that is where the user's attention already is, and blocking a page somebody is still working on answers a question they did not ask. The three differ only in where the report appears. None of them permits silence and none of them permits a default: every failed request is reported somewhere the user can see it, and no failure is ever answered with a value.

**A Block Belongs to Whoever Raised It**: Each block is owned by the surface that raised it, and only that surface clears it. A surface finishing a clean load never removes a block another surface put up, and a failure raised later never takes the place of one the user has not answered yet. A surface that goes away clears its own block, so a block never outlives the surface it describes and none is left waiting on the next page.

**Who Asked Does Not Matter**: A read the surface started on its own, such as restoring work in the background, blocks on failure exactly like one the user triggered. Being interrupted by something you did not ask for is only a trap when there is no way out, and there is no way out to hunt for here: the block has nothing to dismiss and the sidebar is live, so leaving costs one click.

**A Refresh Is Not a Load Step**: This is about becoming usable, not about staying current. A surface that repeats a read on a timer to keep already-rendered data fresh reports a failed tick in place, keeps the last good data and says so, as the Dashboard does with its connection pill and the workbench's session artifacts sheet does with the line above its file list, which keeps the files last read and states beside them that this is the last known state. That is a failure reported, not a failure absorbed, and it is the only shape in which a failed read leaves a surface usable. The same surface's other reads are load steps and behave like load steps: the sheet's own refresh, and the reads that follow the session starting, a run starting or ending and the sheet being opened, block on failure like any other read a surface performs to become usable.

**A Surface Reads Only for What It Is Showing**: A surface performs the reads its visible content needs and no others. A page divided into sections reads a section's data when the user opens that section, so a section nobody is looking at can never block the page over data that is not on screen. Reading everything a page might eventually need on arrival is not done: it makes the whole page hostage to the least relevant thing in it, and someone reading the About section of Settings would be stopped by a tag list that section never uses.

**What the User Sees**: The rest of this section describes the page case, which is the one that needs a treatment of its own; a blocked modal shows the error state it already has. The route content, including any floating sheet open on the page, blurs and stops responding the moment the failure is known. There is no intermediate state, no interstitial and no second step. A card sits over the blurred content carrying a warning icon, a title naming everything that did not load, one line stating that the page cannot be used until what failed has loaded, a folded Details disclosure, and a single Try Again.

| Element | Property | Value |
|---------|----------|-------|
| Blocked region | Treatment | Blurred, under a translucent wash, and inert |
| Card | Position | Centered over the blocked region, outside it |
| Icon | Semantic | Warning |
| Title | Pattern | "Could not load `<what failed>`", naming every read that failed rather than the first |
| Description | Pattern | Agrees with the title: "This page cannot be used until it loads." when one read failed, "This page cannot be used until they load." when more than one did |
| Disclosure | Label | "Details", folded by default |
| Primary action | Label | "Try Again" |
| Dismissal | Behavior | None; the card has no Cancel and no close control |

**Blocked Means Inert, Not Covered**: Nothing inside the blocked region accepts a click, a tap, a keystroke or the Tab key, and it stays that way if the covering layer is removed. A transparent layer that swallows pointer events is not sufficient and is not used: focus, the keyboard and assistive technology all reach straight through it, so the page would remain fully operable for anyone not driving it with a mouse. The blur and the wash laid over the region are presentation only and take no input of their own; the region is genuinely unreachable underneath them. The card is not part of the blocked region, so it stays usable.

**The Boundary Is the Route Content**: Blocking stops where the page ends and the application begins. The navigation sidebar, its theme and collapse controls and everything else belonging to the application shell stay sharp and fully usable, so the user is never stranded and leaving is always one click away. A failed read on one page is no reason to take the whole product away.

**No Cancel**: The card offers no way to dismiss itself, because there is nothing for dismissal to mean. Behind it is a page that cannot be trusted, so handing it back would only restore the problem the block exists to prevent. Navigating away is the way out, and the sidebar is what makes that available at all times.

**Recovery**: Try Again re-runs only the reads that failed. Nothing else is re-fetched, no state is discarded, and the records already on the page are untouched. On success the card disappears and the page is completely normal. On failure the card stays and carries the new failure in its place, so repeated attempts do not accumulate and the user always sees the most recent reason.

**A Retry Re-Reads, It Never Replays**: Every retry a surface offers, the card's Try Again and the surface's own alike, re-runs each of that surface's reads that failed in the load and reports only what those fresh attempts come back with. A reason is never replayed from memory, so everything on screen describes an attempt that has just been made and recovery is one gesture. Re-running part of the set and remembering the rest is wrong twice over: a surface whose own retry re-read only its records put the card straight back up quoting a supporting read this attempt had never touched, one that may already have recovered, and getting out of a single failed load then cost two separate retries.

**Every Failure Is Named**: Each read carries the name it is known by in the title, a short name with no leading article, so that several of them join into one sentence and a name two failed reads share is said once. When more than one of a surface's reads fails in the same load, the surface names all of them rather than only the first. Naming one and dropping the rest is not done: it hides a failure that is about to be reported as fixed, and it leaves the user with no way to tell why the surface is still stopped after a retry they watched succeed. On a blocked page the details then carry each failed read's own request and reason under the name of the read it belongs to, and Try Again re-runs exactly the set that failed, narrowing the card to whichever of them failed again.

**Failure Detail**: The Details disclosure is folded by default, because the normal reading of the card is "this did not load, try again" and the rest is for the person who has to report it. Opened, it holds everything needed to identify one request in a server log.

| Field | Content |
|-------|---------|
| Request | Method and path |
| Outcome | Status code and reason; omitted when the request never got a response, and the message then names the transport failure |
| Duration | Elapsed time until the failure |
| Correlation | Request ID |
| Time | Local timestamp |
| Message | The server's own message, including any trace it returned |

The panel is height-capped and scrolls on both axes. A one-line reason and a hundred-line trace therefore occupy the same space: Try Again never moves, and the card never grows past the window whatever the server sends back.

**Copying the Detail**: A copy control sits in the top-right corner of the details panel, icon only, with a tooltip naming what it does. It appears only while the details are open, and it holds its corner while the panel scrolls. It copies the whole payload rather than the part currently in view, because the reason to copy a trace is to paste all of it somewhere else. Copying works over plain HTTP on a local network, where the browser's clipboard interface is unavailable.

**No Write Is Built From a Failed Read**: While a page is blocked, no editor, dialog or action reachable only from that page can be opened, so nothing can construct a write from data that never arrived. This is the point of blocking rather than warning: a page that merely announced the failure would still let the user open a form whose fields are empty because the values were never loaded, and saving it would erase them. Blocking is not the only guard. A write assembled from a read known to have failed is refused wherever it is attempted, and the refusal is always reported: a refusal is never a gesture that appears to do nothing. Where the surface the user is looking at survives the refusal, such as an editor that stays open, the refusal appears in that surface's own error line, naming what did not load and saying to load it again before saving. Where the surface does not survive, the refusal is reported at the gesture in a toast naming the action that did not happen and the read that did not arrive; confirmation dialogs are the case that matters, since confirming always dismisses them and an error line inside one could never be read. The guarantee therefore does not rest on the block having reached every surface first.

**Accessibility**: The card takes focus when it appears and is announced as an alert. Because the blocked region is genuinely inert, it is absent from the tab order and from the accessibility tree, so no keyboard or screen-reader path leads into it from the card or from the sidebar. Nothing about the arrival is animated: the card is drawn at full strength and the region behind it blurs at the same moment, the moment the failure is known. There is no fade, no entrance transition and therefore no separate reduced-motion path, because there is no motion to suppress.
