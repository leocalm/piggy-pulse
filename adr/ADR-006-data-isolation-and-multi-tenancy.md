# ADR-006: Data Isolation & Multi‑Tenancy – Shared Schema, Application‑Level Isolation

## Context
PiggyPulse is a multi‑user SaaS platform where each user's financial data (transactions, accounts, categories, periods) must be strictly isolated. A data leak between users would be a critical security and trust failure. The isolation model must be simple, auditable, and resistant to accidental cross‑user data exposure while remaining practical for a single‑database, single‑service architecture.

## Decision
Adopt the following data isolation strategy:
- **Tenancy model:** Shared database, shared schema, user‑level row isolation
- **Enforcement layer:** Application‑level — every data‑access query includes an explicit `user_id` filter
- **Ownership model:** All financial entities have a `user_id` foreign key; ownership is enforced at the query boundary, not at the ORM or database layer
- **No RLS:** PostgreSQL Row‑Level Security is not used; isolation is enforced entirely in the Rust data‑access layer

## Alternatives Considered

### Tenancy Model
- **Database‑per‑tenant:** Strongest isolation (separate PostgreSQL instances per user), but operationally impractical for a consumer SaaS with potentially thousands of users. Connection management, migrations, and backups scale linearly with user count.
- **Schema‑per‑tenant:** Each user gets a dedicated PostgreSQL schema within a shared database. Better isolation than shared‑schema, but migration complexity scales with tenant count and cross‑tenant queries (admin, analytics) become difficult.
- **Shared schema with RLS:** Shared tables with PostgreSQL Row‑Level Security policies filtering by `user_id`. Provides database‑level enforcement as a safety net, but adds complexity to the migration and testing workflow, and RLS policies can mask query behavior in ways that are hard to debug.
- **Shared schema, application‑level isolation:** All users share the same tables; every query explicitly filters by `user_id` in the Rust data‑access layer.

*Shared schema with application‑level isolation was chosen for its simplicity, auditability, and alignment with the codebase's explicit‑SQL philosophy (SQLx with hand‑written queries). Every query's isolation boundary is visible in the SQL — there are no hidden RLS policies silently filtering results. For a single‑service Rust backend with compile‑time query checking, this provides sufficient guarantees without database‑level complexity.*

### Enforcement Layer
- **ORM‑level scoping (automatic tenant filter):** Some ORMs support default scopes that inject `WHERE user_id = ?` automatically. Reduces boilerplate but hides the isolation logic, making it harder to audit and easier to bypass with raw queries.
- **PostgreSQL RLS as safety net:** Application queries filter by `user_id`, and RLS policies provide a secondary enforcement layer. Defense‑in‑depth, but the added RLS complexity (session variables, policy management, migration overhead) was deemed unnecessary given Rust's type safety and SQLx's compile‑time query validation.
- **Middleware‑level injection:** A request middleware extracts the authenticated user and injects the `user_id` into every query automatically. Reduces repetition but obscures the data‑access pattern and complicates testing.

*Explicit application‑level enforcement was chosen because it keeps isolation visible and auditable at the query level. Combined with SQLx's compile‑time checks and Rust's type system, the risk of accidentally omitting a `user_id` filter is mitigated by code review discipline and the explicit query patterns enforced by the team's conventions.*

## Consequences

### Positive
- **Auditability:** Every query's tenant boundary is visible in the SQL — no hidden database‑level policies to reason about.
- **Simplicity:** No RLS policies to manage, no session variables to set, no special migration handling.
- **Testability:** Isolation can be tested with straightforward integration tests that assert cross‑user data is inaccessible.
- **Explicit SQL alignment:** Consistent with the project's SQLx philosophy of explicit, hand‑written, compile‑time‑checked queries.

### Negative
- **No database‑level safety net:** A missing `WHERE user_id = ?` in a query could expose cross‑user data. Mitigation: code review, compile‑time query checks, integration tests.
- **Repetitive filter logic:** Every data‑access query must include the `user_id` filter, increasing boilerplate.
- **Scaling limits:** Shared‑schema isolation works well at moderate scale but may require re‑evaluation if tenant‑level performance isolation becomes necessary.

## Reversibility
- **Tenancy model (shared schema):** **Low** – migrating to schema‑per‑tenant or database‑per‑tenant would require significant data migration and architectural changes.
- **Enforcement layer (application‑level):** **Medium** – adding RLS as a supplementary layer is additive and can be done incrementally without changing application code.

## Related Documents
- [`ADR-001-backend-technology-stack.md`](ADR-001-backend-technology-stack.md) – backend technology decisions (SQLx explicit‑SQL philosophy)
- [`SECURITY.md`](../SECURITY.md) – data isolation policy and security design principles
- [`AGENTS.md`](../AGENTS.md) – coding conventions for backend data access
