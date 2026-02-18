# PiggyPulse — Account Detail Page Wireframe (Structural + Mock)

This document defines the structural and spatial layout of the Account Detail Page.

This is not a visual design.
This is a layout contract.

The page is fully period-anchored.

---

# GLOBAL CONTEXT

Selected Period: June 2026

This page must not override global period.

---

==================================================
PAGE MOCK — DEFAULT (LIQUID ACCOUNT)
==================================================

--------------------------------------------------
HEADER
--------------------------------------------------

Main Checking                         [ Liquid ]
$4,520
+ $320 this period
June 2026                                  Updated 2h ago


--------------------------------------------------
BALANCE CHART
--------------------------------------------------

                [ Period | 3P | 90D | 1Y ]

   $5,000 |                 ●
   $4,800 |           ●
   $4,600 |       ●
   $4,400 |  ●
   $4,200 |
           --------------------------------
            1   5   10   15   20   25

(Hover: Date | Balance | Daily Delta)

Rules:
- Selector changes chart only.
- Period remains anchor.


--------------------------------------------------
THIS PERIOD — CASH FLOW
--------------------------------------------------

Inflows     $1,200
Outflows      $880
Net          +$320


--------------------------------------------------
CATEGORY IMPACT (THIS ACCOUNT)
--------------------------------------------------

Groceries        $320   36%
Dining Out       $180   20%
Transport        $140   16%
Other            $240   28%

(Percentages relative to account outflows)


--------------------------------------------------
HISTORICAL CONTEXT
--------------------------------------------------

Closed positive in 4 of last 6 periods
Average closing balance (6P): $4,180
Highest closing balance: $5,020
Lowest closing balance:  $3,940


--------------------------------------------------
TRANSACTIONS (DEFAULT: CURRENT PERIOD)
--------------------------------------------------

Date        Category       Counterparty      Amount
------------------------------------------------------
06 Jun      Groceries      Albert Heijn      -$82
08 Jun      Dining Out     Restaurant X      -$45
10 Jun      Salary         Employer         +$2,000
...

[ Filters: Date | Category | Amount ]

---

==================================================
ALLOWANCE ACCOUNT VARIANT MOCK
==================================================

--------------------------------------------------
HEADER
--------------------------------------------------

Personal Allowance                 [ Allowance ]
-$50
-$150 this period
June 2026                                  Updated 2h ago


--------------------------------------------------
BALANCE CHART
--------------------------------------------------

                [ Period | 3P | 90D | 1Y ]

   $100  |
    $50  |       ●
     $0  |-------------- ZERO LINE --------------
    -$50 |             ●
   -$100 |
            --------------------------------
             1   5   10   15   20

Zero line must be visible if negative possible.


--------------------------------------------------
THIS PERIOD — ALLOWANCE SUMMARY
--------------------------------------------------

Transfer In     +$100
Spent           -$150
Net             -$50
Current Balance -$50

Next Transfer (01 Jul):
+ $100
Projected Balance After Transfer: $50

No warnings.
No red framing.
Overspending allowed.


--------------------------------------------------
CATEGORY IMPACT (ALLOWANCE SPENDING)
--------------------------------------------------

Hobbies         $90    60%
Books           $60    40%

(Allowance spending does NOT affect category budgets)


--------------------------------------------------
TRANSACTIONS
--------------------------------------------------

Date        Category       Counterparty      Amount
------------------------------------------------------
03 Jun      Hobbies        Game Store        -$90
07 Jun      Books          Amazon            -$60

---

==================================================
DEBT ACCOUNT VARIANT MOCK
==================================================

--------------------------------------------------
HEADER
--------------------------------------------------

Credit Card                          [ Debt ]
-$3,150
-$420 this period
June 2026                                  Updated 2h ago


--------------------------------------------------
THIS PERIOD — DEBT FLOW
--------------------------------------------------

Charges     $520
Payments    $100
Net Change  -$420


Optional:
Credit Limit: $5,000
Utilization: 63%

No urgency language.


--------------------------------------------------
TRANSACTIONS
--------------------------------------------------

Date        Category       Counterparty      Amount
------------------------------------------------------
04 Jun      Dining Out     Restaurant Y      -$75
12 Jun      Payment        Bank Transfer    +$100

---

# PROHIBITED ELEMENTS

Designer must NOT introduce:

- Advisory messages
- Predictive insights
- Emotional warnings
- Budget deviation commentary
- Mixed time scopes
- Auto-generated "suggestions"

The page is analytical, not interpretive.