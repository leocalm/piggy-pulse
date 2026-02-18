# PERIOD_SEMANTICS.md

## Core Principle

In PiggyPulse, **periods are a logical layer**, not a constraint.

- Transactions are **not** mapped to periods (no `period_id`).
- Periods are **time windows** used to *interpret* transactions.
- Editing periods or transactions is allowed (with clear consequences), because the system is **reflective**, not compliance/audit-focused.

This design enables native “moving things around” (changing dates/categories/accounts) because all reporting is derived from the current ledger state.

---

## Definitions

### Transaction
A ledger event that exists in time.
- Has a date/time and impacts account balance based on direction/semantics.
- May be edited at any time.
- Never stores derived period membership.

### Period
A named time window:
- `start_date` (inclusive)
- `end_date` (exclusive or inclusive per implementation; UI must present a clear range)
- `name` / label (optional but recommended)
- status derived from current date, not stored as a mutable flag

### Schedule
A configuration that generates future periods automatically (optional).
- Users may run fully manual periods or automatic generation.
- Users may disable schedule at any time.

---

## Derived Period Membership

A transaction is *considered* part of a period if its `date` falls within the period’s window.

Example (inclusive start, exclusive end):
- Period: `[2026-02-01, 2026-03-01)`
- Transaction date: `2026-02-14` → belongs to February 2026

This membership is not stored; it is derived for every query/report.

---

## Status Model (Time-Driven)

Periods “close” automatically based on time.

Recommended derived statuses (do not store):
- **Upcoming**: `now < start_date`
- **Open**: `start_date <= now < end_date`
- **Closed**: `now >= end_date`

There is no manual “close” or “re-open”. Time drives status.

---

## Editing Rules (Reflective Ledger Model)

### Transactions (Always Editable)
Transactions may be edited at any time, including those whose dates place them in **Closed** periods.

Allowed edits include (subject to validations):
- date (may move transaction between periods)
- amount
- vendor
- category
- account(s) (including transfer endpoints)
- notes

**Effect:** editing a past transaction recalculates:
- affected period summaries (past and/or current)
- category performance
- account balances and charts
- trends and projections

There is no audit trail in V1. The system reflects current truth as the user understands it.

### Period Instances (Structurally Stable)
Period boundaries are **structural**. To preserve integrity:
- `start_date` / `end_date` are immutable after creation
- `name` may be editable (optional)
- schedule-driven regeneration affects only *future* periods (see below)

### Targets (Editable Anytime)
Budget/target definitions may be edited anytime.
- Changing a target recalculates how the user’s performance is displayed for that period.
- Targets are analytical, not transactional.

---

## Schedule Regeneration Semantics

When schedule configuration changes:

- **Current period**: unaffected
- **Past periods**: unaffected
- **Future periods**: may be deleted and regenerated according to new schedule rules

If a future period has transactions (because users can create transactions in future dates):
- It is still treated as a future window; regeneration can change which window those future-dated transactions fall into.
- UI must disclose this clearly.

---

## Zero Snapshot / No Intermediate State Policy

PiggyPulse stores:
- initial balances (structural base)
- transactions (events)
- configuration (schedule, targets, etc.)

PiggyPulse does **not** store:
- precomputed period totals
- cached daily balances
- snapshot reports
- intermediate “closed period” aggregates

All derived numbers are computed on demand. This ensures:
- consistency across UI
- correctness after edits
- no backfills or reconciliation jobs

---

## Interaction Language Constraints

PiggyPulse uses neutral, descriptive language:
- no coaching
- no moralizing
- no punitive messaging

Edits to past data must be described plainly:
- “This updates historical totals and charts.”
not:
- “Warning! This is dangerous.”

---

## Non-Goals (V1)

Explicitly out of scope:
- audit logs / immutable ledger
- versioned reports (“as of last month” snapshots)
- compliance-grade controls
