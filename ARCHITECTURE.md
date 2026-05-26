# Architecture

PiggyPulse is structured as a layered, contract-driven SaaS system.

---

## Backend

Language: Rust  
Framework: Rocket  
Database: PostgreSQL  
Data Layer: SQLx  

Design characteristics:

- Explicit DTO separation
- Compile-time guarantees
- Manual mapping between domain and response objects
- SQL transparency (no heavy ORM abstraction)
- Stateless request handling

---

## Frontend

Framework: React + Vite  
UI: Mantine  
Deployment: Cloudflare Pages  

Design characteristics:

- Thin client
- No sensitive token storage in browser
- Cookie-based authentication
- Environment-based API configuration
- Version-bound API usage

---

## Encryption Layer

PiggyPulse encrypts all user financial data at rest using per-user AES-256-GCM keys:

- Each user has a unique Data Encryption Key (DEK)
- The DEK is wrapped (encrypted) with a Key Encryption Key derived from the user's password via Argon2id
- The plaintext DEK is never stored on disk — it exists in client memory and, after login, in a server-side per-session store
- Both web and iOS clients implement the full client-side encryption/decryption stack
- Encrypted columns span seven database tables (transactions, accounts, categories, vendors, budgets, subscriptions)

For the full design, see [ADR-010: Encryption at Rest](adr/ADR-010-encryption-at-rest.md).

```
Browser Client  or  iOS Client
↓
AES-256-GCM encrypt/decrypt (client-side)
↓
Versioned API (`/v1`)
↓
Domain Layer
↓
Encrypted PostgreSQL
```

---

## Documentation

OpenAPI specification is generated at runtime and consumed by a statically hosted Swagger UI.

Swagger UI is not exposed directly from the production API runtime.

---

## Deployment

- Dockerized backend
- CI-enforced linting and testing
- Controlled container restarts
- Independent frontend pipeline

Future improvements include:

- Observability stack
- Structured health checks
- Staging environment
