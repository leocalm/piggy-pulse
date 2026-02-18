# TRANSACTIONS_SPEC

## Purpose
Define the full behavioral and UX contract for the Transactions page (desktop + mobile).

## Core Principles
- Category defines semantic direction.
- Transfer is a special dual-account transaction.
- Amount is always a positive value (minor units in backend).
- UI provides meaning; DB enforces invariants.

---

## Desktop Behavior

### Ledger
- Grouped by date (descending).
- Quick-add row fixed at top.
- Inline edit (no modal/drawer).
- Delete uses inline neutral confirmation row.

### Quick Add
Fields:
Date | Notes | Category | Counterparty | Amount

If Category = Transfer:
- Counterparty becomes `To account`.
- `From account` auto-filled from current account scope.
- Validation: From ≠ To.
- Single amount only.

Keyboard:
- Tab navigates fields.
- Enter (on last field) saves.
- Esc cancels and resets row.

### Inline Edit
- Enter saves.
- Esc cancels.
- Flash feedback on successful save.

---

## Mobile Behavior

### Layout
- Card list grouped by date.
- FAB opens drawer immediately.

### Drawer
Neutral title: “Transaction”.
Mode: add or edit (no visible distinction).

Dynamic fields:
Normal → Vendor
Transfer → From / To

Rules:
- From auto-filled from scope unless scope=All.
- From disabled unless scope=All.
- Switching category clears stale hidden state.
- Validation: From ≠ To.
- Enter (on input) saves.
- Esc closes drawer.

---

## Delete Tone
- Neutral.
- No red destructive emphasis.
- Confirmation required.
