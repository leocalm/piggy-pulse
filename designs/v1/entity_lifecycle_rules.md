# PiggyPulse — Entity Lifecycle Rules
_Last updated: 2026-02-16 UTC_

This document defines lifecycle behavior, edit constraints, archival rules,
and cross-entity invariants for PiggyPulse.

These rules ensure deterministic financial behavior and immutable historical meaning.

---

# 1. Accounts Lifecycle

## Creation
- Account type must be selected at creation.
- Account type is immutable.
- Initial balance stored once and never recomputed.
- Initial balance must not be derived from transactions.

## Editing
Allowed:
- Name
- Description
- Visual metadata

Not allowed:
- Type change
- Currency change (future-proof rule)

## Archiving
Allowed even if transactions exist.
Effects:
- Hidden from default lists.
- Cannot create new transactions.
- Cannot be transfer source or target.
- Historical data remains accessible.

If account has scheduled allowance transfer:
- Transfer rule must be disabled automatically.

Hard deletion allowed only if no transactions exist.

---

# 2. Categories Lifecycle

## Creation
- Must have type: Incoming or Outgoing.
- Type immutable.
- (name, type) combination must be unique.

## Editing
Allowed:
- Name
- Visual metadata

Not allowed:
- Type change (prevents historical reinterpretation).

## Archiving
Allowed.
Effects:
- Historical transactions preserved.
- No new transactions allowed.
- No new budgets created for future periods.

Deletion allowed only if no transactions exist.

---

# 3. Budget Lifecycle (Per Period Model)

Budgets are scoped per category per period.

## On Period Creation
For each active category:
- Copy budget from most recent period.
- If none exists → default to 0.

## Editing Budget
Default:
- Affects current period only.

Optional:
- Apply to future periods (update existing future rows).

Never:
- Modify past period budgets.

## Category Archived Mid-Period
- Current period budget remains.
- No budget created for future periods.

---

# 4. Vendors Lifecycle

## Creation
- Name must be unique (case-insensitive).

## Editing
Allowed:
- Name
- Metadata

## Merging
Merge Vendor B into Vendor A:
- Reassign all transactions from B to A.
- Archive B.
- No transaction loss.

## Archiving
Allowed.
- No new transactions allowed.
- Historical data preserved.

Deletion allowed only if no transactions reference vendor.

---

# 5. Period Lifecycle

## Creation
Manual or automatic.
Constraints:
- No overlapping periods.
- Start date < End date.
- Timezone consistent.

## Editing
Allowed:
- Start date
- End date
- Name

On edit:
- Must compute impact preview:
  - Transactions entering period
  - Transactions leaving period

No silent reshuffling.

## Gaps
Allowed.
Detected and surfaced informationally.

## Deleting Period
Allowed if not current period.
Cascade delete associated CategoryBudget rows.
Transactions remain intact (period derived by date).

---

# 6. Transactions Lifecycle

## Creation
Must satisfy:
- amount > 0
- direction defined (In | Out)
- Either category_id OR transfer_group_id must be present, never both.

## Editing
Allowed:
- Date
- Amount
- Category (non-transfer only)
- Vendor
- Description

If transfer:
- Must update both entries atomically.

## Deletion
If normal transaction:
- Delete single entry.

If transfer:
- Delete entire transfer pair.

## Date Change
Changing date may move transaction across periods.
Allowed and derived automatically.

---

# 7. Transfer Lifecycle

## Creation
- Atomic creation of two transactions.
- Must satisfy transfer invariants.

## Editing
- Atomic update of both entries.
- No partial edits allowed.

## Deletion
- Deletes entire transfer group.

---

# 8. Allowance Configuration Lifecycle

## Setup
Allowance account includes:
- Transfer source account
- Transfer amount
- Period schedule

## Transfer Execution
At start of each period:
- Create transfer from source → allowance.
- Balance rolls forward automatically.

## Disabling Allowance
- Stops future transfers.
- Does not reset balance.
- Does not alter historical data.

---

# 9. Cross-Entity Invariants

The following must never be violated:

1. Periods do not overlap.
2. Account type is immutable.
3. Transaction direction is explicit.
4. Transfers are always balanced pairs.
5. Category budgets are per period and immutable historically.
6. Allowance spending excluded from category budget aggregation.
7. No transaction exists without an account.
8. No transfer exists without exactly two entries.
9. Historical financial meaning must never be silently mutated.

---

End of ENTITY_LIFECYCLE_RULES.md
