# Security Policy

PiggyPulse is a security-informed SaaS platform. This document describes security principles, threat assumptions, supported versions, and how to report vulnerabilities.

---

## Reporting a Vulnerability

If you believe you’ve found a security issue, please report it responsibly.

Preferred: open a private security advisory on GitHub (if enabled for the repository).  
Alternative: open an issue requesting a private communication channel.

Please include:

- A clear description of the issue
- Steps to reproduce (proof-of-concept if possible)
- Impact assessment (what an attacker can do)
- Any relevant logs, requests, or screenshots (redact secrets)

Please do not include:

- Real user data
- Credentials or active tokens
- Exploit payloads that enable broad abuse

---

## Supported Versions

Security updates apply to:

- The current production deployment
- The latest version on the default branch

Older versions are not maintained.

If you are self-hosting PiggyPulse, it is recommended to stay aligned with the latest release.

---

## Disclosure Timeline

PiggyPulse follows a responsible disclosure process:

1. Report received and acknowledged.
2. Issue reproduced and assessed.
3. Patch developed and tested.
4. Fix deployed to production.
5. Public disclosure (if appropriate).

The goal is to minimize user risk while maintaining transparency.

There is no formal bug bounty program at this time.

---

## Security Design Principles

PiggyPulse is built with:

- Strong boundary separation between internal models and external API contracts
- Defense-in-depth at authentication and session boundaries
- “Assume the code is public” design (no reliance on obscurity)

Security is treated as a structural property of the system rather than a feature.

---

## Authentication & Session Security

### Session Model

- Authentication uses HttpOnly session cookies
- No authentication tokens are stored in browser localStorage
- All requests are validated server-side on every call

This reduces the impact of client-side script compromise by keeping credentials inaccessible to JavaScript.

### CSRF Protection

- CSRF protection is enforced for state-changing operations
- The backend validates CSRF measures on every applicable request

### Two-Factor Authentication (2FA)

- Optional 2FA support adds an additional verification layer
- Recovery mechanisms are supported where applicable

### Password Recovery

- Password reset uses time-limited tokens
- Tokens are single-use and invalidated upon reset
- Reset flows are delivered via email

---

## Password Hashing

- Passwords are hashed using Argon2 with per-user salts and memory-hard parameters
- Weak passwords are rejected at the boundary using password strength estimation (`zxcvbn`)

The author previously contributed to Lyra, a Password Hashing Competition candidate, during graduate research. That background informs the selection and configuration of modern password hashing strategies.

---

## Abuse Mitigation

- Rate limiting is applied to sensitive endpoints (e.g., login and password recovery)
- Authentication flows are designed to mitigate brute-force and credential-stuffing attempts

---

## Data Isolation

- PiggyPulse is multi-user and enforces user-level data isolation
- Financial entities are owned by a user and access is enforced at query boundaries

---

## Secrets Management

- Secrets are never committed to the repository
- Production configuration is provided via environment variables
- Secrets should be rotated upon suspected compromise

Sensitive values include:

- Database credentials
- Secret keys
- SMTP credentials
- Token signing keys

---

## OpenAPI / API Documentation Exposure

- The public OpenAPI contract is treated as a first-class artifact
- Documentation is hosted separately from the production runtime
- Swagger UI is not served directly from the production API service

---

## Non-Goals (Current Scope)

The project intentionally does not include (yet):

- Full observability stack (metrics/log aggregation)
- Role-based access control (RBAC)
- Dedicated staging environment
- Advanced anomaly detection

These are planned improvements as the platform evolves.

---

## Security Update Policy

Security-related changes are prioritized and reviewed carefully.

Changes affecting:

- Authentication
- Session handling
- Token logic
- Password hashing
- Authorization boundaries

Require explicit review before release.

The API contract is treated as a public boundary and must not change in ways that weaken security guarantees.
