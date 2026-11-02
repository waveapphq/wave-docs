### 15.9 Design System

This document defines the complete visual design system for WAVE. Every value is concrete and directly usable during implementation. This is THE source of truth for all frontend visual decisions.

#### 15.9.1 Design Philosophy

##### Visual Identity Statement

**"A clean, data-dense engineering tool with subtle depth and exceptional readability."**

WAVE is built for test engineers who spend hours designing, executing, and analyzing complex workflows. The interface prioritizes:

- **Information density** over whitespace-heavy minimalism
- **Readability** over decorative elements
- **Functional clarity** over aesthetic flourish
- **Quiet confidence** over attention-seeking UI

##### Reference Applications

| Application | What to Adopt |
|-------------|---------------|
| **Linear** | Subtle gradients, bold typography hierarchy, keyboard-first interactions, polished dark mode option |
| **VS Code** | Data-dense layouts, syntax highlighting palette, panel-based workspace, command palette |
| **Vercel Dashboard** | Clean card layouts, status indicators, execution timeline visualization |

##### Design Balance

- **Minimal chrome, maximum content** - UI elements fade into background; data takes center stage
- **Professional, not corporate** - Modern and polished without being sterile or boring
- **Functional beauty** - Every visual element serves a purpose; decoration is functionality

---

#### 15.9.2 Color System

All colors are defined with exact hex values. CSS custom property names are provided for implementation.

##### 15.9.2.1 Light Theme (Primary)

Light mode is the primary theme for WAVE, providing optimal readability and a clean, professional appearance. Dark mode is fully supported as an alternative for user preference or low-light environments.

**Background Colors**

| Token | Hex | CSS Variable | Usage |
|-------|-----|--------------|-------|
| Background Page | `#FFFFFF` | `--color-bg-page` | Main page background |
| Background Card | `#FAFAFA` | `--color-bg-card` | Cards, panels, sidebars |
| Background Elevated | `#FFFFFF` | `--color-bg-elevated` | Dropdowns, popovers, modals |
| Background Overlay | `#00000066` | `--color-bg-overlay` | Modal backdrop (40% opacity) |
| Background Subtle | `#F4F4F5` | `--color-bg-subtle` | Hover states, zebra striping |
| Background Muted | `#E4E4E7` | `--color-bg-muted` | Disabled backgrounds, dividers |

**Text Colors**

| Token | Hex | CSS Variable | Usage | Contrast on #FFFFFF |
|-------|-----|--------------|-------|---------------------|
| Text Primary | `#09090B` | `--color-text-primary` | Headings, important text | 19.5:1 ✓ |
| Text Secondary | `#3F3F46` | `--color-text-secondary` | Body text, descriptions | 10.4:1 ✓ |
| Text Muted | `#71717A` | `--color-text-muted` | Placeholders, hints | 4.6:1 ✓ |
| Text Disabled | `#A1A1AA` | `--color-text-disabled` | Disabled states | 2.6:1 (decorative) |

**Border Colors**

| Token | Hex | CSS Variable | Usage |
|-------|-----|--------------|-------|
| Border Default | `#E4E4E7` | `--color-border-default` | Card borders, dividers |
| Border Subtle | `#F4F4F5` | `--color-border-subtle` | Subtle separators |
| Border Strong | `#A1A1AA` | `--color-border-strong` | Focus rings, active states |

**Primary Accent (Blue)**

| Token | Hex | CSS Variable | Usage | Contrast on #FFFFFF |
|-------|-----|--------------|-------|---------------------|
| Primary Default | `#2563EB` | `--color-primary` | Buttons, links, active states | 5.3:1 ✓ |
| Primary Hover | `#1D4ED8` | `--color-primary-hover` | Hover states | 6.6:1 ✓ |
| Primary Active | `#1E40AF` | `--color-primary-active` | Pressed states | 8.2:1 ✓ |
| Primary Muted | `#DBEAFE` | `--color-primary-muted` | Selected backgrounds | - |

**Semantic Colors - Success (Green)**

| Token | Hex | CSS Variable | Usage |
|-------|-----|--------------|-------|
| Success Background | `#DCFCE7` | `--color-success-bg` | Success alert background |
| Success Text | `#166534` | `--color-success-text` | Success messages (6.4:1 ✓) |
| Success Border | `#86EFAC` | `--color-success-border` | Success alert border |

**Semantic Colors - Warning (Amber)**

| Token | Hex | CSS Variable | Usage |
|-------|-----|--------------|-------|
| Warning Background | `#FEF3C7` | `--color-warning-bg` | Warning alert background |
| Warning Text | `#92400E` | `--color-warning-text` | Warning messages (5.8:1 ✓) |
| Warning Border | `#FCD34D` | `--color-warning-border` | Warning alert border |

**Semantic Colors - Error (Red)**

| Token | Hex | CSS Variable | Usage |
|-------|-----|--------------|-------|
| Error Background | `#FEE2E2` | `--color-error-bg` | Error alert background |
| Error Text | `#991B1B` | `--color-error-text` | Error messages (7.1:1 ✓) |
| Error Border | `#FCA5A5` | `--color-error-border` | Error alert border |

**Semantic Colors - Info (Cyan)**

| Token | Hex | CSS Variable | Usage |
|-------|-----|--------------|-------|
| Info Background | `#CFFAFE` | `--color-info-bg` | Info alert background |
| Info Text | `#155E75` | `--color-info-text` | Info messages (5.7:1 ✓) |
| Info Border | `#67E8F9` | `--color-info-border` | Info alert border |

**Syntax Highlighting (Light Theme)**

| Token | Hex | CSS Variable | Usage |
|-------|-----|--------------|-------|
| Syntax Keyword | `#7C3AED` | `--color-syntax-keyword` | Keywords, control flow |
| Syntax String | `#059669` | `--color-syntax-string` | String literals |
| Syntax Number | `#DB2777` | `--color-syntax-number` | Numbers, constants |
| Syntax Function | `#2563EB` | `--color-syntax-function` | Function names |
| Syntax Variable | `#09090B` | `--color-syntax-variable` | Variables, identifiers |
| Syntax Comment | `#6B7280` | `--color-syntax-comment` | Comments |
| Syntax Type | `#D97706` | `--color-syntax-type` | Types, classes |
| Syntax Operator | `#475569` | `--color-syntax-operator` | Operators, punctuation |
| Syntax Property | `#0891B2` | `--color-syntax-property` | Object properties, attributes |
| Syntax Tag | `#DB2777` | `--color-syntax-tag` | HTML/XML tags |

##### 15.9.2.2 Dark Theme

**Background Colors**

| Token | Hex | CSS Variable | Usage |
|-------|-----|--------------|-------|
| Background Page | `#09090B` | `--color-bg-page` | Main page background |
| Background Card | `#0F0F12` | `--color-bg-card` | Cards, panels, sidebars |
| Background Elevated | `#18181B` | `--color-bg-elevated` | Dropdowns, popovers, modals |
| Background Overlay | `#000000CC` | `--color-bg-overlay` | Modal backdrop (80% opacity) |
| Background Subtle | `#1A1A1E` | `--color-bg-subtle` | Hover states, zebra striping |
| Background Muted | `#27272A` | `--color-bg-muted` | Disabled backgrounds, dividers |

**Text Colors**

| Token | Hex | CSS Variable | Usage | Contrast on #09090B |
|-------|-----|--------------|-------|---------------------|
| Text Primary | `#FAFAFA` | `--color-text-primary` | Headings, important text | 19.5:1 ✓ |
| Text Secondary | `#A1A1AA` | `--color-text-secondary` | Body text, descriptions | 7.2:1 ✓ |
| Text Muted | `#71717A` | `--color-text-muted` | Placeholders, hints | 4.6:1 ✓ |
| Text Disabled | `#52525B` | `--color-text-disabled` | Disabled states | 3.2:1 (decorative) |

**Border Colors**

| Token | Hex | CSS Variable | Usage |
|-------|-----|--------------|-------|
| Border Default | `#27272A` | `--color-border-default` | Card borders, dividers |
| Border Subtle | `#1F1F23` | `--color-border-subtle` | Subtle separators |
| Border Strong | `#3F3F46` | `--color-border-strong` | Focus rings, active states |

**Primary Accent (Blue)**

| Token | Hex | CSS Variable | Usage | Contrast on #09090B |
|-------|-----|--------------|-------|---------------------|
| Primary Default | `#3B82F6` | `--color-primary` | Buttons, links, active states | 4.7:1 ✓ |
| Primary Hover | `#60A5FA` | `--color-primary-hover` | Hover states | 6.8:1 ✓ |
| Primary Active | `#2563EB` | `--color-primary-active` | Pressed states | 4.0:1 (large text) |
| Primary Muted | `#1E3A5F` | `--color-primary-muted` | Selected backgrounds | - |

**Semantic Colors - Success (Green)**

| Token | Hex | CSS Variable | Usage |
|-------|-----|--------------|-------|
| Success Background | `#052E16` | `--color-success-bg` | Success alert background |
| Success Text | `#4ADE80` | `--color-success-text` | Success messages (7.8:1 ✓) |
| Success Border | `#166534` | `--color-success-border` | Success alert border |

**Semantic Colors - Warning (Amber)**

| Token | Hex | CSS Variable | Usage |
|-------|-----|--------------|-------|
| Warning Background | `#422006` | `--color-warning-bg` | Warning alert background |
| Warning Text | `#FBBF24` | `--color-warning-text` | Warning messages (9.4:1 ✓) |
| Warning Border | `#92400E` | `--color-warning-border` | Warning alert border |

**Semantic Colors - Error (Red)**

| Token | Hex | CSS Variable | Usage |
|-------|-----|--------------|-------|
| Error Background | `#450A0A` | `--color-error-bg` | Error alert background |
| Error Text | `#F87171` | `--color-error-text` | Error messages (6.2:1 ✓) |
| Error Border | `#991B1B` | `--color-error-border` | Error alert border |

**Semantic Colors - Info (Cyan)**

| Token | Hex | CSS Variable | Usage |
|-------|-----|--------------|-------|
| Info Background | `#083344` | `--color-info-bg` | Info alert background |
| Info Text | `#22D3EE` | `--color-info-text` | Info messages (8.9:1 ✓) |
| Info Border | `#155E75` | `--color-info-border` | Info alert border |

**Syntax Highlighting (Dark Theme)**

| Token | Hex | CSS Variable | Usage |
|-------|-----|--------------|-------|
| Syntax Keyword | `#C084FC` | `--color-syntax-keyword` | Keywords, control flow |
| Syntax String | `#86EFAC` | `--color-syntax-string` | String literals |
| Syntax Number | `#FDA4AF` | `--color-syntax-number` | Numbers, constants |
| Syntax Function | `#60A5FA` | `--color-syntax-function` | Function names |
| Syntax Variable | `#F9FAFB` | `--color-syntax-variable` | Variables, identifiers |
| Syntax Comment | `#6B7280` | `--color-syntax-comment` | Comments |
| Syntax Type | `#FBBF24` | `--color-syntax-type` | Types, classes |
| Syntax Operator | `#94A3B8` | `--color-syntax-operator` | Operators, punctuation |
| Syntax Property | `#67E8F9` | `--color-syntax-property` | Object properties, attributes |
| Syntax Tag | `#F472B6` | `--color-syntax-tag` | HTML/XML tags |

#### 15.9.3 Typography

##### Font Families

| Usage | Font Stack |
|-------|------------|
| Body (UI and headings) | `"Plus Jakarta Sans", system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif` |
| Monospace (Code) | `"JetBrains Mono", "Fira Code", "SF Mono", Consolas, monospace` |

**Font Pairing Rationale:**
- **Plus Jakarta Sans**: Clean geometric sans-serif, excellent for UI text and headings
- **JetBrains Mono**: Purpose-built for code with ligatures and clear character distinction

**Heading Wrapping**: A heading that wraps onto more than one line balances its lines, so no single word is left alone on the last line.

##### Type Scale

Base size: `16px` (1rem)

| Token | Size (rem) | Size (px) | Line Height | Letter Spacing | Usage |
|-------|------------|-----------|-------------|----------------|-------|
| `text-xs` | 0.75rem | 12px | 1.5 (18px) | 0.01em | Badges, captions |
| `text-sm` | 0.875rem | 14px | 1.5 (21px) | 0 | Secondary text, table cells |
| `text-base` | 1rem | 16px | 1.5 (24px) | 0 | Body text, inputs |
| `text-lg` | 1.125rem | 18px | 1.5 (27px) | -0.01em | Subheadings |
| `text-xl` | 1.25rem | 20px | 1.4 (28px) | -0.01em | Section titles |
| `text-2xl` | 1.5rem | 24px | 1.3 (31px) | -0.02em | Page titles |
| `text-3xl` | 1.875rem | 30px | 1.25 (38px) | -0.02em | Hero text |
| `text-4xl` | 2.25rem | 36px | 1.2 (43px) | -0.02em | Display text |

##### Font Weights

| Token | Weight | Usage |
|-------|--------|-------|
| `font-normal` | 400 | Body text, descriptions |
| `font-medium` | 500 | Labels, table headers, navigation |
| `font-semibold` | 600 | Headings, buttons, emphasis |
| `font-bold` | 700 | Strong emphasis, alerts |

##### Monospace Sizing

Code and configuration editors use monospace font at slightly smaller sizes for density:

| Context | Size | Line Height |
|---------|------|-------------|
| Inline code | 0.875em | inherit |
| Code blocks | 14px | 1.6 (22px) |
| Editor panels | 13px | 1.5 (20px) |
| Terminal output | 13px | 1.4 (18px) |

---

#### 15.9.4 Spacing Scale

Base unit: `4px`

| Token | Value | Pixels | Usage |
|-------|-------|--------|-------|
| `space-0` | 0 | 0px | Reset |
| `space-0.5` | 0.125rem | 2px | Tight inline spacing |
| `space-1` | 0.25rem | 4px | Icon gaps, tight padding |
| `space-1.5` | 0.375rem | 6px | Compact elements |
| `space-2` | 0.5rem | 8px | Button padding (y), input padding |
| `space-3` | 0.75rem | 12px | Card padding (compact), gaps |
| `space-4` | 1rem | 16px | Card padding (default), section gaps |
| `space-5` | 1.25rem | 20px | Medium gaps |
| `space-6` | 1.5rem | 24px | Section padding, large gaps |
| `space-8` | 2rem | 32px | Page margins, major sections |
| `space-10` | 2.5rem | 40px | Hero spacing |
| `space-12` | 3rem | 48px | Large section dividers |
| `space-16` | 4rem | 64px | Page-level spacing |
| `space-20` | 5rem | 80px | Extra large spacing |
| `space-24` | 6rem | 96px | Maximum spacing |

##### Spacing Guidelines

| Context | Recommended Spacing |
|---------|---------------------|
| Button padding | `space-2` (8px) vertical, `space-4` (16px) horizontal |
| Input padding | `space-2` (8px) vertical, `space-3` (12px) horizontal |
| Card padding | `space-4` (16px) default, `space-3` (12px) compact |
| Modal padding | `space-6` (24px) |
| Section gaps | `space-6` (24px) to `space-8` (32px) |
| Page margins | `space-8` (32px) |
| Related items gap | `space-2` (8px) to `space-3` (12px) |
| Unrelated items gap | `space-4` (16px) to `space-6` (24px) |

---

#### 15.9.5 Border Radius Scale

| Token | Value | Pixels | Usage |
|-------|-------|--------|-------|
| `rounded-none` | 0 | 0px | Sharp corners (tables, code blocks) |
| `rounded-sm` | 0.125rem | 2px | Subtle rounding |
| `rounded` | 0.25rem | 4px | Inputs, small buttons |
| `rounded-md` | 0.375rem | 6px | Buttons, cards, tags |
| `rounded-lg` | 0.5rem | 8px | Modals, large cards |
| `rounded-xl` | 0.75rem | 12px | Feature cards, hero elements |
| `rounded-2xl` | 1rem | 16px | Large containers |
| `rounded-full` | 9999px | pill | Badges, avatars, pills |

##### Default Border Radius by Component

| Component | Border Radius |
|-----------|---------------|
| Buttons | `rounded-md` (6px) |
| Inputs | `rounded` (4px) |
| Cards | `rounded-lg` (8px) |
| Modals | `rounded-xl` (12px) |
| Dropdowns | `rounded-md` (6px) |
| Tags/Badges | `rounded-full` (pill) |
| Avatars | `rounded-full` (circle) |
| Toast notifications | `rounded-lg` (8px) |

---

#### 15.9.6 Shadow Scale

Shadows provide subtle depth. In dark mode, shadows are de-emphasized; elevation is conveyed through background color differences.

##### Light Theme Shadows

| Token | CSS Value | Usage |
|-------|-----------|-------|
| `shadow-none` | `none` | Flat elements |
| `shadow-sm` | `0 1px 2px 0 rgb(0 0 0 / 0.05)` | Subtle lift |
| `shadow` | `0 1px 3px 0 rgb(0 0 0 / 0.1), 0 1px 2px -1px rgb(0 0 0 / 0.1)` | Cards, buttons |
| `shadow-md` | `0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1)` | Dropdowns, popovers |
| `shadow-lg` | `0 10px 15px -3px rgb(0 0 0 / 0.1), 0 4px 6px -4px rgb(0 0 0 / 0.1)` | Modals |
| `shadow-xl` | `0 20px 25px -5px rgb(0 0 0 / 0.1), 0 8px 10px -6px rgb(0 0 0 / 0.1)` | Floating sheets |

##### Dark Theme Shadows

| Token | CSS Value | Usage |
|-------|-----------|-------|
| `shadow-none` | `none` | Flat elements |
| `shadow-sm` | `0 1px 2px 0 rgb(0 0 0 / 0.3)` | Subtle lift |
| `shadow` | `0 1px 3px 0 rgb(0 0 0 / 0.4), 0 1px 2px -1px rgb(0 0 0 / 0.4)` | Cards, buttons |
| `shadow-md` | `0 4px 6px -1px rgb(0 0 0 / 0.5), 0 2px 4px -2px rgb(0 0 0 / 0.5)` | Dropdowns |
| `shadow-lg` | `0 10px 15px -3px rgb(0 0 0 / 0.6), 0 4px 6px -4px rgb(0 0 0 / 0.6)` | Modals |
| `shadow-xl` | `0 20px 25px -5px rgb(0 0 0 / 0.7), 0 8px 10px -6px rgb(0 0 0 / 0.7)` | Floating sheets |

##### Shadow Usage Guidelines

| Element | Shadow Level |
|---------|--------------|
| Cards (resting) | `shadow-sm` or `shadow` |
| Cards (hover) | `shadow-md` |
| Buttons | `shadow-none` (rely on color) |
| Dropdowns/Popovers | `shadow-md` |
| Modals | `shadow-lg` |
| Toast notifications | `shadow-lg` |

---

#### 15.9.7 Animation & Transitions

##### Duration Scale

| Token | Duration | Usage |
|-------|----------|-------|
| `duration-75` | 75ms | Micro-interactions (checkbox) |
| `duration-100` | 100ms | Quick feedback (button press) |
| `duration-150` | 150ms | Default transitions (hover states) |
| `duration-200` | 200ms | Dropdowns, popovers |
| `duration-300` | 300ms | Modals, sidebars |
| `duration-500` | 500ms | Page transitions |

##### Easing Functions

| Token | CSS Value | Usage |
|-------|-----------|-------|
| `ease-linear` | `linear` | Progress bars |
| `ease-in` | `cubic-bezier(0.4, 0, 1, 1)` | Exit animations |
| `ease-out` | `cubic-bezier(0, 0, 0.2, 1)` | Enter animations |
| `ease-in-out` | `cubic-bezier(0.4, 0, 0.2, 1)` | Default easing |
| `ease-bounce` | `cubic-bezier(0.34, 1.56, 0.64, 1)` | Playful emphasis (sparingly) |

##### Default Transition

```css
transition: all 150ms cubic-bezier(0.4, 0, 0.2, 1);
```

##### What Animates

| Element | Animation |
|---------|-----------|
| Hover states | Color/background transitions (150ms) |
| Focus rings | Opacity fade (150ms) |
| Dropdowns | Scale + opacity (200ms) |
| Modals | Scale + opacity (300ms) |
| Sidebars | Transform slide (300ms) |
| Toasts | Slide in + opacity (300ms) |
| Loading spinners | Continuous rotation (1000ms) |
| Skeleton loaders | Shimmer effect (1500ms) |

##### What Does NOT Animate

- Layout shifts (avoid CLS)
- Large images loading
- Initial page render
- Any animation when `prefers-reduced-motion: reduce` is set

##### Reduced Motion

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

---

#### 15.9.8 Component Patterns

##### 15.9.8.1 Buttons

**Sizes**

| Size | Height | Padding (x) | Font Size | Icon Size |
|------|--------|-------------|-----------|-----------|
| `sm` | 32px | 12px | 14px | 16px |
| `md` | 40px | 16px | 14px | 18px |
| `lg` | 48px | 24px | 16px | 20px |

**Variants**

| Variant | Background | Text | Border | Hover Background |
|---------|------------|------|--------|------------------|
| Primary | `--color-primary` | `#FFFFFF` | none | `--color-primary-hover` |
| Secondary | `--color-bg-elevated` | `--color-text-primary` | `--color-border-default` | `--color-bg-subtle` |
| Ghost | transparent | `--color-text-secondary` | none | `--color-bg-subtle` |
| Danger | `#DC2626` | `#FFFFFF` | none | `#B91C1C` |

**States**

- **Hover**: Background shift, no scale
- **Active/Pressed**: Darker background, slight inset shadow
- **Disabled**: 50% opacity, cursor not-allowed
- **Loading**: Spinner replaces text/icon, maintains width

##### 15.9.8.2 Inputs

| Property | Value |
|----------|-------|
| Height | 40px (md), 32px (sm), 48px (lg) |
| Padding | 8px vertical, 12px horizontal |
| Border | 1px solid `--color-border-default` |
| Border Radius | 4px (`rounded`) |
| Background | `--color-bg-card` |
| Font Size | 14px |

**States**

| State | Style |
|-------|-------|
| Default | Border: `--color-border-default` |
| Hover | Border: `--color-border-strong` |
| Focus | Border: `--color-primary`, Ring: 2px `--color-primary` at 20% opacity |
| Error | Border: `--color-error-border`, Ring: 2px error at 20% opacity |
| Disabled | Background: `--color-bg-muted`, opacity: 0.5 |

##### 15.9.8.2.1 Field Primitives

The application standardizes text-shaped form inputs onto a small set of shared primitives. Plugins, controls, profiles, environments, the variable manager, and entity-name dialogs all render text inputs through these primitives so that overflow handling, length-cap enforcement, and character feedback are identical across surfaces.

**Layered architecture**

| Layer | Responsibility |
|-------|----------------|
| Base | Owns the visual contract: width, focus / blur overflow behavior, optional length cap, optional character counter. Carries no type-specific logic, parsing, or validation |
| Specialized | Composes the base for a specific role (display name, variable name, parameter literal value, etc.) and adds role-specific defaults and validation |

Type-shaped fields that are not text (booleans, enums) are unaffected and continue to use their dedicated controls.

**Overflow contract (applies to every text-shaped primitive)**

| State | Behavior |
|-------|----------|
| Container width | The field's parent always declares it can shrink, so the field cannot grow horizontally with content |
| Field width | Always fills its parent up to the parent's width |
| Content shorter than field | Rendered fully, no overflow effects |
| Content longer than field, blurred | Truncated with an ellipsis at the right edge |
| Content longer than field, focused | Native horizontal scroll within the field; cursor stays in view |

Multi-line content edits open in a dedicated text editor modal rather than a sized-down inline textarea, so the inline cell stays at single-row height across every form surface.

**Length-cap contract**

When a primitive is configured with a maximum length, the cap is enforced natively (the input rejects further characters) and the user is given graduated feedback rather than a single error state at the limit.

| Filled fraction of cap | Counter display |
|------------------------|-----------------|
| Below 80 percent | Hidden |
| Between 80 and 99 percent | Muted "X / N" below the field |
| 100 percent | Same counter rendered in destructive color |

The counter is announced politely to assistive technology when it crosses the 80-percent threshold.

**Length constants**

Four centralized constants govern name-length caps across the application. Forms must reference these constants rather than duplicating numeric values.

| Constant | Value | Applies to |
|----------|-------|-----------|
| Element name length | 64 | Display name shown on canvas elements |
| Variable name length | 64 | Any variable name (workflow, system, suite, regression, test, global) |
| Entity name length | 64 | Suite, regression, environment, profile names |
| Test name length | 255 | Test names only |

Tests carry the larger cap because they describe specific scenarios and benefit from longer descriptive names. Suites, regressions, environments, and profiles are categories or identifiers and follow the same 64-character contract as variables and canvas elements.

**Adoption rule**

Any new form that needs a text input adopts the appropriate specialized primitive. Direct usage of bare HTML inputs and textareas is reserved for cases where none of the primitives apply (search-as-you-type quick filters, in-place table editors with bespoke keyboard handling, etc.); new bare usages for entity names, variable names, parameter literal values, or descriptions are not added.

##### 15.9.8.3 Cards

| Property | Value |
|----------|-------|
| Background | `--color-bg-card` |
| Border | 1px solid `--color-border-default` |
| Border Radius | 8px (`rounded-lg`) |
| Padding | 16px default, 12px compact |
| Shadow | `shadow-sm` (light - primary), `shadow-none` (dark) |

**Interactive Cards**

| State | Style |
|-------|-------|
| Hover | Background: `--color-bg-subtle`, shadow: `shadow-md` |
| Selected | Border: `--color-primary`, Background: `--color-primary-muted` |

##### 15.9.8.4 Tables

| Property | Value |
|----------|-------|
| Row Height | 48px (default), 40px (compact), 56px (comfortable) |
| Cell Padding | 12px horizontal, centered vertical |
| Header Background | `--color-bg-subtle` |
| Header Font | `font-medium`, `text-sm`, `--color-text-secondary` |
| Zebra Striping | No (use row hover instead) |
| Row Hover | Background: `--color-bg-subtle` |
| Border | Horizontal only: `--color-border-subtle` |

##### 15.9.8.5 Modals

| Property | Value |
|----------|-------|
| Width | `sm`: 400px, `md`: 500px, `lg`: 640px, `xl`: 800px, `full`: 100% - 32px |
| Max Height | 90vh |
| Padding | 24px (`space-6`) |
| Border Radius | 12px (`rounded-xl`) |
| Shadow | `shadow-lg` |
| Backdrop | `--color-bg-overlay` |

**Structure**

- Header: 24px bottom margin, flex with close button
- Body: Scrollable if needed, 24px bottom margin
- Footer: Flex end, button gap 12px

##### 15.9.8.6 Toasts/Notifications

| Property | Value |
|----------|-------|
| Position | Bottom-right, 16px from edges |
| Width | 360px max |
| Padding | 16px |
| Border Radius | 8px (`rounded-lg`) |
| Shadow | `shadow-lg` |
| Animation | Slide up + fade in (300ms) |
| Auto-dismiss | 5000ms (configurable) |
| Stack | 8px gap between toasts |

##### 15.9.8.7 Sidebar Navigation

| Property | Value |
|----------|-------|
| Width (expanded) | 200px |
| Width (collapsed) | 56px |
| Item Height | 40px |
| Item Padding | 12px horizontal |
| Item Border Radius | 6px (`rounded-md`) |
| Active Indicator | Left border 3px `--color-primary` OR background `--color-primary-muted` |
| Icon Size | 20px |
| Transition | Width 300ms ease-in-out |

The sidebar carries navigation only. It holds no search field: a search box that accepts text and does nothing teaches the reader that the product is unfinished. Collapsing the sidebar hides labels, never the way back - the expand control lives at the bottom of the rail.

##### 15.9.8.8 Tooltips

A tooltip is a floating surface, styled from the same family as menus and popovers rather than from an accent color. Accent colors in WAVE carry meaning - selected, primary action - and a hover hint claims neither. The surface is outlined so it holds its shape when it lands over a panel of the same tone, and in the light theme it is filled one step off the page so it separates by shade as well as by edge.

| Property | Value |
|----------|-------|
| Surface | `--color-bg-card` (light) / popover surface (dark) |
| Border | One step stronger than the default border (light); `--color-border-default` at low opacity (dark) |
| Text | `--color-text-primary` |
| Radius | 6px (`rounded-md`) |
| Padding | 5px vertical, 10px horizontal |
| Font size | 12px |
| Shadow | `shadow-md` |
| Arrow | Present, pointing at the trigger, carrying the same fill and outline as the surface |
| Placement | Above the trigger by default; below where the trigger sits at the bottom edge of a card whose own content would be covered |

The arrow is not decoration. Tooltips frequently attach to controls sitting a few pixels apart - the icons in a block's action cluster, a row of table controls - and without an arrow a floating label near three icons does not say which of them it describes.

A tooltip may carry a key badge naming the input that reaches the same action. Badges use the shared key-badge component with its own faint outline, so they stay legible against the tooltip surface instead of dissolving into it. Where the affordance is a mouse gesture rather than a key, the badge carries a pointer glyph and names the gesture. Multiline tooltips wrap and cap their width; the surface, outline and badge treatment are identical.

**One tooltip, one component.** Every hover hint in WAVE - on a button, a badge, a clipped name, a canvas handle, a disabled control - comes from the single shared tooltip. There is no second implementation and no hand-assembled variant at a call site. A product that shows four different hover chips reads as four products.

The component owns the behavior a call site must not re-decide:

| Fixed by the component | Why it is not an option |
|------------------------|-------------------------|
| Appearance | One surface, or the convergence is undone at the first call site that disagrees |
| Stacking above other layers | A tooltip that can be covered is a bug, not a preference |
| Escape dismisses | Consistent escape behavior across every overlay in the product |
| Closes when the pointer leaves the trigger | The popup is a label, not a surface to move onto |

What a call site may choose:

| Option | Purpose |
|--------|---------|
| Reveal delay | One default for ordinary hints; a shorter one for disabled or inert controls, where the user is asking why a click did nothing and the answer must arrive quickly |
| Side and alignment | So a tooltip near an edge, or below a control whose own label sits above it, points the right way |
| Wide wrapping variant | For content of unbounded length - entity names, paths, breadcrumb segments - which would otherwise render as a screen-wide ribbon |
| No content means no tooltip | Passing empty content renders the trigger untouched, so "show only when there is something to say" needs no conditional at the call site |

---

#### 15.9.9 Iconography

##### Icon Library

**Primary**: [Lucide Icons](https://lucide.dev/) - ISC licensed, consistent stroke width, excellent coverage

##### Icon Sizes

| Token | Size | Stroke Width | Usage |
|-------|------|--------------|-------|
| `icon-xs` | 14px | 1.5px | Inline badges |
| `icon-sm` | 16px | 1.5px | Buttons (sm), table actions |
| `icon-md` | 18px | 2px | Buttons (md), navigation |
| `icon-lg` | 20px | 2px | Buttons (lg), section headers |
| `icon-xl` | 24px | 2px | Empty states, feature highlights |
| `icon-2xl` | 32px | 2px | Hero elements |

##### Icon Color

Icons inherit text color by default. Use `currentColor` for stroke.

```css
.icon {
  stroke: currentColor;
  fill: none;
}
```

---

#### 15.9.9a Centralized Icon Registry

##### REQ-5.2.1.46 Icon Name to Component Mapping

The icon registry provides a centralized mapping from string identifiers to Lucide React components. This enables components to reference icons by name rather than direct imports, supporting both hardcoded control definitions and dynamic plugin data from the backend API.

**Architecture**: A single registry module exports a lookup function that resolves icon names to their corresponding React components. The registry validates icon names at compile time using TypeScript union types.

**Control Icons**: Controls use predefined icon names that map to semantic representations of their function:

| Control | Icon Name | Lucide Component | Purpose |
|---------|-----------|------------------|---------|
| If/Else | GitFork | `GitFork` | Branching paths |
| Switch/Case | ListTree | `ListTree` | Multi-branch selection |
| Assert | CheckCircle | `CheckCircle` | Validation checkpoint |
| For | Repeat | `Repeat` | Count-based iteration |
| ForEach | IterationCw | `IterationCw` | Collection iteration |
| While | RefreshCw | `RefreshCw` | Condition-based loop |
| Break | LogOut | `LogOut` | Exit loop early |
| Continue | SkipForward | `SkipForward` | Skip to next iteration |
| Parallel | Split | `Split` | Concurrent execution |
| Try/Catch | ShieldCheck | `ShieldCheck` | Error handling |
| Halt | OctagonX | `OctagonX` | Stop execution |
| Variable Setter | Variable | `Variable` | Variable assignment |
| Delay | Timer | `Timer` | Fixed time pause |
| Wait Until | Hourglass | `Hourglass` | Condition-based pause |

**Plugin Icons**: Plugins specify icon names in their backend metadata. The registry resolves these to components at render time.

**Fallback Behavior**: Unknown icon names resolve to a default placeholder icon (e.g., `HelpCircle`) to prevent render failures while indicating missing configuration.

##### REQ-5.2.1.47 Plugin Palette Renders From Registry Data

The plugin side of the palette is a projection of the plugin registry, never a list maintained by hand. A plugin appears in the palette because it is registered and declares a category; it disappears when it is not. The palette therefore has no plugin knowledge of its own to fall out of date.

The projection is total: the same registry data drives the expanded palette panel, the collapsed rail's flyouts, and the canvas Quick Insert catalog, so a plugin is present in all three or in none. Category grouping comes from each plugin's declared category rather than from a layout decision, which is what lets a new category appear without a palette change.

A category with no plugins projected into it is not rendered - no header, no count, no empty state. The rule is a property of the projection rather than an exclusion naming a particular category: an empty group is simply absent, and fills in by itself when plugins declare it. A group that can only ever report that it is empty spends a reader's trust in the groups that do carry blocks.

##### REQ-5.2.1.48 Hardcoded Control Definitions

Controls are a fixed set of flow modifiers that ship with WAVE. Unlike extensible plugins, controls are not user-definable. The frontend maintains hardcoded control definitions that include all metadata needed for palette rendering and canvas interaction.

**Control Definition Structure**:

| Field | Type | Purpose |
|-------|------|---------|
| id | string | Unique identifier (e.g., "if-else", "delay") |
| name | string | Display name (e.g., "If / Else", "Delay") |
| category | enum | One of: conditional, loop, flow, timing |
| iconName | string | Icon registry key (e.g., "GitFork", "Timer") |
| colorClass | string | Tailwind color class for category theming |

**Control Categories and Colors**:

| Category | Color Class | Semantic Meaning |
|----------|-------------|------------------|
| Conditional | `text-orange-500` | Decision and branching |
| Loop | `text-cyan-500` | Iteration and repetition |
| Flow | `text-green-500` | Execution path modification |
| Timing | `text-yellow-500` | Time-based operations |

**Control Inventory**:

| Category | Controls |
|----------|----------|
| Conditional | If/Else, Switch/Case, Assert |
| Loop | For, ForEach, While, Break, Continue |
| Flow | Parallel, Try/Catch, Halt, Variable Setter |
| Timing | Delay, Wait Until |

**Palette Integration**: The block palette renders controls from these definitions. Each control displays its icon (resolved via registry), name, and category color. Controls support drag-to-canvas interaction for workflow construction.

---

#### 15.9.10 Workflow Builder Visual Specifications

The workflow builder is the centerpiece of WAVE. These specifications ensure a polished, professional appearance.

##### 15.9.10.1 Canvas

| Property | Value |
|----------|-------|
| Background (dark) | `#09090B` |
| Background (light) | `#FAFAFA` |
| Grid Pattern | Dots |
| Grid Dot Color (dark) | `#27272A` |
| Grid Dot Color (light) | `#E4E4E7` |
| Grid Spacing | 20px |
| Grid Dot Size | 1px |
| Min Zoom | 0.25 (25%) |
| Max Zoom | 2.0 (200%) |
| Default Zoom | 1.0 (100%) |

##### 15.9.10.2 Nodes (Blocks)

**Base Node Style**

| Property | Value |
|----------|-------|
| Min Width | 200px |
| Max Width | 320px |
| Border Radius | 8px (`rounded-lg`) |
| Border Width | 1px (default), 2px (selected/executing) |
| Padding | 12px |
| Shadow | `shadow-sm` |

**Node Header**

| Property | Value |
|----------|-------|
| Height | 36px |
| Background | Slightly darker than body |
| Font Size | 14px (`text-sm`) |
| Font Weight | `font-medium` (500) |
| Icon Size | 16px |
| Icon Gap | 8px |

**Node Colors by Type**

| Type | Border Color | Header BG (Light) | Header BG (Dark) |
|------|--------------|-------------------|------------------|
| Default Block | `#A1A1AA` / `#3F3F46` | `#F4F4F5` | `#18181B` |
| Plugin Block | `#3B82F6` | `#DBEAFE` | `#1E3A5F` |
| Control Flow | `#A855F7` | `#F3E8FF` | `#3B1A5F` |
| Parallel Branch | `#F59E0B` | `#FEF3C7` | `#422006` |
| Error Handler | `#EF4444` | `#FEE2E2` | `#450A0A` |

##### 15.9.10.3 Connection Lines (Edges)

| Property | Value |
|----------|-------|
| Stroke Width | 2px |
| Color (default) | `--color-border-strong` |
| Color (selected) | `--color-primary` |
| Style | Bezier curves |
| Arrow | Filled triangle, 8px |
| Animated (executing) | Dashed, moving pattern |

##### 15.9.10.4 Execution State Colors

| State | Node Border | BG Tint (Light) | BG Tint (Dark) | Badge Color |
|-------|-------------|-----------------|----------------|-------------|
| Pending | `#A1A1AA` / `#3F3F46` | none | none | `#71717A` |
| Running | `#3B82F6` | `#DBEAFE` at 50% | `#1E3A5F` at 20% | `#3B82F6` (pulsing) |
| Success | `#22C55E` | `#DCFCE7` at 50% | `#052E16` at 20% | `#22C55E` |
| Warning | `#F59E0B` | `#FEF3C7` at 50% | `#422006` at 20% | `#F59E0B` |
| Failed | `#EF4444` | `#FEE2E2` at 50% | `#450A0A` at 20% | `#EF4444` |
| Skipped | `#71717A` | none | none | `#71717A` |
| Stopped | `#F59E0B` | `#FEF3C7` at 50% | `#422006` at 20% | `#F59E0B` |

##### 15.9.10.5 Execution Animation

| Animation | Duration | Style |
|-----------|----------|-------|
| Running pulse | 2000ms | Border opacity 0.5 → 1.0 |
| Edge flow | 1000ms | Dash offset animation |
| State change | 300ms | Background color fade |
| Node highlight | 150ms | Border color transition |

##### 15.9.10.6 Selection & Interaction

| State | Style |
|-------|-------|
| Hover | Border color lightens, subtle glow |
| Selected | Border: `--color-primary` 2px, shadow: primary glow |
| Multi-select | Selection box: dashed primary border |
| Drag preview | 50% opacity, slight scale (1.02) |
| Drop target | Dashed border, background highlight |

---

#### 15.9.11 Responsive Design

WAVE's layouts adapt across desktop viewports using defined breakpoints.

##### 15.9.11.1 Breakpoint Scale

| Token | Min Width | CSS Variable | Tailwind Prefix | Target Devices |
|-------|-----------|--------------|-----------------|----------------|
| `sm` | 640px | `--breakpoint-sm` | `sm:` | Below the supported desktop range |
| `md` | 768px | `--breakpoint-md` | `md:` | Below the supported desktop range |
| `lg` | 1024px | `--breakpoint-lg` | `lg:` | Small laptops |
| `xl` | 1280px | `--breakpoint-xl` | `xl:` | Laptops, desktops |
| `2xl` | 1536px | `--breakpoint-2xl` | `2xl:` | Large desktops, ultrawide |

##### 15.9.11.2 Container Widths

| Breakpoint | Container Max Width | Padding (horizontal) |
|------------|---------------------|----------------------|
| lg (1024px+) | 1024px | 32px (`space-8`) |
| xl (1280px+) | 1280px | 32px (`space-8`) |
| 2xl (1536px+) | 1440px | 32px (`space-8`) |

##### 15.9.11.3 Layout Adaptations

**Sidebar Navigation**

| Breakpoint | Behavior |
|------------|----------|
| ≥ 1024px | Fixed sidebar, always visible (200px width expanded, 56px collapsed) |

**Main Content Area**

| Breakpoint | Behavior |
|------------|----------|
| ≥ 1024px | Content area with sidebar offset |

**Grid Columns**

| Context | 1024px+ | 1280px+ |
|---------|---------|---------|
| Card grids | 3 col | 4 col |
| Form layouts | 2 col | 2 col |
| Dashboard widgets | 4 col | 4 col |
| Data tables | Full | Full |

##### 15.9.11.5 Typography Scaling

Font sizes remain consistent across breakpoints. Only layout and spacing adapt.

| Element | All Breakpoints |
|---------|-----------------|
| Headings | Fixed sizes per type scale |
| Body text | 16px base |
| UI text | 14px |

##### 15.9.11.7 Media Query Strategy

The responsive system applies base styles, with progressive enhancements applied at each breakpoint threshold using minimum-width conditions.

| Approach | Description |
|----------|-------------|
| Base styles | Applied to all viewports |
| Breakpoint overrides | Applied at `min-width` thresholds (sm, md, lg, xl, 2xl) |
| Progressive enhancement | Each larger breakpoint adds or modifies styles |
| Cascade order | Larger breakpoints override smaller ones naturally |

##### 15.9.11.8 Accessibility Integration

Responsive layouts integrate with the accessibility considerations defined in section 15.9.7:

| Consideration | Responsive Behavior |
|---------------|---------------------|
| Reduced motion | Animation and transition behavior defined in 15.9.7 applies across all breakpoints |
| Focus indicators | Visible focus states maintained at all breakpoints |
| Zoom support | Layouts remain functional up to 200% browser zoom |

---

#### 15.9.12 Requirements Coverage

This design system addresses the following requirements:

| Requirement | Coverage |
|-------------|----------|
| REQ-5.2.1.7 | Modern design system with consistent styling ✓ |
| REQ-5.2.1.8 | Light and dark theme support ✓ |
| REQ-5.2.1.9 | Professional color palette with semantic colors ✓ |
| REQ-5.2.1.10 | Typography system ✓ |
| REQ-5.2.1.11 | Icon library integration (Lucide) ✓ |
| REQ-5.2.1.12 | Smooth animations and micro-interactions ✓ |
| REQ-5.2.1.17 | Responsive layout (desktop breakpoints) ✓ |
| REQ-5.2.1.34 | Sufficient color contrast ratios |

---

#### 15.9.13 Implementation Notes

##### CSS Custom Properties Structure

```css
:root {
  /* Colors - Light theme (Primary) */
  --color-bg-page: #FFFFFF;
  --color-bg-card: #FAFAFA;
  --color-text-primary: #09090B;
  /* ... all light theme tokens ... */
}

:root.dark,
[data-theme="dark"] {
  /* Colors - Dark theme */
  --color-bg-page: #09090B;
  --color-bg-card: #0F0F12;
  --color-text-primary: #FAFAFA;
  /* ... all dark theme tokens ... */
}
```

##### Tailwind CSS Integration

These tokens map directly to a Tailwind configuration extending the default theme.

##### Component Library

Components are built using:
- **Radix UI** primitives for accessibility and behavior
- **Tailwind CSS** for styling
- **Custom tokens** defined in this document

This approach follows the shadcn/ui pattern: copy-paste components with full ownership and customization control.

---

#### 15.9.14 Token Count Summary

| Category | Count |
|----------|-------|
| Background colors | 12 (6 per theme) |
| Text colors | 8 (4 per theme) |
| Border colors | 6 (3 per theme) |
| Primary accent | 8 (4 per theme) |
| Semantic colors | 24 (12 per theme) |
| Syntax highlighting | 20 (10 per theme) |
| Typography sizes | 8 |
| Font weights | 4 |
| Spacing scale | 15 |
| Border radius | 8 |
| Shadows | 12 (6 per theme) |
| Animation durations | 6 |
| Icon sizes | 6 |
| Responsive breakpoints | 5 |
| **Total** | **142 tokens** |
