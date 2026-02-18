# AUTHENTICATION_MICROCOPY_RULES.md

## Purpose

This document defines the microcopy and tone rules for all authentication-related interfaces in PiggyPulse.

Authentication is security-critical, but it must remain calm, neutral, and non-alarming.

We never:
- Expose whether a user exists.
- Expose whether an email is registered.
- Expose whether 2FA is enabled.
- Reveal internal validation specifics.
- Use emotional or punitive language.

---

# Core Tone Principles

## 1. Calm Reflection

Language should be:
- Neutral
- Clear
- Direct
- Non-judgmental

Avoid:
- Urgency language ("Act now", "Immediate action required")
- Emotional triggers ("Warning!", "Danger!", "Critical error!")
- Blame ("Incorrect password", "You entered wrong data")

Prefer:
- "Login unsuccessful."
- "Unable to complete request."
- "If this email is registered, you will receive instructions."

---

# Login Microcopy Rules

## Success

"Login successful."
Redirect immediately.

No celebratory messaging.

---

## Failure (Generic)

Always show:

"Login unsuccessful. Please verify your credentials and try again."

Never show:
- "User not found"
- "Email not registered"
- "Incorrect password"
- "2FA required"

All failure states collapse into the same message.

---

## Rate Limit / Lock

"When too many attempts are detected:"

"Login temporarily unavailable. Please try again later."

Do NOT disclose:
- Lock duration
- Whether lock is account-based or IP-based

---

# 2FA Microcopy Rules

## Prompt

"Enter your authentication code."

Subtext:
"Use your authenticator app."

No explanation of:
- Which factor failed
- Whether password was correct

---

## Invalid Code

"Authentication unsuccessful. Please try again."

Same tone as login failure.

---

# Password Recovery Microcopy

## Request Submitted

Always show:

"If this email is registered, you will receive reset instructions."

Never:
- Confirm registration
- Confirm absence

---

## Token Invalid / Expired

"Reset link is no longer valid. Request a new one."

Do not specify:
- Expired vs invalid
- Token tampering suspicion

---

# Reset Password Rules

## Success

"Password updated successfully."

Immediately redirect to login.

---

## Weak Password

"Password does not meet security requirements."

Do NOT disclose:
- Strength score
- Internal hashing rules
- Detailed entropy metrics

Optional hint (generic):
"Use a longer and less predictable combination."

---

# Timing Consistency Principle

UI copy must never vary based on:
- User existence
- Account state
- Lock state

Backend ensures consistent timing via dummy hash computation.
Frontend must maintain identical UI behavior.

---

# Email Microcopy Rules

## Password Reset Email

Subject:
"Reset your PiggyPulse password"

Body tone:
- Neutral
- No urgency pressure
- Clear expiration mention (without exact timestamp if desired)

Example:

"You requested a password reset.

If this was you, use the link below to continue.
If not, you can ignore this message."

---

## 2FA Setup Email (if applicable in future)

Subject:
"Your PiggyPulse authentication setup"

Tone:
Informational, not celebratory.

---

# Visual Consistency Rules

Authentication pages must:

- Use PiggyPulse branding.
- Maintain soft contrast.
- Avoid red-heavy danger styling.
- Use neutral emphasis colors.

Security does not equal fear.

---

# Absolute Invariants

1. Never reveal user existence.
2. Never reveal 2FA status before password validation.
3. Never leak structural details in error messages.
4. Never differentiate error copy by backend branch.
5. All errors must feel identical in weight and tone.

---

# Design Philosophy

PiggyPulse authentication is:

Calm.
Deliberate.
Quietly secure.
