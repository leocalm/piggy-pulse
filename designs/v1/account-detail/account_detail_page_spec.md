
# PiggyPulse — Account Detail Page Specification (Final)

## 1. Purpose

The Account Detail Page explains how a single financial instrument behaves within the currently selected period.

It answers:

1. How is this account behaving in this period?
2. What caused the balance movement?
3. How does this account behave over time?
4. What transactions reconcile these numbers?

This page is analytical and descriptive.
It does not coach, warn, or interpret.

---

## 2. Core Design Principle

The page is fully period-anchored.

The globally selected period defines:

- Header delta
- Default chart scope
- Cash flow breakdown
- Category impact
- Default transaction filter

Historical views are secondary and must be intentionally selected.

No silent time scope switching.

---

## 3. Page Structure Overview

1. Header (Period Context)
2. Balance Evolution Chart (Primary)
3. Period Cash Flow Breakdown
4. Category Impact (Account-relative)
5. Stability Context (Multi-period summary)
6. Transactions (Ground Truth Layer)

---

## 4. Header — Period Context Block

### Purpose

Provide immediate clarity about the account's state within the selected period.

### Content

- Account Name
- Account Type Label (Liquid / Allowance / Protected / Debt)
- Current Balance (absolute)
- Period Delta (net change within selected period)
- Period Label (e.g., June 2026)
- Last Updated Timestamp (subtle)

### Rules

- Period delta reflects only selected period.
- No color-coded emotional framing.
- No advisory language.

---

## 5. Balance Evolution Chart (Primary Component)

### Default Scope

Current Period (period start → today)

### Chart Behavior

- Single balance line
- Subtle gradient fill
- X-axis = time
- Y-axis = balance
- Hover shows:
  - Date
  - Balance
  - Delta from previous day

Optional:
- Thin vertical marker for "today"

---

### 5.1 Chart Scope Selector

Segmented control embedded inside chart:

- Period (default)
- 3P (last 3 periods)
- 90D
- 1Y

Switching scope:
- Affects chart only
- Does NOT change header delta
- Does NOT change cash flow block
- Does NOT change category impact
- Does NOT change transactions

The period remains the cognitive anchor.

---

## 6. Period Cash Flow Breakdown

### Structure

This Period:

- Inflows
- Outflows
- Net

For debt accounts:

- Charges
- Payments
- Net change

No interpretation.
No warnings.

---

## 7. Category Impact (Period-Scoped)

### Applies To

- Liquid accounts
- Allowance accounts

### Structure

Per category:

- Category name
- Amount from this account
- % of this account’s total outflow

Percentages are account-relative only.

---

## 8. Stability Context (Multi-Period Summary)

### Example Metrics

- Closed positive in X of last Y periods
- Average closing balance (last Y periods)
- Highest closing balance
- Lowest closing balance

Purely descriptive.

---

## 9. Transactions (Ground Truth Layer)

### Default Filter

Current Period

### Sortable Fields

- Date
- Amount
- Category
- Counterparty
- Notes

### Filters

- Date range
- Category
- Amount range

All summaries must reconcile to this table.

---

## 10. Account Type Variants

### 10.1 Liquid Accounts

Standard behavior:
- Cash flow
- Category impact
- Stability context

---

### 10.2 Allowance Accounts

Allowance accounts are behavioral reservoirs.

Spending:
- Does NOT count toward category budget totals
- DOES affect allowance balance

#### Allowance Period Summary Block

This Period:
- Transfer In
- Spent
- Net
- Current Balance

Next Transfer:
- Expected Transfer
- Projected Balance After Transfer

Rules:
- Overspending allowed
- Negative balances allowed
- Carry-forward automatic
- No blocking
- No warnings

---

### 10.3 Protected Accounts

Focus on:
- Contributions
- Withdrawals
- Long-term evolution

---

### 10.4 Debt Accounts

Terminology:

- Inflows → Payments
- Outflows → Charges

Optional:
- Credit limit
- Utilization %

Descriptive only.

---

## 11. Interaction States

### 11.1 Empty Account

- Balance = 0
- Delta = 0
- Chart flat
- Transactions empty

---

### 11.2 No Activity in Current Period

- Historical data exists
- Period delta = 0
- Chart flat
- Transactions empty for period

---

### 11.3 Allowance Negative Balance

- Balance negative
- Projection visible
- No warning states

---

### 11.4 Debt Fully Paid

- Balance = 0
- Optional subtle note: "No outstanding balance."

---

### 11.5 Loading State

- Skeleton header
- Skeleton chart
- Skeleton rows

---

### 11.6 Error State

Message:

"Unable to load account data."
Retry button.

---

## 12. Invariants

The page must:

- Never mix time scopes silently
- Never coach
- Never warn (unless system error)
- Never override global period
- Always reconcile to transactions

---

## 13. System Alignment

Dashboard → Period alignment summary
Categories → Budget deviation visibility
Accounts → Structural financial inventory
Account Detail → Instrument-level explanation

The period remains the cognitive anchor of PiggyPulse.
