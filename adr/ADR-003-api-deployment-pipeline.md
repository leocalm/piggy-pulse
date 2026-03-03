# ADR-003: API Deployment Pipeline – Hetzner, GitHub Actions, Drone CI

## Context
The PiggyPulse API is a Rust/Rocket service backed by PostgreSQL. It handles sensitive financial data and must be deployed reliably with strong security boundaries. The deployment pipeline must support automated testing, image building, and zero‑inbound‑access delivery to production — meaning the server should never expose SSH or accept incoming deployment connections.

## Decision
Adopt the following deployment infrastructure for the PiggyPulse API:
- **Hosting:** Hetzner Cloud (dedicated VPS)
- **CI (test + build + publish):** GitHub Actions
- **CD (pull + apply):** Drone CI (self‑hosted on Hetzner)
- **Artifact transport:** Container images via GitHub Container Registry (GHCR)

The pipeline works as follows:
1. **PR opened / push to main** → GitHub Actions runs `cargo fmt`, `cargo clippy`, `cargo test`, and SQLx compile‑time checks.
2. **Merge to main** → GitHub Actions builds the production Docker image and pushes it to GHCR.
3. **Drone CI on Hetzner** polls GHCR for new images, pulls them, and applies the update (container restart with health checks).

This architecture eliminates the need to open SSH or any inbound deployment port on the production server. Drone initiates all connections outbound.

## Alternatives Considered

### Hosting
- **AWS (EC2/ECS/Fargate):** Maximum flexibility and managed services, but significantly higher cost and operational complexity for a single‑service workload.
- **DigitalOcean:** Comparable VPS pricing, but Hetzner offers better price‑to‑performance ratio (especially in EU regions) and dedicated vCPU options.
- **Fly.io:** Edge‑deployed containers with minimal config, but less control over networking, higher per‑request cost at scale, and less predictable pricing.
- **Railway/Render:** Zero‑ops PaaS, but limited control over container runtime, networking, and persistence; pricing scales unfavorably.

*Hetzner was chosen for its excellent price‑to‑performance ratio, EU data residency (relevant for financial data), dedicated vCPU options, and full control over the server environment without managed‑service overhead.*

### CI/CD Architecture
- **GitHub Actions for everything (CI + CD via SSH deploy):** Simpler single‑platform pipeline, but requires opening an SSH port on the production server — an unnecessary attack surface for a financial application.
- **GitHub Actions + Watchtower:** GH Actions builds images, Watchtower auto‑pulls — simpler than Drone, but Watchtower lacks pipeline logic (no health checks, no rollback, no conditional deployment).
- **Full Drone CI (CI + CD):** Single pipeline tool, but Drone's GitHub integration for PR checks is weaker than native GH Actions, and running CI builds on the production server wastes its resources.
- **ArgoCD / FluxCD:** GitOps‑native CD, but designed for Kubernetes — overkill for a single‑container VPS deployment.

*The GH Actions (CI + build) → Drone (CD via pull) split was chosen because it eliminates inbound server access, leverages GH Actions' native PR integration for CI, and uses Drone's polling model for secure, pull‑based deployment. The production server never accepts incoming connections for deployment.*

### Container Registry
- **Docker Hub:** Widely used, but rate limits on free tier and less integration with GitHub‑native workflows.
- **Self‑hosted registry on Hetzner:** Full control, but adds operational burden and requires securing another service.
- **AWS ECR:** Robust, but adds AWS dependency and cross‑provider networking complexity.

*GHCR was chosen for its native GitHub Actions integration (no extra credentials), generous free tier for private images, and proximity to the CI pipeline.*

## Consequences

### Positive
- **Zero inbound attack surface:** No SSH port, no deployment webhook — Drone pulls outbound only.
- **Native CI integration:** GitHub Actions provides first‑class PR checks, status badges, and branch protection enforcement.
- **Cost efficiency:** Hetzner's pricing allows dedicated vCPU and ample storage at a fraction of AWS/GCP cost.
- **Clear separation:** CI concerns (test/lint/build) stay in GH Actions; deployment concerns (pull/apply/restart) stay on the server.
- **Reproducible deployments:** Docker images are immutable artifacts — what's tested is what's deployed.

### Negative
- **Two CI systems:** Maintaining both GH Actions workflows and Drone pipeline configuration adds cognitive overhead.
- **Polling latency:** Drone's pull‑based model introduces a small delay between image push and deployment (seconds to minutes depending on poll interval).
- **Self‑hosted Drone maintenance:** Drone on Hetzner requires updates and monitoring as an additional service.
- **Single‑server risk:** No managed orchestration means failover and scaling require manual intervention.

## Reversibility
- **Hetzner:** **Medium** – the application is containerized, so migrating to another VPS or cloud provider requires reprovisioning but no code changes.
- **GitHub Actions (CI):** **Medium** – workflows would need to be rewritten for another CI platform, but the underlying commands (`cargo test`, `docker build`) are portable.
- **Drone CI (CD):** **High** – Drone can be replaced with any pull‑based deployment tool (Watchtower, custom cron, etc.) since it only pulls and restarts containers.
- **GHCR:** **High** – switching registries requires updating image push/pull URLs in CI and Drone config only.

## Related Documents
- [`ADR-001-backend-technology-stack.md`](ADR-001-backend-technology-stack.md) – backend technology decisions
- [`ADR-002-frontend-technology-stack.md`](ADR-002-frontend-technology-stack.md) – frontend technology and Cloudflare Pages deployment
- [`ARCHITECTURE.md`](../ARCHITECTURE.md) – system architecture overview
- [`SECURITY.md`](../SECURITY.md) – security design principles and threat model
