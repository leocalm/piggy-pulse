# PiggyPulse — V1 Master Tracking Document
_Last updated: 2026-02-17 UTC_

Purpose:
Track all structural, lifecycle, and UX work required to move PiggyPulse from v0 → v1 with consistency and architectural integrity.

Implementation (migrations/tests) will follow design finalization.

Recent updates:
- 2026-02-18: Repository reorganized — all specs moved into `designs/v1/` feature folders, filenames normalized to `snake_case`, UX Infrastructure v1+v2 merged into single canonical spec, stubs and prompt files removed, `.gitignore` added.
- 2026-02-17: Settings page wireframe completed (`designs/v1/settings/piggypulse-settings.html`) and documentation references aligned.

---

# Immediate Priority Fixes (Top Of Queue)

## Transactions v10 Spec-Compliance Fixes
- [ ] Add global vs single-account table variants per spec
  - Global view: Date | Notes | Category | Account | Vendor | Amount
  - Single-account view: Date | Notes | Category | Vendor | Amount | Running Balance
- [ ] Fix transfer quick-add behavior for scope = All
  - From account must be selectable (not hardcoded)
- [ ] Add required filter set
  - Account, Category, Direction (In/Out), Vendor (optional), Date range
- [ ] Enforce amount validation rules
  - Amount > 0, no negative input, inline non-modal errors
- [ ] Render amount with explicit directional display (+ / -) by semantic direction
- [ ] Add subtle allowance spending badge in transaction rows/cards

---

# Phase 1 — Lifecycle & Governance (Lock First)

## Transaction & Transfer
- [x] Finalize transaction direction model
- [x] Finalize transfer atomicity rules
- [x] Finalize allowance spending invariants
- [x] Define DB-level constraints (documentation only for now)

## Periods
- [x] Lock period editing rules
- [x] Lock gap detection behavior
- [x] Lock non-overlapping invariant
- [x] Define impact preview for boundary edits

## Categories & Budgets
- [x] Finalize budget-per-period model
- [x] Define copy-on-new-period behavior
- [x] Define budget edit rules (current vs future)
- [x] Lock category archiving behavior

## Accounts
- [x] Lock immutable account type rule
- [x] Define archive behavior
- [x] Lock allowance transfer scheduling behavior

## Vendors
- [x] Define merge behavior
- [x] Define archive behavior

---

# Phase 2 — UX Finalization (v1 Look & Feel)

## Core Pages
- [x] Dashboard (final polish baseline delivered)
- [x] Categories page (final layout baseline delivered)
- [x] Accounts page (final layout baseline delivered)
- [x] Account detail page (final layout baseline delivered)
- [ ] Transactions page (add + list) final compliance pass pending (see top fixes)

## Management Pages
- [x] Accounts management
- [x] Categories management
- [x] Vendors management
- [x] Periods management
- [x] Settings page

## Authentication
- [x] Login
- [x] 2FA flow
- [x] Forgot password
- [x] Reset password

## Onboarding
- [x] First period setup
- [x] Category setup
- [x] Account setup
- [x] Optional allowance setup

---

# Phase 3 — UX Infrastructure

- [x] Sidebar / Navigation consistency
- [x] Global search (optional for v1)
- [x] Empty states
- [x] Error states
- [x] Loading states
- [x] Confirmation overlays
- [x] Notification patterns

---

# Phase 4 — Observability & Security (Post-UX Lock)

- [ ] Failed login tracking
- [ ] Account lock rules
- [ ] Audit logging
- [ ] Metrics hooks
- [ ] Rate limiting verification

---

# Explicit Non-Goals for v1

- ❌ Gamification
- ❌ Coaching engine
- ❌ Budget templates
- ❌ Recurring transactions
- ❌ Multi-currency

---

Execution Philosophy:
1. Lock semantics.
2. Finalize UI/UX consistently.
3. Implement.
4. Harden with migrations & tests.
5. Ship v1 with structural clarity.

---

End of PIGGYPULSE_V1_MASTER_TRACKING.md
