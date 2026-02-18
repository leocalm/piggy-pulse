# PiggyPulse — Transactions Page Specification
_Last updated: 2026-02-16 UTC_

This document defines the full UX and interaction model for the Transactions page,
including batch entry mode and transfer behavior.

---

# 1. Purpose

The Transactions page is the global ledger view.

It answers:
- What happened?
- When?
- Through which account?
- Under which category?
- For what amount?

It is descriptive and ledger-oriented.
No coaching logic. No behavioral nudging.

---

# 2. Entry Modes

## 2.1 View Mode

- Standard ledger table
- Default sorting: Date descending
- “Add” button opens Quick Add modal
- “Enter transactions” button switches to Batch Entry Mode

## 2.2 Batch Entry Mode (Primary for reconciliation)

- Inline entry row pinned at top of ledger
- “Done” button exits batch mode
- Sticky defaults maintained until exit

---

# 3. Table Structure

## 3.1 Global View (All Accounts)

Columns:
Date | Notes | Category | Account | Vendor | Amount

Running balance hidden.

## 3.2 Single Account Context

Triggered when:
- Navigated from account detail
OR
- Account filter applied

Columns:
Date | Notes | Category | Vendor | Amount | Running Balance

Account column hidden.
Running balance visible.

---

# 4. Inline Entry Row (Batch Mode)

Pinned at top of table.

Field order (keyboard optimized):

Date | Notes | Category | Account | Vendor | Amount | Save

If single account context:
Account field hidden (implicitly fixed).

---

# 5. Sticky Defaults

Persist during batch session:

- Date (default today)
- Account (if global context)
- Last used category NOT sticky (avoid accidental repetition)
- Vendor not sticky unless explicitly reselected

After save:
- New blank row appears
- Date and Account preserved
- Focus returns to Category

---

# 6. Transfer Behavior

Entry row supports two modes:

Transaction Mode (default):
Date | Notes | Category | Account | Vendor | Amount

Transfer Mode:
Date | Notes | From Account | To Account | Amount

Rules:
- Category and Vendor hidden in transfer mode
- On save: atomic pair created
- UI displays as single transfer row

Editing transfer:
- Must edit both sides atomically
- Never expose internal pair rows

---

# 7. Validation Rules

- Amount > 0
- Date required
- Category required (non-transfer)
- From/To required (transfer)
- No negative input allowed

Errors inline, non-modal.

---

# 8. Filtering

Filters:
- Account
- Category
- Direction (In/Out)
- Vendor (optional)
- Date range (optional)

Filters combinable and resettable.

---

# 9. Display Rules

Amount:
- Always positive input
- Display with + or − according to direction
- No dramatic red styling

Transfers:
- Display arrow indicator (→ or ←)
- Not associated with categories

Allowance spending:
- Small subtle badge
- Excluded from budget aggregation

---

End of TRANSACTIONS_PAGE_SPEC.md
