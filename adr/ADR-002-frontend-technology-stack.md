# ADR-002: Frontend Technology Stack – React, Vite, Mantine, TanStack Query, Cloudflare Pages

## Context
PiggyPulse's frontend is a thin client that renders financial data served by the Rust API. It must be fast, accessible, and security‑conscious (no tokens in localStorage, cookie‑based auth). The UI follows a calm, reflective design language — no gamification, no moral coding. The frontend needs a component library that supports this neutral aesthetic, a fast build pipeline for iteration, and a deployment target that provides global edge delivery with minimal ops overhead.

## Decision
Adopt the following technology stack for the PiggyPulse frontend:
- **UI Library:** React 19
- **Build Tool:** Vite
- **Component Library:** Mantine v8
- **Server‑State Management:** TanStack Query (React Query)
- **Deployment:** Cloudflare Pages

This stack provides a cohesive foundation that emphasizes developer velocity, accessible component defaults, declarative data management, and zero‑ops global delivery.

## Alternatives Considered

### UI Library
- **Vue 3:** Excellent DX, smaller bundle, but smaller ecosystem for financial/enterprise UI libraries and fewer candidates familiar with it.
- **Svelte/SvelteKit:** Minimal runtime overhead and elegant reactivity, but younger ecosystem, fewer component libraries, and higher risk for long‑term maintenance.
- **Solid.js:** Superior performance via fine‑grained reactivity, but nascent ecosystem and limited component library options.

*React was chosen for its ecosystem depth, mature tooling, broad hiring pool, and first‑class support from Mantine. React 19's server components and improved concurrent features align with future optimization paths.*

### Build Tool
- **Webpack:** Battle‑tested and highly configurable, but slow dev‑server startup and complex configuration.
- **esbuild (direct):** Extremely fast but lacks the plugin ecosystem and dev‑server conveniences needed for a full SPA workflow.
- **Turbopack:** Promising Webpack successor, but still in beta and not production‑ready.

*Vite was chosen for its near‑instant HMR, native ESM dev server, Rollup‑based production builds, and first‑class React support. It dramatically reduces local iteration time.*

### Component Library
- **Chakra UI:** Accessible and composable, but v3 migration introduced instability and the styling approach (emotion) adds runtime overhead.
- **shadcn/ui:** Excellent control via copy‑paste primitives, but requires building and maintaining more infrastructure (no built‑in data tables, date pickers, notifications).
- **Ant Design:** Feature‑rich enterprise library, but opinionated visual style that conflicts with PiggyPulse's calm, neutral aesthetic, and large bundle size.
- **MUI (Material UI):** Comprehensive and mature, but Material Design's visual language is prescriptive and harder to neutralize for a non‑opinionated financial UI.

*Mantine v8 was chosen for its headless‑friendly architecture, native CSS modules (no runtime CSS‑in‑JS), rich built‑in components (dates, notifications, modals, data tables), excellent accessibility defaults, and a neutral visual baseline that aligns with PiggyPulse's reflective design language.*

### Server‑State Management
- **SWR:** Lightweight and simple, but fewer features for cache invalidation, optimistic updates, and query coordination.
- **Redux Toolkit Query (RTK Query):** Powerful but couples data fetching to Redux, adding unnecessary global state machinery for a thin‑client architecture.
- **Manual fetch + useState:** No library overhead, but reinvents caching, deduplication, retry, and stale‑while‑revalidate logic.

*TanStack Query was chosen for its declarative cache management, automatic background refetching, built‑in optimistic updates, and clean separation of server state from UI state — critical for a financial app where data freshness and consistency matter.*

### Deployment
- **Vercel:** Excellent DX and Next.js integration, but PiggyPulse uses Vite (not Next.js), and Vercel's pricing scales less favorably.
- **Netlify:** Comparable static hosting, but Cloudflare's edge network is larger and Workers integration provides future extensibility.
- **AWS S3 + CloudFront:** Maximum control, but significantly more operational overhead for a static SPA.

*Cloudflare Pages was chosen for its global edge network, zero‑config static deployments, generous free tier, native Workers integration for future edge logic, and minimal ops burden.*

## Consequences

### Positive
- **Developer velocity:** Vite's near‑instant HMR enables rapid local iteration.
- **Rich component set:** Mantine provides accessible, production‑ready components with no runtime CSS overhead.
- **Data freshness:** TanStack Query's declarative cache management ensures consistent, up‑to‑date financial data.
- **Global delivery:** Cloudflare Pages provides edge‑cached static hosting with near‑zero operational burden.
- **Ecosystem depth:** React's mature ecosystem offers broad library support and a large talent pool.

### Negative
- **Bundle size:** React's bundle is larger than Svelte or Solid alternatives.
- **Migration risk:** Mantine major‑version upgrades may require non‑trivial migration effort.
- **Learning curve:** TanStack Query's declarative cache patterns require upfront investment from developers unfamiliar with the approach.
- **Deployment coupling:** Cloudflare Pages introduces mild platform lock‑in (mitigated by the app being a standard static SPA deployable anywhere).

## Reversibility
- **React:** **Low** – switching UI libraries would require a full rewrite of the frontend.
- **Vite:** **High** – build tools are swappable with configuration changes; application code is unaffected.
- **Mantine:** **Medium** – component imports are pervasive but encapsulated; migration to another library is significant but incremental.
- **TanStack Query:** **Medium** – hooks are localized to data‑fetching layers; replacing requires rewriting fetch logic but not UI components.
- **Cloudflare Pages:** **High** – the app is a standard static SPA deployable to any static hosting provider.

## Related Documents
- [`ADR-001-backend-technology-stack.md`](ADR-001-backend-technology-stack.md) – backend technology decisions
- [`vision_and_mission.md`](../piggypulse_product_alignment_docs/vision_and_mission.md) – product philosophy and core values
- [`AGENTS.md`](../AGENTS.md) – repository layout, tech stack summary, and frontend conventions
- [`design-system.md`](../designs/v1/design-system.md) – visual system baseline
- [`ux_infrastructure_spec.md`](../designs/v1/ux-infrastructure/ux_infrastructure_spec.md) – UX infrastructure specification
