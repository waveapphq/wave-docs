### 15.14 UI Design Process

#### 15.14.1 Purpose

This subsection establishes a rigorous, user-centered design methodology for WAVE's user interface. While other sections in `Web UI Design` document the technical implementation (components, APIs, state management), this section focuses on the **design discipline** that must precede and guide that implementation.

#### 15.14.3 Design Philosophy

**User-First, Not API-First**

Every design decision must trace back to a user need:
- "Why does this screen exist?" → Because the user needs to accomplish X
- "Why is this button here?" → Because it's the next logical action after Y
- "Why these navigation items?" → Because these are the primary workflows

**Visual Before Code**

No production code should be written until:
1. The screen has been wireframed
2. The wireframe has been reviewed
3. The interactions have been specified
4. The states have been documented
5. A Playwright test has been written from the spec

**Design System Discipline**

Every visual element must trace to the design system:
- No magic numbers (use tokens)
- No one-off colors (use palette)
- No custom spacing (use scale)
- No arbitrary decisions (document rationale)

#### 15.14.4 Scope

This subsection covers:

| Document | Purpose |
|----------|---------|
| User Research | Personas, jobs-to-be-done, success metrics |
| Information Architecture | Site map, navigation spec, screen inventory |
| User Flows | Step-by-step workflows for key tasks |
| Design Decisions | Design decisions |

**Note**: Cross-cutting concerns live in the design system, not here. This section is about the *process*, not the *artifacts*.

#### 15.14.5 Relationship to Other Sections

```
┌─────────────────────────────────────────────────────────────────┐
│                       web-ui-design                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────────────┐    ┌─────────────────────────────────┐ │
│  │ ui-design-process   │    │  Other Sections                 │ │
│  │  (THIS SECTION)     │    │                                 │ │
│  │                     │    │  frontend-foundation            │ │
│  │  • WHO uses it      │--->│  design-system                  │ │
│  │  • WHAT they see    │    │  navigation-components          │ │
│  │  • WHY it's there   │    │  feedback-patterns              │ │
│  │  • HOW it behaves   │    │  ...                            │ │
│  │                     │    │                                 │ │
│  │  DESIGN DECISIONS   │    │  TECHNICAL IMPLEMENTATION       │ │
│  └─────────────────────┘    └─────────────────────────────────┘ │
│                                                                 │
│  Design informs implementation, not the other way around.       │
└─────────────────────────────────────────────────────────────────┘
```

#### 15.14.6 Success Criteria

The UI design process is complete when:

1. **Every screen has a documented reason to exist** - traced to user workflow
2. **Every navigation item has a documented rationale** - not "because the API exists"
3. **Every screen has a wireframe** - reviewed before implementation
4. **Every screen has a specification** - states, interactions, content
5. **Every screen has Playwright tests** - written from spec, not reverse-engineered
6. **Visual QA passes** - design system compliance verified

#### 15.14.7 Quality Bar

WAVE's UI must meet the standard of professional engineering tools:

- **Calm confidence** - Not flashy, not cluttered; quietly competent
- **Information density** - Respect user expertise; don't oversimplify
- **Predictable behavior** - Same action, same result, everywhere
- **Zero confusion** - User always knows where they are and what to do next

The goal is not "a UI that works" but "a UI that test engineers love to use."
