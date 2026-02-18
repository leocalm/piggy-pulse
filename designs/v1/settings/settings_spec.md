# PiggyPulse — Settings Specification
Generated: 2026-02-17

---

# 1. ROLE OF SETTINGS

Settings is the structural control layer of PiggyPulse.

It governs:
- Identity
- Security
- Structural configuration
- UI behavior
- Data ownership
- Account lifecycle

It is not a miscellaneous container. It is the control room.

---

# 2. INFORMATION ARCHITECTURE

Top-level sections:

1. Profile
2. Security
3. Period Model
4. Preferences
5. Data & Export
6. Danger Zone

Sections are domain-driven, not technical-grouped.

---

# 3. LAYOUT SYSTEM

## Desktop

Two-column layout:

Left:
- Vertical section navigation

Right:
- Section content panel

## Mobile

Single column:
- Accordion-style sections
- One expanded at a time

---

# 4. PROFILE

Purpose: Identity metadata

Fields:
- Name (editable)
- Email (read-only, change via verified flow)
- Timezone
- Default currency

States:
- View
- Editing
- Saving
- Success (toast)
- Error (inline)

Rules:
- Email change requires verification flow
- No auto-save
- Explicit Save required

---

# 5. SECURITY

Purpose: Access control

## Password
- Change password flow
- Requires current password
- Argon2 hashing enforced server-side

## Two-Factor Authentication
- Status display (Enabled / Disabled)
- Manage 2FA
- Regenerate recovery codes

## Active Sessions
- Device label
- Approximate location (country-level only)
- Last active timestamp
- Revoke session option

Rules:
- No IP exposure
- No device fingerprint exposure
- Revoking current session logs out immediately

---

# 6. PERIOD MODEL

Embeds schedule editor.

Displays:
- Mode (Automatic / Manual)
- Start day
- Duration
- Generate ahead
- Weekend rule
- Name pattern

Warning:
Changes regenerate future periods only.
Past and current remain unchanged.

---

# 7. PREFERENCES

Non-structural UI behavior.

- Theme (System / Light / Dark)
- Date format
- Number format
- Compact mode toggle

Rules:
- Immediate preview
- Explicit Save
- No feature flags allowed here

---

# 8. DATA & EXPORT

Purpose: User data ownership

Actions:
- Export transactions (CSV)
- Export full dataset (JSON)
- Request account data copy

Rules:
- Clear format explanation
- No silent downloads
- Confirmation before export

---

# 9. DANGER ZONE

Visually separated block.

## Reset Structure
Removes:
- Accounts
- Categories
- Period model

Transactions remain.

## Delete Account
Permanent deletion.
Requires typed confirmation: DELETE

Rules:
- Cannot be undone
- Explicit confirmation modal
- No optimistic deletion

---

# 10. VISUAL RULES

Cards:
- 1px soft border
- Radius 16–24px
- Surface background
- Padding 20–24px

Section Headers:
- Small uppercase label
- Strong section title

Danger Zone:
- Subtle red border
- No flashing
- Calm tone

---

# 11. INTERACTION CONTRACT

- Explicit Save
- Inline validation
- Toast on success
- Overlay confirmation for destructive actions
- No auto-save
- No silent mutations

---

# 12. ACCESS CONTROL

Settings accessible only to authenticated users.

Logout:
- Sidebar footer
- Settings → Security (session management)

Session expiration:
- Redirect to login
- Neutral messaging

---

# 13. ERROR HANDLING

All settings mutations follow unified rendering contract:

state = loading | success | error

Error:
- Inline message
- No technical stack traces
- Retry available

---

END OF SPEC
