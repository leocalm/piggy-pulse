
# ACCOUNTS_MANAGEMENT_PAGE_SPEC.md

## Purpose

Accounts Management is structural configuration, not reflection.

This page allows users to:
- Create accounts
- Edit account metadata
- Archive accounts
- Delete accounts (when allowed)
- Adjust starting balance (with lifecycle constraints)

This page does NOT show:
- Trends
- Targets
- Deviation
- Charts

Tone: Administrative, calm, structurally clear.

---

# Page Structure

## Header

Title: Accounts  
Subtitle: Define and manage your financial accounts  
Primary CTA: + Add Account

---

# Layout

Flat list grouped by account type:

- Checking
- Savings
- Credit Card
- Wallet
- Allowance

All accounts visible on one page.

---

# Account Row Structure

[Icon] Account Name
Type: Checking
Currency: EUR
Starting Balance: €1,000
Status: Active / Archived
Actions: Edit | Archive | Delete

No live balance displayed here.

---

# Create Account

Desktop: Modal  
Mobile: Drawer

Fields:
- Account Name (required)
- Type (required, immutable after creation)
- Currency (required)
- Starting Balance (required)
- Optional description

If Type == Allowance:
Show helper text:

"This account represents a periodic personal allowance. Balance may go negative. Transfers define allowance amount."

---

# Edit Account

Editable:
- Name
- Description

Conditionally Editable:
- Currency (only if no transactions exist)

Not Editable:
- Type
- Starting Balance (after transactions exist)

If transactions exist:
- Starting Balance field replaced by "Adjust Starting Balance" action

---

# Adjust Starting Balance

Allowed only if earliest affected period is still open.

If allowed:
Open modal:

Title: Adjust Starting Balance

Text:
"The starting balance defines the base from which all account history is calculated. Changing it will update all derived balances."

Fields:
- New starting balance
- Optional note

Behavior:
System creates a structural adjustment entry.

If not allowed:
Disable action with tooltip:

"Starting balance can no longer be adjusted because its initial period is closed. You may create a transaction to correct the balance."

---

# Archive Rules

If account has transactions:
- Archive only (no delete)

If account has no transactions:
- Delete allowed

Archive confirmation text:

"This will archive the account. Historical transactions remain unchanged."

Archived accounts:
- Hidden from active pages
- Still visible in management page under Archived section
- Cannot receive new transactions

---

# Period Lifecycle Alignment

- Periods auto-close when now > period.end_date
- Targets immutable after closure
- Starting balance adjustments disallowed if earliest affected period is closed
- Transactions remain editable even in closed periods
- Historical reports always show recalculated current truth (no audit layering)

---

# Domain Invariants

- Account type immutable
- Currency immutable once transactions exist
- Transfers must respect direction invariants
- Allowance balance may go negative
- Deleting account with transactions is prohibited

---

# Interaction Language Alignment

- Neutral tone
- No alarm states
- No coaching language
- Clear consequence statements
- Modal for destructive confirmation
- Drawer for mobile editing
