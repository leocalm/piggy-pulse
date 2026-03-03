# ADR-005: API Versioning & Contract Strategy – URI Path Prefix, Code‑First OpenAPI

## Context
PiggyPulse exposes a versioned REST API consumed by its own SPA frontend and potentially by future clients (mobile, third‑party integrations). The API serves as a public boundary — changes to it affect every consumer. The project needs a clear strategy for how the API is versioned, how breaking changes are managed, and how the contract is defined and validated.

## Decision
Adopt the following API versioning and contract strategy:
- **Versioning scheme:** URI path prefix (`/api/v1`, `/api/v2`, …)
- **Breaking‑change policy:** Breaking changes require a major version bump; no silent breaking changes
- **Contract format:** OpenAPI 3.x, generated code‑first from Rocket route definitions
- **Contract review:** OpenAPI spec reviewed before release — treated as a public boundary
- **Spec hosting:** Static Swagger UI served from `piggy-pulse-docs`

## Alternatives Considered

### Versioning Scheme
- **Header‑based versioning (`Accept: application/vnd.piggypulse.v1+json`):** Keeps URLs clean, but harder to test (requires custom headers in every request), less visible in logs, and poorly supported by browser dev tools and link sharing.
- **Query parameter (`?version=1`):** Simple to add, but easy to omit accidentally, caching is fragile, and it conflates versioning with query semantics.
- **No versioning (evolve in place):** Minimizes overhead, but any breaking change risks silently breaking consumers — unacceptable for a financial API where data integrity is paramount.
- **Content negotiation (GraphQL‑style):** Maximum client flexibility, but adds query complexity, makes caching harder, and introduces a fundamentally different API paradigm.

*URI path prefixing was chosen for its explicitness, visibility, and simplicity. The version is obvious in every URL, trivially routable at the reverse proxy layer, cache‑friendly, and impossible to accidentally omit. For a single‑SPA consumer where URL aesthetics are irrelevant, path‑based versioning has no practical downsides.*

### Contract Approach
- **Contract‑first (spec written manually, code implements it):** Ensures the spec is always intentional and reviewed, but creates a synchronization burden — the spec and code can drift, requiring additional validation tooling.
- **Hybrid (generated then hand‑edited):** Captures code reality and allows manual refinement, but hand‑edits are overwritten on next generation unless a merge workflow is maintained.
- **No formal contract:** Lowest overhead, but consumers have no reliable reference, and breaking changes are discovered at runtime.

*Code‑first generation was chosen because it guarantees the spec always reflects the actual implementation — no drift, no stale documentation. The spec is generated from Rocket route definitions and Rust types, leveraging the compiler's type safety. Pre‑release review of the generated spec serves as the intentionality check.*

### Breaking‑Change Policy
- **Semantic versioning with deprecation windows:** Formal deprecation period before removal — appropriate for public APIs with many consumers, but heavyweight for a single‑consumer SPA where the frontend and API are deployed in lockstep.
- **Additive‑only evolution (never break):** Avoids versioning complexity entirely, but accumulates cruft and constrains API design over time.
- **Feature flags per consumer:** Maximum flexibility, but adds routing complexity and makes the API surface unpredictable.

*A strict "breaking changes require a version bump" policy was chosen as the simplest guarantee. Since the SPA and API are co‑deployed, version transitions can be coordinated. The policy prevents silent breakage while avoiding the overhead of formal deprecation windows.*

## Consequences

### Positive
- **Explicitness:** The version is visible in every request URL — no ambiguity about which contract a consumer is using.
- **Zero drift:** Code‑first generation ensures the OpenAPI spec always matches the running implementation.
- **Safe evolution:** Breaking changes are isolated to new version prefixes; existing consumers are unaffected until they opt in.
- **Type safety:** Rust types flow through to the generated spec, catching contract errors at compile time rather than runtime.

### Negative
- **Path clutter:** Version prefixes add noise to every URL and route definition.
- **Parallel maintenance:** Supporting multiple live versions means maintaining parallel route handlers until old versions are retired.
- **Code‑first limitations:** Generated specs may lack the narrative documentation and examples that hand‑written specs provide; supplemental documentation may be needed.
- **Review discipline:** The spec must be reviewed before each release to catch unintended contract changes — this is a process requirement, not an automated guarantee.

## Reversibility
- **URI path versioning:** **Low** – changing the versioning scheme would break all existing consumer URLs and require coordinated migration.
- **Code‑first OpenAPI:** **Medium** – switching to contract‑first requires writing the spec manually and adding validation that code matches it, but doesn't affect the API itself.
- **Breaking‑change policy:** **High** – the policy is a convention, not a technical constraint; it can be adjusted at any time.

## Related Documents
- [`ADR-001-backend-technology-stack.md`](ADR-001-backend-technology-stack.md) – backend technology decisions (Rocket context)
- [`AGENTS.md`](../AGENTS.md) – API versioning summary and conventions
- [`ARCHITECTURE.md`](../ARCHITECTURE.md) – system architecture overview
