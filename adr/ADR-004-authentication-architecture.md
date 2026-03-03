# ADR-004: Authentication Architecture – Argon2id, HttpOnly Sessions, CSRF, TOTP 2FA

## Context
PiggyPulse manages sensitive financial data and requires a robust authentication layer that protects user credentials at rest, prevents session hijacking, and resists common web attacks (CSRF, credential stuffing, XSS‑based token theft). The platform is a traditional server‑rendered SPA backed by a stateful Rust API — not a distributed microservice mesh — so the auth model can prioritize simplicity and security over federation or token portability. The lead developer is a co‑author of Lyra, a finalist in the Password Hashing Competition (PHC), bringing direct expertise in memory‑hard password hashing design to the selection and configuration of the hashing strategy.

## Decision
Adopt the following authentication architecture for PiggyPulse:
- **Password hashing:** Argon2id with per‑user salts and memory‑hard parameters
- **Session model:** Server‑side sessions with HttpOnly, Secure, SameSite=Strict cookies
- **CSRF protection:** Server‑validated CSRF tokens on all state‑changing requests
- **2FA:** Optional TOTP‑based second factor
- **Password quality:** Boundary‑level rejection of weak passwords via zxcvbn estimation
- **No tokens in browser:** No JWTs, no access tokens in localStorage or sessionStorage

## Alternatives Considered

### Password Hashing
- **bcrypt:** Well‑established and widely deployed, but fixed memory cost (~4 KB) makes it vulnerable to GPU/ASIC attacks at scale. Its 72‑byte input limit silently truncates long passwords.
- **scrypt:** Memory‑hard and configurable, but couples CPU and memory cost parameters, making independent tuning difficult. Less reviewed than Argon2 post‑PHC.
- **PBKDF2:** NIST‑recommended and FIPS‑compliant, but purely CPU‑bound with no memory hardness — highly parallelizable on GPUs and ASICs.
- **Lyra2:** Memory‑hard PHC finalist with competitive performance and tunable parameters. Co‑authored by the lead developer. However, Argon2id won the PHC and has received significantly more third‑party scrutiny, library support, and ecosystem adoption since 2015. Choosing the competition winner over one's own candidate reflects a principled preference for community consensus.

*Argon2id was chosen because it combines Argon2i's resistance to side‑channel attacks with Argon2d's resistance to GPU/ASIC cracking. Its independently tunable time, memory, and parallelism parameters allow precise cost calibration. As the winner of the Password Hashing Competition (2015), it represents the current consensus best practice. The lead developer is a co‑author of Lyra, one of the PHC finalist candidates, bringing direct expertise in memory‑hard password hashing design to the selection and parameter tuning of Argon2id.*

### Session Model
- **JWT in localStorage:** Stateless and portable, but exposes tokens to XSS attacks (JavaScript can read localStorage). Revocation requires a server‑side blocklist, negating the stateless benefit.
- **JWT in HttpOnly cookie:** Better XSS resistance, but JWTs are self‑contained and harder to revoke mid‑session. Token size grows with claims, inflating every request.
- **Token‑based auth (opaque tokens in headers):** Requires client‑side token management in JavaScript, re‑introducing XSS exposure. Common in mobile/API contexts but unnecessary for a browser SPA.

*Server‑side sessions with HttpOnly cookies were chosen because they keep session identifiers completely inaccessible to JavaScript, support immediate server‑side revocation, and add zero client‑side complexity. For a single‑origin SPA talking to its own API, stateless token portability provides no benefit — and the security trade‑offs are unfavorable.*

### CSRF Protection
- **SameSite=Strict cookies alone:** Prevents cross‑origin cookie attachment, but older browsers and certain redirect flows may not enforce it reliably.
- **Double‑submit cookie pattern:** No server‑side state needed, but vulnerable to subdomain cookie injection if the domain structure is not tightly controlled.
- **Origin/Referer header checking:** Lightweight, but headers can be stripped by proxies or privacy extensions.

*Explicit CSRF tokens validated server‑side were chosen as the primary defense, with SameSite=Strict as a defense‑in‑depth layer. Belt‑and‑suspenders: the CSRF token covers edge cases where SameSite alone might not suffice.*

### Two‑Factor Authentication
- **SMS‑based OTP:** Familiar to users, but vulnerable to SIM‑swapping and SS7 interception — inappropriate for a financial application.
- **WebAuthn/FIDO2:** Phishing‑resistant and hardware‑backed, but higher implementation complexity and user friction for initial adoption. Planned as a future enhancement.
- **Email‑based OTP:** Simple, but adds latency and depends on email delivery reliability; email accounts themselves may be compromised.

*TOTP (RFC 6238) was chosen for its balance of security, user familiarity, and implementation simplicity. It works with standard authenticator apps (Google Authenticator, Authy, 1Password), requires no SMS infrastructure, and resists phishing better than SMS. WebAuthn is earmarked as a future addition.*

## Consequences

### Positive
- **Credential protection:** Argon2id's memory hardness makes offline cracking prohibitively expensive even with GPU/ASIC hardware.
- **Session security:** HttpOnly cookies eliminate an entire class of XSS‑based session theft.
- **CSRF resilience:** Layered defense (token + SameSite) protects against cross‑origin state mutations.
- **Simplicity:** Server‑side sessions avoid the complexity of token lifecycle management, refresh flows, and client‑side storage security.
- **Immediate revocation:** Sessions can be invalidated server‑side instantly (logout, password change, suspicious activity).

### Negative
- **Server‑side state:** Sessions require server‑side storage and management (memory or database‑backed), adding a stateful component to the API.
- **Single‑origin coupling:** Cookie‑based auth ties the frontend to the same origin as the API; future mobile clients or third‑party integrations would need a separate auth mechanism.
- **2FA adoption friction:** Optional TOTP means not all users will enable it; the system must remain secure without it.
- **Parameter tuning burden:** Argon2id parameters must be calibrated to the production server's resources and periodically re‑evaluated as hardware improves.

## Reversibility
- **Argon2id:** **Low** – changing the hashing algorithm requires re‑hashing all passwords (typically done lazily on next login), and the new algorithm must be at least as strong.
- **HttpOnly cookie sessions:** **Low** – switching to token‑based auth would require rearchitecting the frontend auth flow, API middleware, and client‑side storage strategy.
- **CSRF tokens:** **High** – the CSRF mechanism is encapsulated in middleware and can be swapped or augmented independently.
- **TOTP 2FA:** **Medium** – replacing the 2FA method requires migration of enrolled users' second factors and UI changes, but the core session model is unaffected.

## Related Documents
- [`ADR-001-backend-technology-stack.md`](ADR-001-backend-technology-stack.md) – backend technology decisions (Rust/Rocket context)
- [`SECURITY.md`](../SECURITY.md) – security design principles, threat model, and password hashing policy
- [`AGENTS.md`](../AGENTS.md) – tech stack summary and frontend auth conventions
