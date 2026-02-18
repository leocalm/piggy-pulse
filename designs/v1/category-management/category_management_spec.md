# CATEGORY_MANAGEMENT_PAGE_SPEC.md

## Purpose

Categories Management is **structural configuration**, not reflection.

This page allows users to:
- Create top-level categories
- Create subcategories (1 level deep)
- Edit category metadata (name, icon, description)
- Archive categories (soft-delete)
- Delete categories (only when safe)
- Search categories (without breaking structural context)

This page does **not**:
- Show targets
- Show deviation
- Show charts
- Show per-period operational status

Tone: administrative, calm, structurally clear.

---

## Core Model

### Category fields (conceptual)
- `id`
- `name`
- `type` = `Incoming | Outgoing` (Transfer is system-level, not user-created)
- `parent_id` (nullable)
- `is_archived` (bool)
- derived: `transaction_count` (integer)

### Hierarchy depth
- **Max depth = 1**
- A child category **cannot** have children.
- A parent category has `parent_id = null`.
- A child category has `parent_id = <parent>` where parent has `parent_id = null`.

### Type inheritance
- Child categories inherit `type` from parent (UI treats it as frozen).

---

## Invariants & Lifecycle Rules

### Type immutability
- Category **type** is editable **only** until the **first transaction** is created using that category.
- After that, type becomes **immutable**.

### Parent immutability (re-parenting)
- Re-parenting is allowed **only if** `transaction_count == 0`.
- Re-parenting must not create depth > 1 (cannot set parent to a child category).
- Re-parenting must preserve type (parent must have same `type`).

### Archive rules
- Archived categories cannot be used for new transactions.
- Historical transactions remain unchanged and continue to display the category name.
- **Parents cannot be archived while they have active children.**
  - UI must block with a neutral explanation.

### Delete rules
Hard delete allowed only if:
- `transaction_count == 0`
- category has **no children**
Otherwise:
- Archive only.

### Restore
- Archived categories can be restored.
- Restoring a child requires its parent to be active (or restore parent first).

---

## Ordering and Search

### Ordering
- Strict alphabetical ordering.
- Within sections:
  - Parents alphabetical
  - Children alphabetical within each parent
- No manual reordering or drag-and-drop.

### Search
- Search is case-insensitive, partial match.
- Search matches:
  - parent names
  - child names

#### Structural-preserving filter behavior
- If a child matches, the parent must be shown even if the parent does not match.
- Under that parent, only matching children are shown.
- If a parent matches, show the parent and all children for context.

---

## Information Architecture

### Header
- Title: **Categories**
- Subtitle: **Define and organize your budgeting structure**
- Primary CTA: **+ Add Category**
- Search input below header (global filter)

### Sections (hard separation)
1. Incoming
2. Outgoing
3. Archived (collapsed by default)

---

## Row Structure (Desktop)

Each category row shows:
- Icon (optional; fallback glyph)
- Name
- Optional description (1 line clamp)
- Metadata:
  - Type (Incoming/Outgoing) — shown for parents; implied for children
  - Transaction count (optional)
  - Status (Active/Archived)

Actions (always visible; no hover-only):
- Edit
- Archive (if allowed) / Restore (if archived)
- Delete (only if allowed)
- **+ Subcategory** (parents only, active only, and only if depth allows)

### Visual hierarchy
- Indentation only for children (no connector lines).
- Children indent by 16–24px.

---

## Mobile Behavior

- Layout becomes stacked cards.
- **Create/Edit uses Drawer** (bottom sheet).
- No hover-only controls.
- Actions always visible in card footer.

---

## Create / Edit Flows

### Create Category (Top-level)
Desktop: Modal  
Mobile: Drawer

Fields:
- Name (required)
- Type (required: Incoming/Outgoing)
- Icon (optional)
- Description (optional)

### Create Subcategory (only from parent row)
Triggered by **+ Subcategory** on a parent row.

Fields:
- Parent (frozen, shown)
- Type (inherited, frozen)
- Name (required)
- Icon (optional)
- Description (optional)

No generic parent dropdown.

### Edit Category
Editable:
- Name
- Icon
- Description

Conditionally editable:
- Type (only if `transaction_count == 0` and top-level)
- Parent (re-parenting) only if `transaction_count == 0` (UI may omit in v1)

If type/parent is frozen, show neutral helper text:
- "Type cannot be changed after the first transaction exists."
- "Parent cannot be changed once transactions exist."

---

## Confirmations

### Archive confirmation
Text:
- "This will archive the category. Historical transactions remain unchanged."
- "Archived categories cannot be used for new transactions."

If blocking (active children):
- "This category has active subcategories. Archive subcategories first."

### Delete confirmation
Only available when allowed.
Text:
- "This category has no transactions and no subcategories. It will be permanently removed."

---

## Empty / Loading / Error States

### Empty
- "No categories defined yet."
- CTA: "Add Category"

### Loading
- Skeleton rows grouped by sections to keep layout stable.

### Error
- "Categories could not be loaded."
- CTA: "Retry"

---

## Out of Scope (v1)
- Multi-level nesting (>1)
- Manual ordering / drag-and-drop
- Targets and deviation (belongs to Categories operational page)
- User-managed Transfer categories (system only)
