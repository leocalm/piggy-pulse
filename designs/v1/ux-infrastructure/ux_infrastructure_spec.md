# PiggyPulse — UX Infrastructure Specification

Status: **Canonical (merged v1 + v2)**
Product posture: **Professional control system** with **reflective, calm** tone (warm, not "coach-y").

This document formalizes the *shell* that every screen lives inside and the *cross-cutting primitives* that every screen uses:
- Navigation + layout grid system
- Notifications + overlays (modal/drawer/confirm) primitives
- A unified **state rendering contract** (locked/error/loading/empty/active) that all pages and cards must follow

---

## 0. Non-Negotiable Principles

### 0.1 Calm Reflection (interaction posture)
- **Neutral, factual microcopy.** No blame, no panic, no guilt.
- **Warm, minimal reassurance** is OK ("You can try again.") — avoid coaching ("You got this!").
- **Transparency beats surprise.** When user actions change derived outputs, surface that change calmly.

### 0.2 Structural correctness over flourish
- Layout is stable across pages; UI behavior is predictable.
- States render consistently; actions appear where users expect them.
- No new patterns unless explicitly added here.

### 0.3 Accessibility baseline
- Full keyboard support (Tab order, Escape closes overlays, focus visible).
- Focus trap in modal/drawer.
- Screen reader labels for nav, drawers, dialogs, and notifications.
- Color is **never** the only carrier of meaning (use text + icon + structure).

---

## 1. Navigation & Layout Grid System

### 1.1 Layout shells

**A. App Layout (Authenticated)**
- Sidebar (fixed 240px desktop)
- Top bar (structural, right-aligned utilities)
- Page header (H1 title, subtitle, primary CTA)
- Content grid (max 1100px)
- Mobile bottom nav (viewport < desktop breakpoint):
  - Dashboard
  - Transactions
  - Periods
  - More (opens drawer)
- 8px spacing system

**B. Focus Layout (Auth / Onboarding)**
- Centered container
- No sidebar
- Max-width 480–720px depending on context

**C. Overlay Layer**
- Modal (blocking)
- Drawer (mobile)
- Toast layer (non-blocking)

### 1.2 Sidebar structure (v1)

Core:
- Dashboard
- Transactions
- Periods

Structure:
- Accounts
- Categories
- Vendors

Session:
- Settings
- Logout

Rules:
- Active item = left 4px accent bar
- No icon-only mode (v1)
- Collapsible reserved for v2
- `(Log out)` in sidebar footer is subtle; profile menu also contains Logout

Mobile navigation (v1):
- Bottom nav items are fixed to: Dashboard, Transactions, Periods, More
- More opens a bottom drawer with: Accounts, Categories, Vendors, Settings

### 1.3 Grid tokens

#### Breakpoints
- **Mobile:** 0–759px
- **Tablet:** 760–1099px
- **Desktop:** 1100px+

#### Content max widths
- App shell max width: **1100px** content container
- Dense pages (tables): may use full width inside container; keep outer margins consistent

#### Spacing scale
- xs: 6 / s: 8 / m: 12 / l: 16 / xl: 24 / 2xl: 32
- Allowed values: 8 / 12 / 16 / 20 / 24 / 32 / 40
- Mobile padding: 16px horizontal

#### Radius & borders
- Standard card radius: **14–16px**
- Shell/card border: **1px** soft border
- "Disabled/locked" border: **1px dashed**

### 1.4 Page header standard
Every management page uses the same header contract:
- Title (H1)
- Subtitle (one sentence)
- Primary action (Create / Add) aligned right on desktop; compact "+" on mobile
- Optional: secondary actions (e.g., "Edit schedule")

Header behavior:
- Primary CTA must remain visible (sticky allowed only if applied everywhere; default not sticky)
- Mobile: keep CTA minimal and icon-first

### 1.5 Active state + location
- Highlight active route with left accent bar OR pill background + accent text (consistent with existing)
- Page title in header must match nav label (avoid "Accounts" vs "Account management" drift)

### 1.6 Logout placement rule
Logout must be:
- **Accessible** without hunting
- **Not** visually loud
- **Not** mixed with destructive domain actions

**Dual placement:**
1. **Profile menu** (top-right) → contains: Account settings, Security, Logout
2. **Sidebar footer** (bottom) as a secondary affordance: subtle "Log out" item (icon + label); requires confirmation only if unsaved changes exist

### 1.7 Layout consistency constraints
- Do not change page container width per screen.
- Do not change primary CTA placement per screen.
- Desktop: inline create/edit where appropriate for tables (Transactions).
- Mobile: drawers for create/edit on dense pages.
- Other pages: modal on desktop, drawer on mobile — must be consistent per page type.

---

## 2. Navigation Behaviors & Keyboard Contract

### 2.1 Keyboard shortcuts (optional v1)
If implemented, must be discoverable (Help modal), non-conflicting, and consistent.

Suggested minimal set:
- `Esc` closes overlay/drawer
- `Cmd/Ctrl + K` reserved for global search (v2); not used in v1

### 2.2 Focus behavior
- Route change: focus moves to page H1
- Overlay open: focus first input; overlay close: return focus to trigger element
- Sidebar nav: tab order is linear, no "focus traps" inside nav

### 2.3 Unsaved changes guardrail
If a form is dirty, closing modal/drawer triggers a neutral confirm:
- Title: "Discard changes?"
- Body: "Your edits won't be saved."
- Actions: "Keep editing" (primary), "Discard" (secondary/danger style)

---

## 3. Overlay Primitives (Modal / Drawer / Confirm)

### 3.1 Modal (desktop default)
Use for: short forms, confirmations, contextual edit where list context is important.

Rules:
- Centered, max width 560–620px
- Backdrop click closes only if safe (not for destructive confirm)
- `Esc` closes unless destructive confirm or unsaved changes exist
- Always has visible "Cancel"

### 3.2 Drawer (mobile default)
Use for: create/edit flows on mobile, any multi-field editing where maintaining context helps.

Rules:
- Bottom sheet drawer
- Has a handle + title
- `Esc` closes; backdrop click closes unless destructive confirm/unsaved changes
- Full height up to ~88–90vh; internal scroll

### 3.3 Confirm overlay (destructive / structural)
Use for: Archive/delete, structural changes (e.g., starting balance adjustments, schedule changes), logout if unsaved changes exist.

Rules:
- Never uses emotional language
- Always states impact in one sentence
- Buttons: Primary = the safe choice (Cancel / Keep); Secondary = action (Archive / Delete / Apply)

### 3.4 Overlay tone rules (microcopy)
- Don't mention whether entities exist (security: no user enumeration).
- Use "This will…" statements, not "Are you sure?"
- For irreversible actions: explicitly say "permanent".

Examples:
- Archive: "This will archive the category. Historical transactions remain unchanged."
- Delete: "This will permanently remove the vendor. Past transactions keep their vendor name."

### 3.5 Confirmation grammar
- **Title**: verb + object ("Archive category", "Delete vendor", "Sign out")
- **Body**: one-liner about impact + what remains unchanged
- **Consequence line** (optional): short bullet or second line for critical details
- **Actions**: Cancel (primary), Confirm (secondary or danger style depending)

### 3.6 Special case: Logout confirmation
- If no unsaved changes: logout is immediate.
- If unsaved changes: Confirm overlay "Sign out?" / "You have unsaved changes. Signing out will discard them." / Actions: "Keep editing" (primary), "Sign out" (secondary)

---

## 4. Notification Primitives

### 4.1 Two notification channels

1. **Inline feedback** (within page/card/form)
   - Validation messages
   - Save success under the button or near header
   - "Last updated" / "Changed" chips

2. **Toast notifications** (global, non-blocking)
   - Short-lived confirmations
   - Background sync outcomes
   - Non-critical errors (critical errors should be inline + retry)

### 4.2 Toast rules
- Position: bottom-right desktop, full-width bottom mobile
- Duration: success 2–3s / info 3–4s / error sticky until dismissed (or 6–8s if non-critical)
- Max 3 stacked
- Contains: title (optional), message (one sentence), optional action ("Undo", "Retry")

Tone examples:
- Success: "Saved."
- Info: "Schedule updated. Future periods regenerated."
- Error: "Could not save. Please try again."

### 4.3 Visual style of state cards

**Locked**
- Dashed border (1px)
- Muted surface background
- Reduced contrast (muted color tokens, not opacity)
- Shows: title, status ("Not configured", "Incomplete"), requirement line, "Configure" link/button (still clickable)

**Empty**
- Normal border (not dashed)
- Shows: title, empty title ("No data yet"), helper line about what will populate it, optional "Add first …" CTA

**Loading**
- Skeleton rows/blocks (ghost shimmer pattern)
- No actions

**Error**
- Normal border
- Shows: title, one-line error, Retry action, optional details link (expands)

**Active**
- Full content + actions

---

## 5. Unified Rendering Contract (Pages and Cards)

### 5.1 State precedence order
1. **Locked** (prerequisite not met)
2. **Error** (request failed)
3. **Loading** (request in-flight)
4. **Empty** (no items / no data)
5. **Active** (has data)

This prevents contradictory UI (e.g., showing empty when user is locked).

### 5.2 State interface (conceptual)
For each renderable unit, define:
- `isLocked: bool` / `lockReason: string` / `lockCTA: { label, href | onClick }?`
- `isLoading: bool`
- `error: { message, retryAction?, details? }?`
- `isEmpty: bool` / `empty: { title, message, cta? }?`
- `data: ...` (active content)

### 5.3 Page-level vs card-level
- **Page-level** contract governs the whole page (used when core data cannot be loaded)
- **Card-level** contract governs dashboard widgets (each card can be empty/loading/error independently)

### 5.4 Partial data rule
Navigation shell + header should still render; then state contract determines content area rendering.

### 5.5 "Change surfaced" contract (transparent updates)
When derived outputs change due to edits, show one of:
- Small inline banner under header: "Updated values reflect your recent changes."
- Toast: "Updated. Historical summaries recalculated."

Never show scary warnings; keep it informational.

---

## 6. Empty / Loading / Error Templates

### 6.1 Empty template (page)
- Title: "No {items} yet."
- Message: one sentence about what this list represents.
- CTA: primary "Add {item}"
- Secondary: link to docs (optional)

### 6.2 Loading template (page)
- Skeleton list rows matching real layout (prevents layout shift)
- Title area stays stable

### 6.3 Error template (page)
- Title: "Could not load {items}."
- Message: "Your structure is unchanged. Try again."
- CTA: "Retry"
- Secondary: "Contact support" (optional, later)

---

## 7. Implementation Notes (Handoff to FE Engineer)
Centralize primitives as reusable components:
- `AppShell`, `SidebarNav`, `TopBar`, `PageHeader`
- `StateRenderer` (implements precedence)
- `Modal`, `Drawer`, `ConfirmDialog`
- `ToastProvider`

Enforce state contract through a single helper:
- `resolveState({locked, error, loading, empty}) -> state`

Keep CSS tokens aligned with `design-system.md` and `INTERACTION_LANGUAGE.md`.

---

## 8. Acceptance Criteria (Phase 3 Complete)
- All pages use the same shell and page header patterns.
- Sidebar items and order match spec, with correct active highlight.
- Logout available in profile menu + sidebar footer.
- All overlays follow primitives and tone rules; keyboard + focus behavior implemented.
- Toasts and inline feedback follow the notification rules.
- All screens implement unified rendering contract (state precedence).
