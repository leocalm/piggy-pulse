# ONBOARDING_SPEC.md

## Purpose

Onboarding defines the minimum structural configuration required before a user can access the main PiggyPulse dashboard.

Onboarding is mandatory.
It defines structure before observation.

Users cannot access the dashboard until onboarding is completed.

---

# Core Principles

1. Structure First – Time model, accounts, and categories must exist before tracking.
2. Atomic Steps – Each step persists independently.
3. Resume Friendly – Users can leave and return at any step.
4. No Partial Structural Corruption – Each step validates before commit.
5. Calm Tone – No gamification, no urgency language.

---

# Flow Overview

Step 1 – Define Period Model
Step 2 – Create Accounts
Step 3 – Create Categories
Step 4 – Structural Summary

Progress is saved after each step.

### Evolution (2026-02-17)
The separate “Enter Application” step was removed.
Entry now happens directly from the Structural Summary step via:
- CTA: `Enter PiggyPulse`

This keeps the flow structural and avoids a redundant confirmation step.

---

# Step 1 – Period Model (Required)

User must choose:

- Automatic schedule OR manual
- Start day
- Duration
- Generate ahead
- Weekend rule
- Naming pattern

Validation:
- No overlapping periods (DB enforced)
- Valid date ranges
- Future periods generated correctly

User cannot proceed without valid configuration.

---

# Step 2 – Accounts (Minimum 1 Required)

User must create at least one account.

Rules:
- At least one asset account required (Checking, Savings, Wallet)
- Starting balance required
- Account type immutable after creation
- No default hidden accounts created automatically

Validation:
- Account name required
- Currency required
- Starting balance numeric

---

# Step 3 – Categories (Minimum Structural Set)

User must create:

- At least 1 Incoming category
- At least 1 Outgoing category

Rules:
- Type immutable after first transaction
- Optional one-level parent/child allowed
- No targets referenced here

Validation:
- Name required
- Type required

---

# Step 4 – Structural Summary

Displays:

- Period model configuration
- Accounts created
- Categories created

Purpose:
Provide transparency before entering system.

CTA:
Enter PiggyPulse

---

# Onboarding State Machine

Canonical state:
`onboarding_status`

Values:
- `not_started`
- `in_progress(step=period|accounts|categories|summary)`
- `completed`

Rules:
- Progress is committed atomically per step.
- Resume target is always the latest incomplete step.
- If state is `completed`, onboarding route redirects to dashboard.
- If state is not `completed`, dashboard route redirects to onboarding.

Dashboard guard:
- `if onboarding_status != completed => redirect /onboarding`

Edge handling:
- If user updates period model during onboarding, recompute step validity and keep resume at earliest invalid step.
- If user removes all accounts after previously satisfying minimums, accounts step becomes invalid and blocks entry until minimum is restored.
- If user opens dashboard in another tab before completion, guard redirects to onboarding without exposing partial app access.

---

# Abandonment Handling

If user leaves mid-onboarding:

- Progress is persisted
- On next login, resume from last incomplete step
- Dashboard access remains locked

---

# Not Allowed

- Access to dashboard before completion
- Silent default entity creation
- Emotional or celebratory UI

Onboarding is configuration, not marketing.
