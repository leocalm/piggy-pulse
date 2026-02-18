
# INTERACTION_LANGUAGE.md

## Core Principle

PiggyPulse interaction must reflect:

Calm Reflection.

The interface is consistent, predictable, and emotionally neutral.
Interaction patterns must not vary arbitrarily across pages.

---

# 1. Creation Grammar

## Desktop

- Repetitive actions → Inline-first (e.g., transactions quick-add row).
- Complex configuration → Modal.
- Destructive confirmations → Modal only.

## Mobile

- Creation and editing → Drawer (bottom sheet).
- No hover-dependent controls.
- No hidden action-only affordances.

---

# 2. Editing Grammar

- Editing must preserve layout context.
- Inline editing preferred when repetitive.
- Drawer editing for mobile or complex forms.
- Visual shift must not disorient layout hierarchy.

Save Feedback:
- Subtle confirmation.
- No celebratory animation.
- No dramatic success messaging.

---

# 3. Destructive Action Grammar

- Always explicit confirmation.
- Neutral warning tone.
- No emotional alarm styling.
- Clearly state consequences.

Example:

"This will delete 3 future periods. Past periods remain unchanged."

---

# 4. Filtering Grammar

- Filters always appear at top of page.
- Same layout across Dashboard, Categories, Accounts, Transactions.
- Reset behavior consistent.
- No hidden filtering logic.

Filters must not change semantic meaning of metrics silently.

---

# 5. Selection & Focus Grammar

- Keyboard navigation supported where applicable.
- Tab order predictable.
- Enter saves when context is clear.
- Escape cancels consistently.

Focus state visible but subtle.

---

# 6. Feedback Grammar

Allowed:
- "Saved."
- "Updated."
- "Deleted."

Not allowed:
- "Great job!"
- "Warning!"
- "You're on track!"
- "You overspent!"

Tone must remain descriptive, not advisory.

---

# 7. Transfer Interaction Rules

- Transfer switches form fields to From / To accounts.
- Auto-fill last used account when applicable.
- Direction invariant must be respected.
- Transfer never impacts category spending.

UI must reflect domain invariants without exposing unnecessary complexity.

---

# 8. Allowance Interaction Rules

- Allowance balance can go negative.
- No blocking when overspending allowance.
- Negative balance displayed neutrally.
- Next period transfer corrects position automatically.

No emotional framing.

---

# 9. Chart Interaction Rules

- Hover reveals additional detail, never changes scale dramatically.
- Projection visually lighter than actual values.
- No dramatic animations on load.

---

# 10. Consistency Requirements

Across all pages:

- Position displayed first.
- Same terminology used everywhere (Target, Deviation, Balance).
- Same destructive confirmation tone.
- Same creation model hierarchy.

If a page violates these rules, it must be refactored.
