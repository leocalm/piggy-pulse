# AUTHENTICATION_SPEC.md

## Philosophy

Authentication in PiggyPulse must reflect:
- Calm reflection
- Structural correctness
- Transparency
- No emotional escalation
- No exposure of user identity or system state

Security decisions are enforced at backend level (constant response time, Argon2 hashing, TOTP support).

---

## Global Rules

### Security
- Never expose whether a user exists.
- Identical failure messages for invalid credentials.
- Identical recovery messages whether email exists or not.
- Constant response timing.
- No detailed failure reasons during login.

### Visual
- Centered card layout.
- Pig logo + colorful PiggyPulse wordmark.
- Generous whitespace.
- Neutral tone.
- No dramatic red states.
- No warning icons or alarming language.

---

## Login

### Positioning Line
"Clarity begins with structure."

### Failure Message
"Login failed. Please check your credentials and try again."

---

## Two-Factor Authentication (TOTP)

### Failure Message
"Verification failed. Please try again."

Behavior:
- Auto-advance input.
- Paste supported.
- Enter submits.
- Backspace moves left.

---

## Forgot Password

### Post-Submit Message
"If the email is registered, you will receive a reset link shortly."

---

## Reset Password

### Invalid or Expired Token
"This reset link is invalid or expired."

Allowed validation:
- Password mismatch
- Minimum length
- Strength rules

---

## Email Specifications

### Password Reset Email

Subject: Password reset request

Content:
- Explanation of request
- Reset link
- Expiration notice
- Calm fallback statement

---

### Two-Factor Enabled Email

Subject: Two-factor authentication enabled

Neutral confirmation message.

---

### Password Changed Email

Subject: Password updated

Neutral confirmation message.
