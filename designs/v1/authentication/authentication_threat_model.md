# AUTHENTICATION_THREAT_MODEL.md

## Purpose

This document defines the security assumptions, invariants, and threat model for
PiggyPulse authentication.

It exists to prevent accidental security regressions as the product evolves.

---

# 1. Security Philosophy

PiggyPulse authentication must be:

- Calm (no fear-based UX)
- Reflective (clear about consequences)
- Non-enumerating (never leaks user existence)
- Structurally secure (defense-in-depth)
- Deterministic (no inconsistent edge behavior)

Security is enforced at the system level, not via UI intimidation.

---

# 2. Attacker Model

We assume attackers may:

- Attempt credential stuffing
- Attempt brute force login
- Attempt timing-based enumeration
- Attempt email enumeration via password recovery
- Attempt token replay (TOTP reuse)
- Attempt session fixation
- Attempt CSRF attacks

We do NOT assume:

- Nation-state adversaries
- Physical device compromise mitigation
- Browser malware mitigation

---

# 3. Core Authentication Invariants

## 3.1 Non-Enumeration Rule

Authentication responses MUST NOT reveal:

- Whether a user exists
- Whether an email is registered
- Whether a password was incorrect
- Whether TOTP failed specifically

All failures return the same neutral message:

> "Login failed. Please verify your credentials."

Password recovery always returns:

> "If this email is registered, you will receive a message shortly."

---

## 3.2 Timing Attack Protection

- Password verification must always perform Argon2 hashing
- If user does not exist, perform dummy Argon2 hash
- Response timing must remain statistically consistent

---

## 3.3 TOTP Constraints

- TOTP codes valid only within configured time window
- No recovery fallback codes (unless explicitly designed later)
- Failed TOTP does not disclose which factor failed

---

## 3.4 Password Storage

- Argon2id hashing
- Proper salt per user
- Memory-hard parameters tuned for production hardware
- No reversible encryption

---

# 4. Session Model Assumptions

- Secure HTTP-only cookies
- SameSite protection
- CSRF protection enabled
- Session invalidation on logout
- Session rotation after login

---

# 5. Recovery Flow Rules

- Recovery tokens are time-limited
- Tokens are single-use
- Reset does not auto-login user
- Reset invalidates all prior sessions

---

# 6. UX-Security Alignment

Security must feel:

- Calm
- Neutral
- Deterministic
- Non-judgmental

No red panic warnings.
No emotional language.
No blame wording.

---

# 7. Structural Guarantees

The following must NEVER change without ADR:

- Non-enumeration rule
- Dummy hash timing defense
- Argon2 usage
- TOTP as second factor (when enabled)
- Reset token invalidation policy

---

# 8. Future Considerations

Potential evolutions:

- Rate limiting spec
- IP throttling policy
- Device trust model
- Session lifetime policy
- WebAuthn support

All must be added via ADR.

---

END OF DOCUMENT
