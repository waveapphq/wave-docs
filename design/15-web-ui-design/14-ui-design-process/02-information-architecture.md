# Information Architecture

> **Design Principle**: Navigation reflects the user's mental model, not the API structure.

---

## 1. Entity Hierarchy

Users think in terms of a natural hierarchy that mirrors their workflow:

```
Regression (release mapping)
└── Suite (feature grouping)
    └── Test (validation)
        └── Canvas (workflow of Blocks & Plugins)
```

### Key Relationships

| Entity | Purpose | Contains | Belongs To |
|--------|---------|----------|------------|
| **Regression** | Maps Suites to a release version (e.g., "26.1") | Suites (linked) | - |
| **Suite** | Groups related Tests by feature area | Tests | Regression (0..n) |
| **Test** | Defines a specific validation | Workflow canvas | Suite (0..n) |
| **Function** | Reusable composed workflow | Plugins, Functions | Library |
| **Plugin** | Atomic operation (HTTP, SSH, SQL, etc.) | - | Library |
| **Profile** | Connection configuration bundle | - | Library |

### Important Notes

- A **Suite** can exist independently (not linked to any Regression)
- A **Suite** can be linked to multiple Regressions
- A **Test** can belong to multiple Suites (playlist model - suites reference tests, not own them)
- A **Test** can also exist independently without any suite membership
- A **Test** workflow can contain any mix of Blocks and Plugins

### Many-to-Many Test-Suite Model

Tests are independent entities. Suites reference tests like playlists. This enables:

- **Shared prep/teardown tests** - A "Deploy Service" test can be referenced by multiple suites
- **Mini-suites** - A quick smoke suite can reference a subset of a larger regression suite's tests
- **No duplication** - One test definition, many suite memberships. Updating the test updates it everywhere.

Tags provide additional cross-cutting organization alongside suite membership:

| Mechanism | Purpose | Example |
|-----------|---------|---------|
| **Suite membership** | Group tests by feature area or purpose | "TLS Validation", "API Health", "Quick Smoke" |
| **Tags/Labels** | Cross-cutting categorization | `tag:smoke`, `tag:critical`, `tag:api` |
| **Filter at run time** | Run subset by tags | "Run Regression 26.1, only `tag:smoke`" |
| **Custom selection** | Cherry-pick specific tests | Checkbox selection before run |

---

## 2. Site Map

### Screen Inventory

```
WAVE
├── Dashboard                          [dashboard]
├── Executions                         [list → detail]
│
├── BUILD
│   ├── Regressions                    [list → detail]
│   │   └── Regression Detail          (linked Suites, run controls)
│   ├── Suites                         [list → detail]
│   │   └── Suite Detail               (Tests table, metadata)
│   │       └── Test Canvas            (workflow builder)
│   ├── Tests                          [list → canvas]
│   │   └── Test Canvas                (workflow builder)
│   └── Workbench                      [canvas]
│       └── Standalone Canvas          (Test building)
│
├── LIBRARY
│   ├── Functions                      [list → detail]
│   │   └── Function Detail            (versions, metadata)
│   └── Profiles                       [list → detail/edit]
│       └── Profile Detail             (connection config)
│
├── RESULTS
│   ├── Reports                        [list → detail]
│   │   └── Report Detail              (execution summary, drill-down)
│   └── Artifacts                      [list → detail/download]
│       └── Artifact Detail            (download)
│
└── ACCOUNT
    ├── Settings                       [form]
    └── User                           [page]
```

### Screen Types

| Type | Pattern | Examples |
|------|---------|----------|
| **Dashboard** | Summary cards, recent items, status overview | Dashboard |
| **List** | Table with search, filter, columns, actions | Suites, Tests, Functions, Reports |
| **Detail** | Header + metadata + related content | Suite Detail (with Tests table) |
| **Canvas** | Visual workflow builder | Test Canvas, Workbench |
| **Form** | Configuration inputs | Settings, Profile Edit |

---

## 3. Navigation Structure

### 3.1 Primary Navigation (Sidebar)

```
┌──────────────────────────────────────┐
│                                      │
│  📊  Dashboard                       │
│  ▶️  Executions                      │
│                                      │
│  ─────────  BUILD  ──────────        │
│                                      │
│  📋  Regressions                     │
│  📁  Suites                          │
│  🧪  Tests                           │
│  ⚡  Workbench                       │
│                                      │
│  ─────────  LIBRARY  ────────        │
│                                      │
│  🧩  Functions                       │
│  📡  Profiles                        │
│                                      │
│  ─────────  RESULTS  ────────        │
│                                      │
│  📈  Reports                         │
│  📦  Artifacts                       │
│                                      │
│  ─────────  ACCOUNT  ────────        │
│                                      │
│  ⚙️  Settings                        │
│  👤  User                            │
│                                      │
└──────────────────────────────────────┘
```

### 3.2 Navigation Rationale

| Item | Section | Why It Exists | Primary User |
|------|---------|---------------|--------------|
| **Dashboard** | Top | Landing page, quick pulse, "what needs attention" | Everyone |
| **Executions** | Top | Live/recent runs, operational monitoring | Will, Valerie, Edward |
| **Regressions** | BUILD | Browse/manage release mappings | Valerie |
| **Suites** | BUILD | Browse/manage feature groupings, primary entry for Tests | Will |
| **Tests** | BUILD | Cross-cutting view, search/filter across all Tests | Will, Ashley |
| **Workbench** | BUILD | Blank canvas for standalone creation | Ashley |
| **Functions** | LIBRARY | Browse/add reusable composed workflows | Ashley, Will |
| **Profiles** | LIBRARY | Browse/add connection configurations | Ashley |
| **Reports** | RESULTS | Historical analysis, the "Excel killer" | Valerie, Edward |
| **Artifacts** | RESULTS | Generated files, logs, downloads | Will, Valerie |
| **Settings** | ACCOUNT | System configuration | Everyone |

### 3.3 Two Access Paths for Tests

Tests can be accessed two ways, serving different use cases:

**1. Through Suite (hierarchical)**
```
Suites → Healthchecks → Tests table → API Health 200
```
- Context-aware: Shows only Tests in that Suite
- Natural browse path for working within a feature area

**2. Through Tests nav (flat)**
```
Tests → Filter/Search → API Health 200
```
- Cross-cutting: Shows ALL Tests across ALL Suites
- For queries like "all OpenSearch tests"

### 3.4 Tests Page Filters

| Filter | Purpose | Example |
|--------|---------|---------|
| Name/Description | Find by text | "opensearch", "healthcheck" |
| Suite | Scope to feature area | Suite = "TLS Validation" |
| Labels/Tags | Find by category | tag:critical, tag:smoke |
| Status | Find by result | Passing, Failing, Never Run |
| Last run date | Find by recency | Last 7 days, Last 30 days |

---

## 4. Secondary Navigation

### 4.1 Breadcrumbs

Breadcrumbs are **contextual** - they show how you navigated, not a canonical path.

| Navigation Path | Breadcrumb |
|-----------------|------------|
| Suites → Healthchecks → API Health 200 | `Suites / Healthchecks / API Health 200` |
| Tests → API Health 200 | `Tests / API Health 200` |
| Executions → #12345 → API Health 200 | `Executions / #12345 / API Health 200` |

### 4.3 Back Behavior

- Browser back button works naturally (history-based)
- Breadcrumb clicks navigate to that level
- Escape key in modals/panels closes them

---

## 6. List View Pattern

All entity lists follow a consistent pattern:

```
┌────────────────────────────────────────────────────────────────────────┐
│  {Entity Name}                                    [+ New {Entity}]     │
├────────────────────────────────────────────────────────────────────────┤
│  🔍 Search...     [Filter ▾]     [Columns ▾]                           │
├────────────────────────────────────────────────────────────────────────┤
│  Name              │ Version │ Count  │ Status  │ Actions              │
├────────────────────┼─────────┼────────┼─────────┼──────────────────────┤
│  {Item Name}       │ x.y.z   │ N      │ ✓ Pass  │ ▶  ⋮                 │
│  {Item Name}       │ x.y.z   │ N      │ ✗ Fail  │ ▶  ⋮                 │
└────────────────────┴─────────┴────────┴─────────┴──────────────────────┘
```

### Pattern Components

| Component | Purpose |
|-----------|---------|
| **Header** | Title + primary action button |
| **Search** | Filter by name/description |
| **Filter dropdown** | Status, date range, tags, etc. |
| **Columns dropdown** | Show/hide secondary columns |
| **Table** | Primary + secondary (toggleable) columns |
| **Quick action (▶)** | Most common action (usually Run) |
| **More menu (⋮)** | Edit, Duplicate, Delete, Export |

### List Configurations

| List | Primary Columns | Quick Action |
|------|-----------------|--------------|
| Regressions | Name, Version, Suites count, Status | ▶ Run |
| Suites | Name, Version, Tests count, Status | ▶ Run |
| Tests | Name, Version, Status | ▶ Run |
| Profiles | Name, Type, Environment | Edit |
| Executions | ID, Entity, Started, Duration, Status | View |
| Reports | Date, Regression, Pass Rate, Status | View |
| Artifacts | Name, Type, Size, Date | ⬇ Download |

---

## 7. Cross-Linking

The system supports contextual navigation between related entities:

### Examples

**Execution → Source Test**
```
Execution #12345
├── Test: "API Health 200" → [Open Test]
└── Click navigates to Test canvas
```

**Report → Drill-down**
```
Report "Regression 26.1 - 2024-01-15"
├── Suite "Healthchecks" (12/12 passed) → [View Details]
└── Click expands to show individual Test results
```

---

## 8. JTBD Validation

Every job has a clear screen path:

| Job | Screen Path |
|-----|-------------|
| "Create a test for [feature]" | Tests → + New Test → Canvas |
| "Debug why this test failed" | Executions → {Execution} → {Test} → Canvas (debug mode) |
| "Run a regression before release" | Regressions → {Regression} → ▶ Run |
| "Find a specific test" | Tests → filter |
| "Organize tests into groups" | Suites → + New Suite OR drag Tests between Suites |
| "See execution history" | Reports → filter by date/regression |

---
