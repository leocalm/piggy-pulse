# Account Detail Page — Design

**Date:** 2026-02-21
**Status:** Approved

## Overview

A dedicated account detail page (`/accounts/:accountId`) that gives users a full picture of a single account: current balance, period flow, balance history chart, category spending breakdown, stability context, and a paginated transaction list.

The page is scoped to the currently selected budget period (from global period context).

---

## Backend API

Four new endpoints, all nested under `/api/v1/accounts/<id>/`. Each is independently cacheable and has a focused SQL query.

### `GET /accounts/<id>/detail?period_id=<uuid>`

Returns the header and flow summary for the page. Loaded immediately on page mount.

```json
{
  "balance": 452000,
  "balance_change": 32000,
  "inflows": 320000,
  "outflows": 288000,
  "net": 32000,
  "period_start": "2025-05-01",
  "period_end": "2025-05-31"
}
```

All monetary values are integer cents. Validates that `period_id` belongs to the authenticated user.

---

### `GET /accounts/<id>/balance-history?range=period|30d|90d|1y&period_id=<uuid>`

Returns daily balance data points for the chart. `period_id` is required only when `range=period`; ignored for calendar ranges.

```json
[
  { "date": "2025-05-01", "balance": 420000 },
  { "date": "2025-05-02", "balance": 418500 }
]
```

Index on `(account_id, occurred_at)` serves all range variants. Frontend refetches when the user switches range tabs.

---

### `GET /accounts/<id>/transactions?period_id=<uuid>&type=all|in|out&cursor=<str>&limit=<n>`

Cursor-paginated transaction list for this account. `type` defaults to `all`; filters by flow direction relative to this account. Returns the existing `TransactionResponse` shape extended with a `running_balance` field (integer cents, computed server-side).

`period_id` is required, consistent with the rest of the app. Frontend refetches when the user switches the type filter.

---

### `GET /accounts/<id>/context?period_id=<uuid>`

Loaded lazily — only on first expansion of the Context section. Looks back at up to 6 prior closed periods for stability stats.

```json
{
  "category_impact": [
    { "category_name": "Housing", "amount": 120000, "percentage": 42 }
  ],
  "stability": {
    "periods_closed_positive": 4,
    "periods_evaluated": 6,
    "avg_closing_balance": 410000,
    "highest_closing_balance": 490000,
    "lowest_closing_balance": 365000,
    "largest_single_outflow": 120000,
    "largest_single_outflow_category": "Housing"
  }
}
```

---

## Frontend Architecture

**Stack:** React + Mantine UI + Mantine Charts (`AreaChart` for balance history) + React Query

### Route

`/accounts/:accountId` — navigated to by clicking an account in the accounts list.

### Data Fetching

| Hook | Query key | Trigger |
|---|---|---|
| `useAccountDetail(accountId, periodId)` | `['account-detail', accountId, periodId]` | Page mount |
| `useAccountBalanceHistory(accountId, range, periodId)` | `['account-balance-history', accountId, range]` | Page mount; refetch on range change |
| `useAccountTransactions(accountId, periodId, type)` | `['account-transactions', accountId, periodId, type]` | Page mount; refetch on type filter change |
| `useAccountContext(accountId, periodId)` | `['account-context', accountId, periodId]` | First expand of Context section |

### Component Tree

```
AccountDetailPage
├── AccountDetailHeader         ← name, type, balance, period change subtitle
├── BalanceHistoryChart         ← Mantine AreaChart + range segment buttons (Period/30d/90d/1y)
├── PeriodFlow                  ← inflows / outflows / net summary grid
├── AccountContextSection       ← collapsible toggle, lazy-fetches on first open
│   ├── CategoryImpact          ← category rows with progress bars
│   └── StabilityContext        ← stats list (periods positive, avg/high/low balance, largest outflow)
└── AccountTransactions
    ├── TransactionTypeFilter   ← All / Inflows / Outflows pill buttons
    └── TransactionTable        ← date-grouped rows, running balance column, cursor pagination
```

### Key UX Decisions

- **Progressive loading:** each section renders its own skeleton independently; the page feels fast even before all data arrives.
- **Lazy context:** `useAccountContext` is only triggered on first expand of the Context section — avoids computing stability stats on every page load.
- **Running balance:** computed server-side and included in the transaction response so the frontend does not need to accumulate it.
- **Empty state:** if the account has no transactions in the selected period, the transactions section shows a zero-state illustration rather than an empty table.
- **Chart zero-line:** Mantine `AreaChart` reference line at `y=0` to show negative balance region visually (matches wireframe).

---

## What Already Exists

- `GET /accounts/<id>` — basic account info (name, type, icon, color); reused for page header
- `GET /transactions/` — global transaction list (no account filter); not reused here
- `balance_per_day` on the list endpoint — not reused; replaced by the dedicated `/balance-history` endpoint

---

## Out of Scope

- Editing transactions from the detail page (link to transaction edit page)
- Multi-account comparison view
- Export / download of transaction list
