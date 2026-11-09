# VerdeFlow UI Design Specification

## Overview

VerdeFlow is a commit-aware API energy profiling platform. The UI serves
three audiences simultaneously: the primary researcher (PhD-level, technical),
lab colleagues (technical), and hackathon developers (less technical). The
design must be immediately legible to all three without sacrificing depth.

**Visual style:** Clean and minimal — like Linear or Vercel. Flat surfaces,
no gradients, no shadows, generous whitespace, 0.5px borders. Everything
should feel native and purposeful.

---

## Design System

### Colors

All colors use CSS variables that auto-adapt to light and dark mode. Never
hardcode hex values in components.

```css
/* Backgrounds */
--color-background-primary        /* white / near-black */
--color-background-secondary      /* light gray surface */
--color-background-tertiary       /* page background */
--color-background-success        /* green tint */
--color-background-warning        /* amber tint */
--color-background-danger         /* red tint */
--color-background-info           /* blue tint */

/* Text */
--color-text-primary              /* primary body text */
--color-text-secondary            /* muted labels */
--color-text-tertiary             /* hints, placeholders */
--color-text-success              /* green text */
--color-text-warning              /* amber text */
--color-text-danger               /* red text */
--color-text-info                 /* blue text */

/* Borders */
--color-border-tertiary           /* default, very subtle */
--color-border-secondary          /* hover, emphasis */
--color-border-primary            /* strong borders */
--color-border-success
--color-border-danger
--color-border-info

/* Typography */
--font-sans
--font-mono
--font-serif

/* Radius */
--border-radius-md                /* 8px — most elements */
--border-radius-lg                /* 12px — cards */
--border-radius-xl                /* 16px — modals */
```

### Brand Green

VerdeFlow's primary brand color is `#1a6b3c`. Use it for:
- Primary buttons
- Active nav items (background tint, text color)
- Positive metric values (energy reductions)
- The logo icon background
- Tab active indicator underline
- Progress bars showing good coverage

Never use it for warnings, errors, or neutral UI chrome.

### Typography

- Font: system-ui / var(--font-sans)
- Body: 13–14px, weight 400
- Labels and secondary text: 11–12px, weight 400
- Headings / page titles: 18px, weight 500
- Section titles: 13–14px, weight 500
- Metric values (large): 20–24px, weight 500
- Code / commit hashes: var(--font-mono), 11–12px, background-secondary,
  padding 2px 6px, border-radius 4px
- Never use font-weight 600 or 700 — too heavy against the host UI
- Sentence case everywhere. Never Title Case. Never ALL CAPS.

### Spacing

- Page padding: 20px
- Section internal padding: 16px
- Card padding: 12px 14px (metric cards), 14px 16px (list rows)
- Gap between cards: 10–12px
- Gap between sections: 16px
- Table cell padding: 8–9px 12px

### Borders and Radius

- Default border: `0.5px solid var(--color-border-tertiary)`
- Emphasis border: `0.5px solid var(--color-border-secondary)`
- Cards and sections: `border-radius: var(--border-radius-lg)` (12px)
- Buttons, inputs, badges: `border-radius: 6–8px`
- Pills / badges: `border-radius: 99px`
- No box-shadows except focus rings (`box-shadow: 0 0 0 2px`)

### Buttons

Four variants:

```
default:  transparent bg, 0.5px border-secondary, hover: background-secondary
primary:  background #1a6b3c, color white, hover: #155e33
danger:   transparent bg, color text-danger, border border-danger
ghost:    no border, hover: background-secondary (used inside tables)
```

All buttons: `padding: 6px 12px`, `font-size: 12px`, `font-weight: 500`,
`border-radius: 6px`, `cursor: pointer`.

Small buttons (inside table rows): `padding: 3px 7px`, `font-size: 10–11px`.

### Form Inputs

```
input, select, textarea:
  width: 100%
  padding: 7px 10px
  font-size: 13px
  border: 0.5px solid var(--color-border-secondary)
  border-radius: 6px
  background: var(--color-background-primary)
  color: var(--color-text-primary)
  outline: none

  :focus
    border-color: var(--color-border-info)
    box-shadow: 0 0 0 2px var(--color-background-info)
```

Form labels: `font-size: 12px`, `font-weight: 500`, `color: text-secondary`,
`display: block`, `margin-bottom: 5px`.

### Badges / Status Pills

```
green  (completed, online, active):
  background: color-background-success  color: color-text-success

amber  (running, warning, busy):
  background: color-background-warning  color: color-text-warning

red    (failed, critical, danger):
  background: color-background-danger   color: color-text-danger

gray   (pending, acknowledged, offline):
  background: color-background-secondary  color: color-text-secondary

blue   (info, manual type, scope):
  background: color-background-info  color: color-text-info
```

All badges: `padding: 2px 8px`, `border-radius: 99px`, `font-size: 11px`,
`font-weight: 500`, `display: inline-flex`, `align-items: center`.

### Metric Cards

Used in groups of 4–6 to show key numbers at a glance.

```
background: color-background-secondary
border-radius: border-radius-md (8px)
padding: 12px 14px
no border

Structure:
  .mc-label    font-size: 11px, color: text-tertiary, margin-bottom: 6px
  .mc-value    font-size: 20px, font-weight: 500
  .mc-sub      font-size: 11px, color: text-tertiary, margin-top: 3px

Color overrides on .mc-value:
  positive trend (energy reduction): color #1a6b3c
  neutral:                           color text-primary (default)
  warning:                           color #b45309
  negative trend (regression):       color #b91c1c
```

### Tables

```
width: 100%, border-collapse: collapse, font-size: 12px

th:
  padding: 8px 12px
  font-size: 11px, font-weight: 500
  color: text-tertiary
  background: color-background-secondary
  border-bottom: 0.5px solid color-border-tertiary
  text-align: left

td:
  padding: 9px 12px
  border-bottom: 0.5px solid color-border-tertiary
  color: text-primary

tr:last-child td: border-bottom: none

Clickable rows (.clickable):
  cursor: pointer
  tr:hover td: background: color-background-secondary
```

### Section Containers

The main content grouping component:

```
background: color-background-primary
border: 0.5px solid color-border-tertiary
border-radius: 10px (slightly less than border-radius-lg)
margin-bottom: 16px
overflow: hidden

.section-header:
  display: flex, align-items: center, justify-content: space-between
  padding: 14px 16px
  border-bottom: 0.5px solid color-border-tertiary

  .section-title: font-size: 13px, font-weight: 500
  .section-actions: display flex, gap 6px
```

### Tabs

Used inside section containers to switch between content views.

```
.tabs:
  display: flex
  border-bottom: 0.5px solid color-border-tertiary

.tab:
  padding: 10px 16px
  font-size: 13px
  cursor: pointer
  color: text-secondary
  border-bottom: 2px solid transparent
  margin-bottom: -0.5px (overlaps section border)

  :hover: color text-primary

  .active:
    color: #1a6b3c
    border-bottom-color: #1a6b3c
    font-weight: 500

.tab-content:
  display: none
  padding: 16px

  .active: display: block
```

### Modals

```
.modal-bg (overlay):
  position: fixed, inset: 0
  background: rgba(0,0,0,0.35)
  display: none (flex when .open)
  align-items: center, justify-content: center
  z-index: 100

.modal (dialog):
  background: color-background-primary
  border: 0.5px solid color-border-secondary
  border-radius: 12px
  padding: 24px
  width: 420px, max-width: 90vw

  .modal-title: font-size 15px, font-weight 500, margin-bottom 16px

  .modal-footer:
    display: flex, justify-content: flex-end, gap: 8px, margin-top: 20px

Close on overlay click. Never close on modal body click.
```

### Status Dots

Used in the sidebar nav and device lists to show live connection state:

```
width: 8px, height: 8px, border-radius: 50%, flex-shrink: 0

.online:  background #22c55e
.busy:    background #f59e0b
.offline: background color-border-secondary
.error:   background color-text-danger
```

### Energy Trend Chart

The primary data visualization. Implemented as a bar chart (not a line chart)
because commit data is categorical, not continuous. Each bar represents one
run keyed to a commit hash.

```
Container: section with section-header containing metric selector dropdown
Chart area: height 180px, padding 16px
Bar container: display flex, align-items flex-end, gap 4px, height 140px

Bar groups (.bar-group):
  display: flex, flex-direction: column, align-items: center, flex: 1, gap: 4px

Individual bars (.bar):
  width: 100%, border-radius: 3px 3px 0 0
  min-width: 8px
  cursor: pointer
  transition: opacity .2s
  :hover: opacity 0.75

Colors:
  Normal run:     background #1a6b3c, varying opacity (0.65–1.0 based on recency)
  Regression run: background #b91c1c (solid, full opacity)
  Running:        background color-border-secondary (animated pulse optional)

Bar labels (.bar-label):
  font-size: 9px, color: text-tertiary
  max-width: 32px, overflow hidden, text-overflow ellipsis
  white-space: nowrap, text-align: center

Legend: display flex, gap 12px, padding 4px 16px 12px
  Items: 8px dot + 11px label text
```

---

## Application Shell

### Layout

```
Full viewport: display flex, height 100vh

Sidebar (200px wide, flex-shrink: 0):
  background: color-background-primary
  border-right: 0.5px solid color-border-tertiary
  display: flex, flex-direction: column

Content area (flex: 1, overflow: auto):
  display: flex, flex-direction: column

  Topbar (48px, flex-shrink: 0):
    background: color-background-primary
    border-bottom: 0.5px solid color-border-tertiary

  Main (flex: 1, overflow: auto):
    padding: 20px
```

### Sidebar Structure

```
1. Logo block (border-bottom)
   - 28px icon (green #1a6b3c bg, border-radius 6px, leaf SVG icon)
   - "VerdeFlow" 14px font-weight 500
   - "v0.1.0 · dev" 11px text-tertiary

2. Navigation (flex: 1, padding: 8px 0)
   Section labels: 10px, font-weight 500, text-tertiary,
                   letter-spacing 0.06em, ALL CAPS, padding 16px 16px 4px
   Nav items: 7px 16px padding, font-size 13px, display flex, gap 8px

   Active state: background color-background-secondary (tinted green),
                 color text-primary, font-weight 500
   Hover state:  background color-background-secondary, color text-primary

   Nav badge (for alert count):
     margin-left: auto, font-size 10px
     background color-background-danger, color color-text-danger
     padding 1px 6px, border-radius 99px

3. Footer (border-top)
   Organisation avatar (24px circle, green bg, white initials 10px)
   Org name 12px font-weight 500
   Tier label 12px text-secondary
```

### Navigation Items

```
MONITOR section:
  Dashboard    — grid icon
  Projects     — folder icon
  Runs         — clock/history icon
  Alerts       — triangle warning icon (+ count badge)

MANAGE section:
  Devices      — status dot (live online/offline indicator)
  Workloads    — list icon

SETTINGS section:
  Settings     — gear icon
```

### Topbar

```
Left: breadcrumb trail (font-size 13px, text-secondary, with "/" separators)
      Current page in text-primary, font-weight 500

Right: action buttons contextual to current page
       Always show "Trigger run" (default btn) + "New project" (primary btn)
       on most pages. Contextual overrides per page detailed below.
```

---

## Pages

### Dashboard (`/dashboard`)

**Purpose:** High-level health check. First thing users see. Must answer:
"Is everything working? Are there any problems? What ran recently?"

**Topbar actions:** "Trigger run" + "New project"

**Content:**

1. Metric cards row (5 cards):
   - Projects (count)
   - Total runs (count + delta this week)
   - Devices online (X/Y format, green if all online, amber if partial)
   - Open alerts (count, red if > 0)
   - Avg energy/req across all projects (amber, shows delta vs last week)

2. Recent runs section:
   Table columns: Project, Commit (code), Status (badge), Energy/req,
   Latency, Triggered, Time (relative)
   All rows clickable → run detail page
   Limit: 5–10 most recent
   "View all →" button in section header → /runs

3. Active regression alerts section:
   Shows unacknowledged alerts only
   Each row: severity badge + description + "Acknowledge" button
   "Manage →" in header → /alerts
   Hidden if no open alerts

**Empty state:** If no runs yet, show a getting-started card with three
steps: "Create a project → Configure endpoints → Install git hook"

---

### Projects (`/projects`)

**Purpose:** List and manage all projects.

**Topbar actions:** "+ New project" (primary, opens project modal)

**Content:**

1. Projects table:
   Columns: Name (font-weight 500), Target URL (text-secondary),
   Endpoints (count), Last run (relative time), Avg energy/req,
   Status (badge: active/alert/inactive), Actions (open button)
   All rows clickable → /projects/:id
   "alert" status badge when project has unacknowledged regressions

**Empty state:** Centered card: "No projects yet" + "Create your first
project" primary button.

---

### Project Detail (`/projects/:id`)

**Purpose:** The most important page. Deep-dive into one project's energy
profile across commits. Must support management of all project configuration.

**Topbar breadcrumb:** Projects / {project name}

**Topbar actions:** "Trigger run" + "Edit" + "Delete" (danger)

**Content:**

1. Page header:
   - Project name (page-title)
   - Subtitle: targetApiUrl · N endpoints · branch name

2. Metric cards (6 cards):
   - Energy / req (green, with delta vs last run below)
   - Mean latency (with p95 as sub-label)
   - Mean power (with peak as sub-label)
   - CO₂e per request
   - Coverage: "N / N endpoints" with green progress bar below
   - Open alerts (red count)

3. Energy trend chart (section):
   Header: "Energy trend across commits" + metric selector dropdown
   (options: Energy/req J, Mean latency ms, Mean power W, CO₂e g)
   + "Compare runs" button (future feature, can be disabled for now)
   Chart: bar chart, one bar per run, keyed to commit hash short form
   Bars colored green (normal) or red (regression)
   Clicking a bar → navigates to that run's detail page

4. Tabbed section (Run history / Endpoints / Workloads / Alerts):

   **Run history tab (default):**
   Table: Commit (code), Branch, Status (badge), Energy/req
   (red text if regression), Latency (red if regression), Power,
   Coverage (N/N), Triggered, View button
   Row click → run detail

   **Endpoints tab:**
   "+ Add endpoint" button (top right, opens endpoint modal)
   List rows: HTTP method badge (colored) + path (monospace, flex:1)
   + weight label + Edit button + Delete (×) button
   Allow drag-to-reorder weights (stretch goal)

   **Workloads tab:**
   "+ New workload" button
   Table: Name, Type (badge), Requests, Concurrency, Ramp-up,
   Edit + Delete buttons

   **Alerts tab:**
   Per-project alerts, same design as global alerts page
   Each row: severity + metric + values + Acknowledge button
   Acknowledged alerts shown faded (opacity 0.45)

---

### Run Detail (`/runs/:id`)

**Purpose:** Full breakdown of a single profiling run. Used to understand
exactly what happened — which endpoints used what energy, what the power
trace looked like, any regressions flagged.

**Topbar breadcrumb:** Runs / {commitHash short}

**Topbar actions:** "Compare with previous" button + status badge

**Content:**

1. Page header:
   - "Run · {commit hash code}" as title
   - Subtitle: project name · branch · commit message (truncated)

2. Metadata grid (12 items, 2-column):
   Energy/req (green), Total energy J, Mean power W, Peak power W,
   Mean latency ms, p95 latency ms, Throughput req/s, CO₂e total g,
   CPU mean %, Memory mean MB, Errors (N/total), Device name

   Each meta item:
     background color-background-secondary, border-radius 6px,
     padding 10px 12px
     .meta-key: 10px, text-tertiary, uppercase, letter-spacing 0.05em
     .meta-val: 13px, font-weight 500

3. Per-endpoint breakdown section:
   Table: Endpoint (method badge + path code), Energy/req, Mean latency,
   p95, Requests, Errors
   Aggregate row (whole-run summary) shown first if available

4. Raw power timeline (future — placeholder section for now):
   "Raw power trace available when PowerJoular is connected"
   Shown as a gray placeholder card

---

### Runs (`/runs`)

**Purpose:** Cross-project run history with filtering. Used to find a
specific run or audit energy trends over time.

**Topbar actions:** "Trigger run" (primary)

**Content:**

1. Filter row:
   Select: All projects / specific project
   Select: All statuses / Completed / Failed / Running / Cancelled
   Select: Last 7 days / Last 30 days / All time
   All filters update the table in real time (client-side if data loaded,
   or refetch on change)

2. Runs table:
   Columns: Project, Commit (code), Status (badge), Energy/req
   (red if regression), Latency (red if regression), Power,
   Device, Triggered, View button
   Row click → run detail
   Regression rows: energy and latency cells colored red + font-weight 500

---

### Alerts (`/alerts`)

**Purpose:** Triage energy regressions. Acknowledge and dismiss.
Manage the thresholds that trigger them.

**Topbar actions:** "Configure thresholds" button (opens threshold modal)

**Content:**

1. Open alerts section (count in title):
   Each alert row:
   - Severity badge (critical red / warning amber)
   - Metric name (underscores replaced with spaces)
   - Values: "baseline → current" format
   - Project name + commit hash + relative time
   - "Acknowledge" button
   Row fades to opacity 0.45 after acknowledge (optimistic UI,
   then removes on next load)

2. Acknowledged section:
   Same row design but all faded, no acknowledge button
   Show last 10 acknowledged, "show more" link

**Threshold modal:**
Table with 4 rows (one per tracked metric):
energy_per_request_j, mean_latency_ms, mean_power_w, mean_cpu_pct
Two editable inputs per row: warning % threshold, critical % threshold
Default values: warning 10–20%, critical 25–50% (vary by metric)

---

### Devices (`/devices`)

**Purpose:** Manage profiling agents. See which are online, register new
ones, revoke compromised keys.

**Topbar actions:** "+ Register device" (primary, opens device modal)

**Content:**

1. Devices list (section):
   Each device row:
   - Status dot (8px circle, green/amber/gray/red)
   - Device name (font-weight 500)
   - Subtitle: platform · agent version · "last seen X ago"
   - Status badge (online/busy/offline/error)
   - "View key" button (shows key prefix — never full key)
   - "Revoke" button (danger)

**Register device modal:**
Fields: Device name, Description (optional)
On submit: shows the generated API key in a green monospace reveal box.
Important copy: "This key will only be shown once. Store it securely."
After reveal: show copy-to-clipboard button, change submit button to "Close"

---

### Workloads (`/workloads`)

**Purpose:** Manage workload configurations across all projects.

**Topbar actions:** "+ New workload" (primary)

**Content:**

1. Workloads table:
   Columns: Name, Project, Type (badge: manual/stress/openapi),
   Requests, Concurrency, Ramp-up, Last used, Edit + Delete buttons

**New workload modal:**
Fields: Name, Project (select), Type (select: manual/stress/openapi),
Total requests (number), Concurrency (number), Ramp-up seconds (number)

---

### Settings (`/settings`)

**Purpose:** Org config, API key management, GitHub integration,
member management.

**Topbar actions:** none (settings page manages itself)

**Content (four sections):**

1. Organisation:
   Fields: Name, Slug (auto-generated, editable), Tier (select)
   "Save changes" button

2. API keys:
   Table: Name, Prefix (code), Scopes (badges), Last used, Expires, Revoke
   "+ Create key" in header → apikey modal
   Apikey modal: Name, Scopes (checkboxes for each scope),
   Expiry date (optional)
   On create: shows full key once in green reveal box with copy button

3. GitHub integration:
   Fields: GitHub App ID, Webhook secret (password input), Private key path
   "Save GitHub config" button
   Help text: link to GitHub App setup docs

4. Members:
   Table: Name, Email, Role (badge: admin/member/viewer), Joined, Remove
   "+ Invite member" in header
   Current user row has no Remove button
   Role is editable via inline select

---

## Modals Reference

### New project modal
Fields: Project name*, Target API URL*, Repository URL, Default branch
(default: main), GitHub repo full name (org/repo), OpenAPI spec URL (optional)
Footer: Cancel + "Create project" primary

### Trigger run modal
Fields: Project (select)*, Workload (select, shows name + request count)*,
Device (select, shows name + status)*, Commit hash (optional, default HEAD),
Branch (optional)
Footer: Cancel + "Queue run" primary

### Add endpoint modal
Fields: Method (select GET/POST/PUT/PATCH/DELETE)*, Path*, Request body
(textarea, JSON), Weight 0–100 (default 50), Description
Footer: Cancel + "Add endpoint" primary

### Register device modal
Fields: Device name*, Description
On submit: API call, then show key reveal div (green monospace, border
color-border-success), show "Copy key" button
Footer changes: Cancel → hidden, button changes to "Close"

### Create API key modal
Fields: Name*, Scopes (multi-checkbox), Expires (date, optional)
On submit: show key reveal, same pattern as device modal
Footer: Cancel + "Create key" primary → "Close" after reveal

### New workload modal
Fields: Name*, Type (select)*, Total requests*, Concurrency*, Ramp-up (s)
Footer: Cancel + "Create workload" primary

### Configure thresholds modal
Table: 4 metrics × 2 inputs (warning %, critical %)
Footer: Cancel + "Save thresholds" primary

---

## API Integration

Every page loads real data from the gateway API. The API client is at
`src/lib/api.ts` and exports typed functions for every endpoint.

**Authentication:** JWT stored in `localStorage` as `vf_token`. Added as
`Authorization: Bearer <token>` header on every request. If a request
returns 401, redirect to `/login`.

**Loading states:** Show a muted centered text "Loading..." while fetching.
Never block the entire page — load sections independently.

**Error states:** Show inline error messages inside the relevant section
when an API call fails. Use `color-text-danger` with a brief message.
Never crash the whole page on a single failed request.

**Optimistic UI:** For acknowledge (alerts) and delete operations, update
the UI immediately and revert if the API call fails.

**Real-time updates:** Poll the runs list every 10 seconds when any run
has status "running" or "queued". Update status badges and metrics in place.
Stop polling when all runs are in terminal states (completed/failed/cancelled).

---

## Routing

```
/login                    LoginPage              (unauthenticated)

── Shared (all authenticated roles) ──────────────────────────────────────
/dashboard                DashboardPage
/projects                 ProjectsPage
/projects/:id             ProjectDetailPage
/runs                     RunsPage
/runs/:id                 RunDetailPage
/alerts                   AlertsPage

── Developer + Org admin + Super admin ────────────────────────────────────
/devices                  DevicesPage
/workloads                WorkloadsPage

── Org admin + Super admin ────────────────────────────────────────────────
/settings                 SettingsPage           (org config, keys, GitHub,
                                                  members — scoped to own org)

── Super admin only ───────────────────────────────────────────────────────
/admin                    AdminDashboardPage      (cross-org overview)
/admin/organisations      OrganisationsPage       (list + create + delete orgs)
/admin/organisations/:id  OrganisationDetailPage  (manage one org)
/admin/users              AllUsersPage            (cross-org user list)
```

All routes except `/login` require authentication. Redirect to `/login`
if no valid JWT is found. After login, redirect to `/dashboard`.

Accessing a route above your role level returns a 403 page inline —
never a blank page or a crash. Super admin routes (`/admin/*`) are
completely invisible to all other roles: no nav items, no links,
no API calls.

---

## Component File Structure

```
src/
├── App.tsx                           routing + role guards
├── main.tsx                          entry
├── lib/
│   ├── api.ts                        typed API client (already exists)
│   └── auth.tsx                      AuthProvider + useAuth + useRole
├── hooks/
│   ├── useRuns.ts                    polling hook for run status
│   ├── useProject.ts                 project data + runs
│   └── useRole.ts                    current user role helpers
├── components/
│   ├── layout/
│   │   ├── Layout.tsx                shell — renders correct sidebar by role
│   │   ├── Sidebar.tsx               org-user sidebar (developer/admin/viewer)
│   │   ├── AdminSidebar.tsx          super admin sidebar (entirely different nav)
│   │   └── Topbar.tsx                breadcrumb + role-aware action buttons
│   ├── guards/
│   │   ├── RequireAuth.tsx           redirect to /login if no token
│   │   ├── RequireRole.tsx           render 403 inline if role insufficient
│   │   └── RoleGate.tsx              conditionally render children by role
│   ├── ui/
│   │   ├── Badge.tsx                 status badges
│   │   ├── Button.tsx                button variants
│   │   ├── Modal.tsx                 modal wrapper
│   │   ├── MetricCard.tsx            single metric card
│   │   ├── SectionCard.tsx           section container
│   │   ├── Table.tsx                 base table styles
│   │   └── StatusDot.tsx             online/offline dot
│   └── charts/
│       └── EnergyTrendChart.tsx      bar chart component
├── pages/
│   ├── auth/
│   │   └── LoginPage.tsx
│   ├── dashboard/
│   │   └── DashboardPage.tsx
│   ├── projects/
│   │   ├── ProjectsPage.tsx
│   │   └── ProjectDetailPage.tsx
│   ├── runs/
│   │   ├── RunsPage.tsx
│   │   └── RunDetailPage.tsx
│   ├── devices/
│   │   └── DevicesPage.tsx            hidden from viewer role
│   ├── workloads/
│   │   └── WorkloadsPage.tsx          hidden from viewer role
│   ├── alerts/
│   │   └── AlertsPage.tsx
│   ├── settings/
│   │   └── SettingsPage.tsx           org admin + super admin only
│   └── admin/                         super admin only — entire folder
│       ├── AdminDashboardPage.tsx
│       ├── OrganisationsPage.tsx
│       ├── OrganisationDetailPage.tsx
│       └── AllUsersPage.tsx
└── types/                             re-export from @verdeflow/domain if needed
```

---

## Key Interaction Patterns

**Navigation:** Single-page app. All nav transitions are instant (no
loading spinners for navigation itself — data loads independently
after mount).

**Tables with clickable rows:** `cursor: pointer`, hover background.
Clicking anywhere on the row navigates to the detail view. Action buttons
inside cells use `event.stopPropagation()` to prevent row click.

**Regression highlighting:** Any metric value that represents a regression
vs the previous run should be colored `#b91c1c` (danger red) with
`font-weight: 500`. This applies to cells in tables, not to chart bars
(bars use the red bar color instead).

**Relative timestamps:** Use `date-fns formatDistanceToNow()` for all
timestamps displayed to the user. Show absolute ISO timestamp in a
`title` attribute for hover tooltip.

**Empty states:** Every list/table must have an empty state. Centered,
muted 13px text, optionally a primary CTA button. Never show an empty
table with only headers.

**Keyboard navigation:** Modals close on Escape key. Tab focus stays
within open modal (focus trap). Form submits on Enter in single-line inputs.

**Delete confirmations:** For destructive actions (delete project, revoke
device key), show a second confirmation step inside the modal before
executing. Never delete on a single click.

---

## Roles and Permissions

VerdeFlow has four roles arranged in a strict hierarchy. Each role is a
superset of the one below it — a developer can do everything a viewer can,
an org admin can do everything a developer can, and so on.

```
super_admin  >  org_admin  >  developer  >  viewer
```

Roles are stored in the JWT payload and enforced both client-side (UI
visibility) and server-side (API). Client-side enforcement is UX only —
never trust it for security. The server is the source of truth.

---

### Role definitions

**viewer**
Read-only access to their organisation's data. Cannot change anything.
Intended for stakeholders who need to see energy reports without touching
configuration. Typical user: a supervisor reviewing research results, or a
hackathon judge.

**developer**
The default role for active users. Can do everything a viewer can, plus
trigger runs, manage their own workloads, and register/manage devices.
Cannot touch org-level settings, API keys, member management, or billing.
Typical user: a PhD researcher, lab member, or hackathon participant.

**org_admin**
Full control over their organisation. Can do everything a developer can,
plus manage org settings, create/revoke API keys, invite/remove members,
configure GitHub integration, and set alert thresholds. Cannot see or
touch any other organisation's data. Typical user: the lab PI or primary
account owner.

**super_admin**
Platform-level administrator. Has a completely separate UI layer (/admin/*)
that is invisible to all other roles. Can create, edit, suspend, and delete
organisations. Can view cross-org aggregate stats. Can manage any user in
any org. Cannot impersonate users or access system config. Typical user:
the VerdeFlow platform operator (i.e. you, for research deployments at
LUT or Lagos).

---

### What each role sees in the UI

#### Sidebar navigation

**viewer sidebar:**
```
MONITOR
  Dashboard
  Projects
  Runs
  Alerts

[org footer — name + "Viewer" label]
```

**developer sidebar:**
```
MONITOR
  Dashboard
  Projects
  Runs
  Alerts

MANAGE
  Devices
  Workloads

[org footer — name + "Developer" label]
```

**org_admin sidebar:**
```
MONITOR
  Dashboard
  Projects
  Runs
  Alerts

MANAGE
  Devices
  Workloads

SETTINGS
  Settings

[org footer — name + "Admin" label]
```

**super_admin sidebar** (completely different shell — different background
tint to make it visually unmistakable that this is the admin layer):
```
PLATFORM
  Overview          (cross-org dashboard)
  Organisations     (list all orgs)
  Users             (list all users cross-org)

SUPER ADMIN

[footer — "Platform Admin" label, no org name]
```

The super admin sidebar uses a slightly different background — use
`#0f1f2e` (dark navy) in light mode and keep the same dark mode color.
This makes it immediately obvious when you are in the admin layer vs the
normal org UI. Add a small "ADMIN" pill badge in the topbar to reinforce
this.

---

### Page-level permission matrix

| Page | viewer | developer | org_admin | super_admin |
|---|---|---|---|---|
| Dashboard | ✓ read | ✓ read | ✓ read | ✓ (own org if switched, else /admin) |
| Projects list | ✓ read | ✓ read | ✓ + create/delete | ✓ |
| Project detail | ✓ read | ✓ + trigger run | ✓ + edit/delete | ✓ |
| Runs list | ✓ read | ✓ read | ✓ read | ✓ |
| Run detail | ✓ read | ✓ read | ✓ read | ✓ |
| Alerts | ✓ read | ✓ + acknowledge | ✓ + configure thresholds | ✓ |
| Devices | ✗ hidden | ✓ + register/revoke | ✓ + register/revoke | ✓ |
| Workloads | ✗ hidden | ✓ + create/edit/delete | ✓ + create/edit/delete | ✓ |
| Settings | ✗ hidden | ✗ hidden | ✓ full access | ✓ |
| /admin/* | ✗ 403 | ✗ 403 | ✗ 403 | ✓ full access |

---

### UI differences per role

#### Viewer — read-only surfaces

- All "Trigger run", "+ New project", "+ Add endpoint", "Edit", "Delete"
  buttons are hidden. Not disabled — fully absent from the DOM.
- Devices and Workloads nav items are not rendered.
- Settings nav item is not rendered.
- Alert "Acknowledge" button is hidden — can read alerts but not act on them.
- Project detail page: no "Edit" or "Delete" in topbar actions. No tab for
  Endpoints or Workloads (read-only, no management). Only "Run history" and
  "Alerts" tabs visible.
- Topbar actions area is empty (no buttons).
- Sidebar org footer shows role badge: `Viewer` in gray.
- The entire page feels read-only — muted "View only" label in topbar
  right side: `font-size: 11px, color: text-tertiary, font-style: italic`.

#### Developer — standard working UI

- Full access to all Monitor and Manage pages.
- Can trigger runs, manage devices, manage workloads.
- Cannot access Settings. The nav item is absent.
- In Project detail: all 4 tabs visible. "Trigger run" button in topbar.
  "Edit project" and "Delete project" are absent (only org_admin can do
  these). No "Configure thresholds" on Alerts page — can acknowledge only.
- Topbar shows normal action buttons scoped to their permissions.
- Sidebar org footer: `Developer` badge in blue.

#### Org admin — full org control

- Everything developer can do, plus:
- Settings nav item is visible and accessible.
- "Edit project" and "Delete project" buttons appear in project detail topbar.
- "Configure thresholds" button appears on Alerts page.
- "Invite member" and member role editing in Settings > Members.
- "+ Create key" in Settings > API keys.
- Sidebar org footer: `Admin` badge in green.

#### Super admin — platform layer

Super admin gets a completely different application shell when they log in.
They do NOT see the normal org sidebar. They land on `/admin` directly.

**Admin dashboard (`/admin`):**
- Cross-org summary cards: Total orgs, Total users, Total runs (all time),
  Active devices (all orgs), Platform energy total (all runs, all orgs).
- Orgs table: Name, Slug, Tier, Users, Projects, Runs, Created, Status,
  Actions (Manage, Suspend, Delete).
- Recent activity feed: last 20 events across all orgs (org created,
  user invited, run triggered, regression alert fired).

**Organisations page (`/admin/organisations`):**
- Table: all orgs with search and filter by tier.
- "+ Create organisation" primary button → create org modal.
- Each row: org name, slug, tier badge, user count, project count,
  device count, created date, status (active/suspended), Actions column
  with "Manage" (→ org detail), "Suspend/Unsuspend" (toggle), "Delete"
  (danger, with confirmation).

**Organisation detail page (`/admin/organisations/:id`):**
- Header: org name + tier badge + status badge + "Suspend" / "Delete" danger
  buttons.
- Tabs: Overview, Projects, Devices, Members, API keys.
  - Overview: metric cards (users, projects, runs, devices, energy total).
  - Projects: list of all projects in this org, each clickable.
  - Devices: list of all devices in this org.
  - Members: full member list with role editing and remove capability.
  - API keys: list of all API keys with revoke capability.
- "Edit organisation" button: can change name, slug, and tier.
- Tier change takes effect immediately — no confirmation modal needed since
  this only changes capabilities, not deletes data.

**All users page (`/admin/users`):**
- Cross-org user table: Name, Email, Org (with link), Role, Last active,
  Created, Actions (Edit role, Remove from org).
- Search bar filters by name or email.
- Filter by org, filter by role.
- Clicking a user shows their details inline (expandable row), not a
  separate page — keeps the flow fast.

**Super admin topbar:**
- Left: breadcrumb shows "Platform admin / {page name}" in the same style.
- Center: a subtle `ADMIN` pill badge — `background: rgba(185,28,28,0.1),
  color: #b91c1c, font-size: 10px, font-weight: 600, padding: 2px 8px,
  border-radius: 99px` — always visible in the topbar when in admin mode.
- Right: "← Back to org view" ghost button if the super admin was previously
  in an org context (stored in sessionStorage). This lets them switch
  between the admin layer and a specific org's dashboard without logging
  out and back in.

---

### Role-switching for super admin

Super admin can optionally "enter" a specific organisation's context to see
what that org's users see. This is NOT impersonation — they remain logged in
as themselves, but the UI shows that org's data through the normal org shell.

Trigger: "View as org" button on the Organisation detail page.
Behaviour: stores `admin_viewing_org_id` in sessionStorage, switches to the
normal org sidebar shell, shows the org's data. A persistent top banner
appears: `background: rgba(185,28,28,0.06), border-bottom: 0.5px solid
rgba(185,28,28,0.2)`, text: "Viewing as LUT University · Exit org view →"
(click to return to /admin). This banner cannot be dismissed until they
exit org view.

---

### RoleGate component

Use a `<RoleGate minRole="org_admin">` wrapper component throughout the UI
to conditionally render elements. This is the single source of truth for
client-side role enforcement.

```tsx
// Usage examples
<RoleGate minRole="developer">
  <button>Trigger run</button>
</RoleGate>

<RoleGate minRole="org_admin">
  <button>Delete project</button>
</RoleGate>

<RoleGate minRole="super_admin">
  <AdminSidebar />
</RoleGate>

// Role order for comparison
const ROLE_RANK = { viewer: 0, developer: 1, org_admin: 2, super_admin: 3 };

function RoleGate({ minRole, children }) {
  const { role } = useRole();
  if (ROLE_RANK[role] < ROLE_RANK[minRole]) return null;
  return children;
}
```

**Rule:** Never use `role === 'org_admin'` comparisons directly in JSX.
Always use `<RoleGate>` or the `useRole()` hook's helper functions
(`canManageOrg()`, `canTriggerRuns()`, `isSuperAdmin()` etc.). This keeps
role logic centralised and easy to update.

---

### JWT payload shape

The JWT must carry enough information for the client to make role decisions
without an extra API call on every route change.

```json
{
  "sub": "user-uuid",
  "orgId": "org-uuid",
  "orgName": "LUT University",
  "orgSlug": "lut-university",
  "role": "org_admin",
  "email": "mbanyick.joof@lut.fi",
  "name": "Mbanyick Joof",
  "tier": "enterprise",
  "iat": 1234567890,
  "exp": 1234567890
}
```

`role` is one of: `viewer`, `developer`, `org_admin`, `super_admin`.

Super admin JWTs have `orgId: null` and `orgName: null` since they do not
belong to a single organisation.

---

### 403 page (inline)

When a user navigates to a route above their role level (e.g. a developer
hits `/settings` directly), render this inline — not a full-page redirect:

```
[centered, padding 48px]
Lock icon (SVG, 32px, text-tertiary)
"Access restricted"           [16px, font-weight 500, text-primary]
"You need {minRole} access    [13px, text-secondary]
 to view this page."
[optional: "Request access" button if org_admin context is known]
```

Never show a blank page, never crash, never show raw error text.
