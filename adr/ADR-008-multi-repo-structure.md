# ADR-008: Multi‑Repo Structure – Separate Repos with Overview Layer

## Context
PiggyPulse consists of three independently deployable components: a Rust API backend, a React SPA frontend, and a static OpenAPI documentation site. The codebase needs a repository structure that supports independent CI/CD pipelines, clear ownership boundaries, and deployment autonomy — the frontend deploys to Cloudflare Pages while the API deploys to Hetzner via a completely different pipeline.

## Decision
Adopt the following repository structure:
- **Repository structure:** Multi‑repo with a root overview repository
- **Repositories:**
  - `piggy-pulse-api` — Rust backend (independent CI/CD to Hetzner)
  - `piggy-pulse-app` — React frontend (independent CI/CD to Cloudflare Pages)
  - `piggy-pulse-docs` — Static OpenAPI/Swagger documentation
  - `piggy-pulse` (this repo) — Product‑level documentation, ADRs, design specs, wireframes, and cross‑cutting governance

## Alternatives Considered

- **Monorepo (all components in one repository):** Single source of truth, atomic cross‑component commits, unified CI. However, Rust and Node.js have fundamentally different toolchains, dependency graphs, and build times. A monorepo would mean: CI runs both pipelines on every commit (wasteful), tooling must handle both ecosystems (Cargo + Yarn), and deployment coupling is implicit — a frontend typo fix triggers a backend CI run. Additionally, Cloudflare Pages and Hetzner/Drone deployments have no shared infrastructure.
- **Monorepo with path‑based CI filtering:** Monorepo with CI rules that only trigger pipelines for changed paths. Solves the wasted‑CI problem, but adds CI configuration complexity, and the temptation to share code across boundaries (direct imports between frontend and backend) undermines the clean API contract.
- **Multi‑repo without overview repo:** Each component is independent, no shared documentation layer. Simpler, but product‑level concerns (ADRs, design specs, product doctrine) have no home — they'd either be duplicated or arbitrarily placed in one component's repo.

*Multi‑repo with a root overview repository was chosen because it aligns repository boundaries with deployment boundaries, gives each component independent CI/CD pipelines with zero cross‑contamination, and provides a dedicated home for cross‑cutting product documentation. The API contract (OpenAPI spec) serves as the integration boundary — there is no shared code to manage.*

## Consequences

### Positive
- **Deployment independence:** Each component has its own CI/CD pipeline with no cross‑triggering.
- **Toolchain isolation:** Rust and Node.js toolchains don't interfere with each other (no shared lockfiles, no mixed dependency trees).
- **Clear ownership:** Each repo has a focused purpose and can evolve independently.
- **Documentation home:** Product‑level concerns (ADRs, design specs, governance) live in the overview repo without cluttering component repos.

### Negative
- **Cross‑repo coordination:** Changes that span the API contract require coordinated commits across repos (API first, then frontend).
- **No atomic commits:** A breaking API change and its frontend adaptation cannot be a single commit.
- **Discoverability:** New contributors must understand the multi‑repo layout before they can navigate the project.
- **Duplication risk:** Shared conventions (linting rules, commit formats) must be configured independently in each repo.

## Reversibility
- **Multi‑repo → monorepo:** **Medium** – git histories can be merged with `git subtree` or similar tools, but CI/CD pipelines, tooling, and deployment configurations would need significant rework.
- **Overview repo removal:** **High** – the overview repo contains only documentation and can be merged into any component repo or dissolved without affecting running systems.

## Related Documents
- [`ADR-003-api-deployment-pipeline.md`](ADR-003-api-deployment-pipeline.md) – API deployment to Hetzner via GH Actions + Drone
- [`ADR-002-frontend-technology-stack.md`](ADR-002-frontend-technology-stack.md) – frontend deployment to Cloudflare Pages
- [`AGENTS.md`](../AGENTS.md) – repository layout and component listing
- [`ARCHITECTURE.md`](../ARCHITECTURE.md) – system architecture overview
