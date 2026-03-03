# Architecture Decision Records

| ADR | Decision |
|-----|----------|
| [001](ADR-001-backend-technology-stack.md) | Backend technology stack — Rust, Rocket, SQLx, PostgreSQL |
| [002](ADR-002-frontend-technology-stack.md) | Frontend technology stack — React 19, Vite, Mantine v8, TanStack Query, Cloudflare Pages |
| [003](ADR-003-api-deployment-pipeline.md) | API deployment pipeline — Hetzner, GitHub Actions, Drone CI |
| [004](ADR-004-authentication-architecture.md) | Authentication architecture — Argon2id, HttpOnly sessions, CSRF, TOTP 2FA |
| [005](ADR-005-api-versioning-and-contract-strategy.md) | API versioning & contract strategy — URI path prefix, code‑first OpenAPI via rocket_okapi |
| [006](ADR-006-data-isolation-and-multi-tenancy.md) | Data isolation & multi‑tenancy — shared schema, application‑level row isolation |
| [007](ADR-007-dto-separation-pattern.md) | DTO separation pattern — explicit request/response types |
| [008](ADR-008-multi-repo-structure.md) | Multi‑repo structure — separate repos with overview layer |
| [009](ADR-009-licensing-agplv3.md) | Licensing — AGPLv3 |
