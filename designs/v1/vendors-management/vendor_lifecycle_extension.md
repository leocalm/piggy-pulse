# VENDOR_LIFECYCLE_EXTENSION.md

This document extends ENTITY_LIFECYCLE_RULES.md with vendor-specific behavior.

## Vendor States

Active:
- Default state
- Available in transaction autocomplete

Archived:
- Hidden from autocomplete
- Visible in management page
- Historical transactions remain unchanged

Deleted:
- Only allowed if no transactions reference vendor

## Autocomplete Behavior

- Case-insensitive matching
- Trim whitespace before matching
- Archived vendors are excluded from suggestions
- If archived vendor name is typed exactly:
  → Option A: Prevent creation
  → Option B (recommended): Auto-unarchive silently

V1 Decision:
Archived vendors remain excluded. No auto-unarchive in V1.

## Invariant

Vendor changes never alter financial history.
