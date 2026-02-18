# VENDORS_MANAGEMENT_PAGE_SPEC.md

## Purpose

Vendors answer one question only:

> Where did the money go?

Vendors are observational metadata. They are not structural entities and do not affect:
- Budget targets
- Category logic
- Account balances
- Period behavior

They exist solely to support behavioral clarity and pattern discovery.

---

## Domain Model

Fields:
- id: UUID
- name: string
- normalized_name: string (lowercase, trimmed)
- description?: string
- created_at: timestamp
- archived_at?: timestamp

Constraints:
- Unique index on normalized_name
- Vendors are identified by UUID, not name

---

## Creation Semantics

Vendors are auto-created from transaction entry.

When user types a vendor:
1. Normalize input
2. If normalized vendor exists → reuse
3. Else → create silently
4. Associate transaction with vendor_id

No manual creation required during transaction flow.

---

## Lifecycle Rules

Active:
- Assignable to transactions

Archived:
- Cannot be assigned to new transactions
- Still visible in historical transactions
- Still visible in management page

Delete allowed only if:
- No transactions reference the vendor

---

## UI Structure

Header:
- Page title
- Subtitle

Controls:
- Search input
- + Add Vendor button

List:
- Alphabetical sorting only
- No grouping
- No financial data
- No trends

Row layout:
- Vendor name
- Optional description
- Transaction count
- Edit
- Archive
- Delete (if no transactions)

Archived section:
- Collapsed by default
- Consistent pattern with categories page

---

## UX Constraints

- Vendors do not carry direction
- Vendors do not affect budgeting
- Vendors do not imply income or expense
- Vendors remain flat (no hierarchy)

---

## Future Considerations (Not V1)

- Vendor merge capability
- Vendor deduplication tools
- Import normalization rules

These are explicitly out of scope for V1.
