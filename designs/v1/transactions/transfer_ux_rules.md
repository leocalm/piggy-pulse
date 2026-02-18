# TRANSFER_UX_RULES

## Definition
A transfer moves value between two accounts.

## Invariants
- Single positive amount.
- From account must not equal To account.
- No direction sign in UI.
- Category = transfer.

## Auto-fill Logic
If account scope != All:
    From = scope account (disabled)
If account scope == All:
    From selectable

## UI Behavior
Switching from transfer → normal:
- Clear To account
- Clear validation errors

Switching normal → transfer:
- Vendor cleared
- From auto-filled
