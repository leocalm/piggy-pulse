# PERIOD_MANAGEMENT_PAGE_SPEC.md

## Purpose

Provide a calm, structural view of:
1) the user’s period schedule configuration (optional)
2) the current period and its status
3) upcoming/future periods
4) past/closed periods

This page is not a “budget coach.” It must be descriptive and neutral.

---

## Visual Hierarchy

1) **Current Period** (position in time, highest relevance)
2) **Schedule** (structural rule layer)
3) **Upcoming Periods** (pipeline)
4) **Past Periods** (history)

The layout must make the user’s *current temporal position* obvious without alarms or celebration.

### Evolution (2026-02-17)
Hierarchy order was updated from `Schedule → Current → Upcoming → Past` to `Current → Schedule → Upcoming → Past` to reinforce position-first salience and reduce cognitive distance to “now.”

---

## Sections

### 1) Header
- Title: “Periods”
- Subtitle: “Time windows that help you track patterns.” (or similar neutral copy)

Primary actions:
- “Create Period” (manual)
- “Schedule” / “Edit Schedule” (if schedule exists)

On mobile: actions may collapse into a single button + overflow menu.

---

### 2) Current Period Card (Highest Salience)
Shows:
- period name
- date range
- derived status: “Open”
- “Days remaining” (neutral)
- key rollups (optional):
  - total incoming, total outgoing, net for the period
  - do not over-emphasize morality (“overspend” language discouraged)
- Actions:
  - “View details” (optional)
  - “Edit targets” (if targets exist on a separate page, link there)

Tone:
- descriptive: “Remaining days” / “Period progress”
- avoid punitive red states; use subtle contrast for deviance if needed

---

### 3) Schedule Card (Structural Configuration)

**If schedule is disabled / not configured:**
- Display “Manual periods” state
- Action: “Set up schedule”
- Copy: schedule generates future periods automatically (neutral)

**If schedule is enabled:**
- Show current schedule summary:
  - mode: Automatic
  - start day
  - duration + unit
  - generate-ahead count
  - weekend adjustment rules
  - naming pattern
- Actions:
  - “Edit schedule” (opens schedule modal/drawer)
  - “Disable schedule” (confirmation)
- Must include disclosure:
  - “Updating schedule will regenerate future periods. Past and current periods are not changed.”

**Device behavior:**
- Desktop: modal for schedule config
- Mobile: bottom drawer for schedule config

---

### 4) Upcoming Periods (Future Pipeline)
List future periods (generated or manual).
For each:
- name
- date range
- status: “Upcoming”
- optionally show target summary (collapsed)

Actions (optional, depending on mode):
- If manual: edit name, archive/delete if empty
- If automatic: no per-period editing (they are outputs of the schedule)
  - optional: “View” only
  - avoid “edit” that implies modifying schedule output

Disclosure:
- “Future periods may be regenerated if schedule changes.”

---

### 5) Past Periods (Closed)
Collapsed section by default.
Each row shows:
- name
- date range
- status: “Closed”
- summary metrics (optional)

Important: Do not imply that transactions are locked.
However, the UI can include a subtle note:
- “Transactions remain editable; historical summaries will update accordingly.”

---

## States

### Empty
- No periods exist yet
- Show explanation:
  - “Create a period manually or configure a schedule.”
- CTA: “Create first period”

### Loading
- Skeleton cards for schedule/current/upcoming/history list

### Error
- Neutral error message and “Retry”

---

## Key Behavioral Rules (Must Be Reflected in UI Copy)

1) **No transaction-period linkage exists.**
   - Period membership is derived from dates.

2) **Editing past transactions is allowed.**
   - UI must disclose consequence when editing, not block.

3) **Period boundaries are structural.**
   - Do not offer editing start/end dates in management UI.

4) **Schedule edits regenerate the future.**
   - Must disclose in schedule editor.

---

## Accessibility & Interaction

- Tab order must follow visual order.
- ESC closes modals/drawers.
- Enter submits when focused on final field.
- Focus trap inside modal/drawer.

---

## Non-Goals

- No audit logs in V1
- No “re-open period” controls
- No snapshot reports
