# AGENTS.md

## What This Repo Is

PiggyPulse is a security-informed SaaS platform for personal financial clarity. This root repository is the portfolio/overview layer. The actual components live in separate repositories:

- `piggy-pulse-api` — Rust + Rocket backend service
- `piggy-pulse-web` — React + Vite frontend client
- `piggy-pulse-docs` — OpenAPI documentation (static)

## Design System

All product specs, wireframes, and HTML mocks live in `designs/`. This is the canonical source for UI behavior, interaction language, and visual conventions.

- Active v1 specs: `designs/v1/` (feature folders, each with spec + wireframe + HTML mock)
- Exploratory prototypes: `designs/prototypes/`
- Product doctrine: `references/doctrine/` (interaction language, governance, keyboard nav, design decisions)
- Agent role contracts: `references/agents/`
- Figma tokens/components: `references/figma/`
- Architecture decisions: `adr/`
- Product alignment: `piggypulse_product_alignment_docs/`

**Before generating any UI, spec, or wireframe — read `designs/v1/design-system.md` and the relevant feature spec.**

## Product Thesis

PiggyPulse is a reflective financial clarity platform:
- Descriptive, not advisory
- Neutral, not judgmental
- No gamification
- Magnitude over morality

Core doctrine: `references/doctrine/calm_reflection_principles.md`

## Non-Negotiable UX Rules

- Never coach users — no "you should", no punitive framing, no moral labels
- No red/green emotional coding for financial performance
- Period is the cognitive anchor; no silent scope switching
- Transfer semantics = single logical entity in UI
- Amounts are always positive in inputs; direction comes from context/category

References: `references/doctrine/interaction_language.md`, `references/doctrine/notes_on_design_decisions.md`

## Source-of-Truth Precedence

1. Feature specs in `designs/v1/**/*.md`
2. Governance + lifecycle rules: `references/doctrine/governance.md`, `designs/v1/entity_lifecycle_rules.md`
3. Wireframes (`*_wireframe.md`, HTML mocks)
4. Figma docs (`references/figma/*`) — implementation aids only, not behavioral authority

## Tech Stack

### Backend (`piggy-pulse-api`)
- Language: Rust
- Framework: Rocket
- Database: PostgreSQL + SQLx
- Auth: HttpOnly cookie sessions, Argon2 hashing, optional 2FA
- Design: explicit DTO separation, compile-time guarantees, SQL transparency, stateless requests

### Frontend (`piggy-pulse-web`)
- Framework: React + Vite
- UI: Mantine
- Deployment: Cloudflare Pages
- Design: thin client, no sensitive token storage in browser, cookie-based auth, version-bound API usage

### API
- Versioned under `/api/v1`
- Breaking changes require version bump
- OpenAPI contract reviewed before release — treated as a public boundary
- No silent breaking changes

## Coding Conventions

### Backend
- Domain types are never exposed directly — always map through DTOs
- No silent data mutations; all writes are explicit
- Errors are typed and propagated structurally, not swallowed
- SQL queries are explicit (no heavy ORM magic); use SQLx with typed queries
- Security-sensitive endpoints must have rate limiting
- No secrets in source; use environment variables

### Frontend
- API calls go through a versioned client layer — never call endpoints ad hoc
- No sensitive data in localStorage or sessionStorage
- Currency formatting must use the global currency context (not hardcoded symbols)
- Components are thin — business logic belongs in hooks or services
- Follow the visual system baseline from `designs/v1/design-system.md`

## Visual System Baseline (for Frontend)
- Max content width: 1100px
- Desktop nav: left sidebar (240px) — Core / Structure / Session groups
- Mobile nav: fixed bottom bar — Dashboard, Transactions, Periods, More
- `More` opens a bottom drawer: Accounts, Categories, Vendors, Settings
- Loading states: ghost skeleton shimmer
- Page titles use `Sora`; body/data copy stays sober
- No theme toggle controls in production wireframes

Full reference: `designs/v1/ux-infrastructure/ux_infrastructure_spec.md`

## Naming Conventions (Design Files)
- Specs: `snake_case_spec.md`
- Wireframes: `snake_case_wireframe.md`
- HTML mocks: `piggypulse-<page>.html` in `designs/v1/<feature>/`
- HTML prototypes: `piggypulse_<page>_vNN_<label>.html` in `designs/prototypes/`

## Active Tracking
- v1 page status: `designs/v1/piggypulse_v1_master_tracking.md`
- Architecture decisions: `adr/`
