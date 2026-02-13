# PiggyPulse

PiggyPulse is a security-informed SaaS platform for flexible personal budgeting.

It consists of a versioned backend API and a mobile-first web client, deployed independently and governed by explicit contract boundaries.

Production:
- App: https://piggy-pulse.com
- API: https://api.piggy-pulse.com
- Documentation: https://docs.piggy-pulse.com
- Storybook: https://piggypulse-storybook.pages.dev/

---

## System Overview

PiggyPulse is designed around three core principles:

- Security-first authentication
- Explicit domain modeling
- Clean separation of concerns

The system is composed of:

- `piggy-pulse-api` — Rust backend service
- `piggy-pulse-web` — React frontend client
- `piggy-pulse-docs` — OpenAPI documentation (static)

Each component is deployed independently.

---

## Architecture

High-level structure:

Browser Client  
↓  
Versioned API (`/api/v1`)  
↓  
Domain Layer  
↓  
PostgreSQL  

Key properties:

- HttpOnly cookie-based authentication
- CSRF protection
- Argon2 password hashing
- DTO separation between internal models and external contracts
- Stateless API design
- OpenAPI as a first-class artifact
- Dockerized deployment

---

## Security Philosophy

Security is not treated as a feature but as a structural property of the system.

Authentication model:

- HttpOnly session cookies
- No tokens stored in localStorage
- Backend validation on every request
- Rate limiting on sensitive endpoints
- Optional 2FA support
- Token-based password recovery with expiration

Password hashing uses Argon2 with memory-hard parameters.

The author previously contributed to Lyra, a Password Hashing Competition candidate, during graduate research. That background informs the authentication design and selection of modern password hashing strategies.

---

## Repository Structure

Backend:
https://github.com/leocalm/budget

Frontend:
https://github.com/leocalm/budget-app

API documentation:
https://docs.piggy-pulse.com

---

## Deployment Model

- Containerized services
- CI-driven deployment
- Automatic migrations on startup
- Independent frontend and backend pipelines
- Separate documentation hosting

Deployment is reproducible via Docker.

---

## API Governance

- Versioned under `/api/v1`
- Breaking changes require version bump
- OpenAPI contract reviewed before release
- No silent breaking changes
- Contract treated as public boundary

---

## License

PiggyPulse is licensed under the GNU Affero General Public License v3.0 (AGPLv3).

You are free to use, modify, and self-host the software.  
If you run a modified version as a network service, you must make the modified source code available under the same license.

See the LICENSE file for full details.
