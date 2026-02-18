# PiggyPulse — Accounts Page Final Specification

## Purpose

Accounts show structural financial reality.

Answers:
- Where is my money?
- How is it evolving?
- What liquidity do I control?

Not advisory.

---

## Top Summary

Displays:
- Total Net Position
- Liquid Balance
- Protected (Savings)
- Debt
- Last updated

Definitions:
Net Position = Sum of all balances (including negative)
Liquid = Checking + Wallet + Allowance
Protected = Savings
Debt = Credit cards

---

## Account Grouping

1. Liquid Accounts
2. Savings
3. Allowance Accounts
4. Credit Cards

Sorted by absolute balance descending within groups.

---

## Standard Account Card

Displays:
- Name
- Current Balance
- Change this period
- Trend (sparkline)

Change This Period = Sum of transactions in current period.

---

## Allowance Account Model

Allowance is a pre-budgeted transfer from shared funds into a personal envelope.

Key Rules:
- Transfer is categorized in shared budget.
- Spending inside allowance does NOT affect shared categories.
- Balance can go negative.
- Negative carries forward.
- Next transfer corrects deficit automatically.

Example:
Start: €0
Transfer: +100
Spend: -150
Balance: -50
Next transfer: +100 → Balance = +50

---

## Allowance Card Displays

- Current Balance
- Transfer this period
- Personal Spend this period
- Net Change
- Remaining this period or deficit carried forward

No warnings.
No emotional styling.

---

## Category Scope

Categories are scoped:

SHARED → Used by non-allowance accounts
ALLOWANCE → Used only by allowance accounts

Allowance spending does not affect shared budget metrics.

---

## Tone Constraints

- Structural
- Calm
- Transparent
- No coaching
