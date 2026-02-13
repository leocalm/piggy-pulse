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
