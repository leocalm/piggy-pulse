# Transactions Page — ASCII Wireframe
_Last updated: 2026-02-16 UTC_

--------------------------------------------------
| Transactions                         [ Add ]  |
| Period: February 2026                            |
--------------------------------------------------

[ Enter Transactions ]   (Batch Mode Toggle)

--------------------------------------------------
| Date | Notes | Category | Account | Vendor | Amount |
--------------------------------------------------

(Batch Entry Row - Visible only in Batch Mode)

| [date] | [notes] | [category] | [account] | [vendor] | [amount] | [Save] |

--------------------------------------------------
| 12 Feb | Groceries | Food | Checking | AH | -45.00 |
| 11 Feb | Salary    | Income | Checking |  | +3000 |
| 10 Feb | Transfer  | → Savings | Checking |  | -500 |
--------------------------------------------------


Single Account Context Layout
--------------------------------------------------
| Date | Notes | Category | Vendor | Amount | Balance |
--------------------------------------------------

Batch Entry Row (Single Account Context)

| [date] | [notes] | [category] | [vendor] | [amount] |

--------------------------------------------------
| 12 Feb | Groceries | Food | AH | -45.00 | 1,255 |
| 11 Feb | Salary    | Income |  | +3000 | 1,300 |
--------------------------------------------------


Transfer Entry Row (Transfer Mode)

| [date] | [notes] | [From Account] | [To Account] | [amount] |

--------------------------------------------------

Keyboard Flow:

Category → Tab → Account → Tab → Vendor → Tab → Amount → Enter → New Row

Esc clears row but preserves sticky defaults.

--------------------------------------------------

End of transactions_page_wireframe.md
