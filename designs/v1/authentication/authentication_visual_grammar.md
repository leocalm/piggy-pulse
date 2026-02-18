# AUTHENTICATION_VISUAL_GRAMMAR.md

## Purpose

Define the visual and interaction grammar for authentication flows in PiggyPulse.
Authentication must feel:

- Structurally serious
- Emotionally neutral
- Calm and reflective
- Security-aware without being alarming

---

## 1. State Model

Every authentication screen supports these states:

- idle
- typing
- validating
- submitting
- error
- success
- locked
- two_factor_required

No additional emotional or visual states.

---

## 2. Input Field Behavior

### Default
- Neutral border
- Neutral background
- No validation shown

### While Typing
- No inline validation errors
- No red borders
- Do not penalize exploration

### On Blur
- Validate format only (email format, minimum length, etc.)
- If invalid → subtle helper text below field
- Border uses accent-primary (never red)

### After Submit Attempt
- Field-level errors only for format issues
- Server rejections → form-level message only

Never expose:
- User not found
- Email incorrect
- Password incorrect

Only:
> Login failed. Please check your credentials.

---

## 3. Loading Behavior

When submitting:

- Disable submit button
- Replace button label with spinner + “Signing in…”
- Layout must remain stable
- No full-screen dimming
- No page jumps

### Timing Rules
- Backend enforces constant response time
- Frontend enforces minimum visible loading of ~400ms
- Prevents timing inference from visual feedback

---

## 4. Error Density Rule

Only one visible error layer at a time:

- Field-level OR
- Form-level

Never stack multiple error messages.

Authentication must not feel chaotic.

---

## 5. Two-Factor (TOTP) Flow

After successful credential verification:

Display:
> Enter your 6-digit verification code.

Do NOT mention:
- 2FA configuration
- Account detection
- Security triggers

### Code Input Behavior

- Single input or 6 segmented inputs (choose globally)
- Paste support
- Auto-focus progression (if segmented)
- Auto-submit on completion

Error message:
> Verification failed. Try again.

No details about expiration or correctness.

---

## 6. Forgot Password Flow

Submission message (always):

> If this email is registered, you will receive instructions shortly.

Never confirm account existence.

Reset page:
- Inline password strength validation (zxcvbn)
- Strength meter uses neutral scale
- No aggressive red states

---

## 7. Visual Tone

Avoid:
- Red backgrounds
- Warning icons
- Shaking animations
- High-contrast alerts

Use:
- Soft focus rings
- Stable layout
- Predictable spacing
- Subtle helper text

Authentication should feel composed and intentional.

---

## 8. Lock State (If Triggered)

If account access is temporarily restricted:

> Login temporarily unavailable. Please try again later.

Do NOT mention:
- Suspicious activity
- Too many attempts
- Timers or countdowns

Maintain opacity.

---

## 9. Accessibility & Keyboard

- Logical tab order
- Enter submits form
- Escape does not exit auth screens
- Screen readers announce errors politely
- Labels must exist independently of placeholders

---

## 10. Motion & Transitions

- Fade transitions: 150–200ms
- No shaking
- No bouncing
- No dramatic motion

PiggyPulse avoids theatrical UI.

Authentication is deliberate, calm, and structurally sound.
