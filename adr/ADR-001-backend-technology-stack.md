# ADR-001: Backend Technology Stack – Rust, Rocket, SQLx, PostgreSQL

## Context
PiggyPulse is a reflective financial‑clarity platform that must manage sensitive financial data, enforce strong consistency guarantees, and provide a reliable foundation for long‑term user trust. The backend is responsible for transaction processing, period‑based calculations, user authentication, and serving a versioned API. Architectural choices must align with the product’s core values of **safety, reliability, and clarity** while supporting an iterative, evidence‑driven development process.

## Decision
Adopt the following technology stack for the PiggyPulse backend:
- **Language:** Rust  
- **Web framework:** Rocket  
- **Database toolkit:** SQLx  
- **Database system:** PostgreSQL  

This stack provides a cohesive foundation that emphasizes compile‑time safety, explicit async‑capable APIs, and strong data‑integrity guarantees.

## Alternatives Considered

### Language
- **Go:** Excellent concurrency model and faster compile times, but lacks Rust’s memory‑safety guarantees without a garbage collector and has a weaker type system.  
- **Python/Node.js:** High developer velocity and rich ecosystems, but dynamic typing and runtime errors increase the risk of subtle financial‑calculation bugs; performance and memory safety are secondary.  
- **Java/JVM:** Mature ecosystem and strong typing, but runtime overhead, garbage‑collection pauses, and less control over memory layout were considered unsuitable for a system where predictable latency and resource efficiency are valued.

*Rust was chosen because its ownership model eliminates whole classes of memory‑safety and data‑race bugs at compile time, its zero‑cost abstractions allow high performance without sacrificing expressiveness, and its type system enables precise domain modeling—all critical for a financial application.*

### Web Framework
- **Actix‑web:** Extremely high performance and mature async support, but its actor‑based model adds conceptual overhead and its configuration is more verbose.  
- **Axum:** Tower‑based, flexible, and gaining adoption, but still relatively young and lacking some of Rocket’s built‑in conveniences (e.g., request guards, fairings).  
- **Warp:** Filter‑based combinator API, lightweight, but can become difficult to reason about in larger applications.

*Rocket was selected for its developer‑friendly, type‑safe routing, built‑in request‑validation mechanisms (guards, fairings), and strong focus on ergonomics without sacrificing async capabilities. Its “simple by default” philosophy aligns with PiggyPulse’s goal of architectural clarity.*

### Database Toolkit
- **Diesel:** A full‑featured ORM with a powerful query‑builder DSL and strong compile‑time guarantees, but its synchronous core requires separate async wrappers and its DSL introduces a learning curve.  
- **sea‑orm:** A modern async ORM built on SQLx, offering higher‑level abstractions, but adds another layer of complexity and is still evolving.  
- **Raw database drivers:** Maximum flexibility but no compile‑time query checking and higher boilerplate.

*SQLx provides a balance: it validates SQL queries at compile time (against a live database schema), supports async/await natively, and exposes a lightweight, type‑safe API that stays close to SQL. This matches the need for explicit, auditable data‑access patterns while preserving async performance.*

### Database System
- **MySQL:** Widely deployed and performant, but its historical limitations around strict SQL compliance, transactional DDL, and advanced data types (e.g., full JSON support) made it a second choice.  
- **SQLite:** Excellent for embedded/desktop scenarios, but not designed for concurrent multi‑user web applications; lacks built‑s in replication and advanced constraint features.

*PostgreSQL was chosen for its rigorous ACID compliance, rich feature set (JSONB, check constraints, partial indexes), extensibility, and strong reputation for data integrity. Its consistency guarantees are essential for financial data where historical accuracy must never be silently corrupted.*

## Consequences

### Positive
- **Memory safety:** Rust’s ownership model eliminates entire categories of security vulnerabilities (use‑after‑free, data races) that could compromise financial data.
- **Performance:** Zero‑cost abstractions and minimal runtime overhead enable low‑latency responses and efficient resource usage.
- **Type safety:** The compiler enforces invariants across the entire stack, reducing runtime errors and increasing confidence in financial calculations.
- **Async readiness:** Native async/await support in Rust, Rocket, and SQLx allows efficient I/O‑bound workloads without blocking threads.
- **Data integrity:** PostgreSQL’s strict SQL compliance and advanced constraint features ensure transactional consistency and historical accuracy.
- **Compile‑time validation:** SQLx checks queries against the live database schema at compile time, catching many data‑access errors before deployment.

### Negative
- **Learning curve:** Rust’s ownership model and the stack’s async patterns require upfront investment from developers.
- **Ecosystem maturity:** The Rust web ecosystem, while growing rapidly, is less mature than those of Go, Python, or Java; some libraries may be missing or less battle‑tested.
- **Compilation speed:** Rust’s compile times are slower than those of Go or interpreted languages, potentially slowing local iteration.
- **Operational complexity:** PostgreSQL requires more operational expertise than embedded databases and may introduce additional deployment considerations.

## Reversibility
- **Language (Rust):** **Low** – switching the core language would necessitate a full rewrite of the backend.
- **Web framework (Rocket):** **Medium** – the framework is encapsulated behind route handlers and request guards; migrating to another Rust framework would require significant refactoring but not a full rewrite.
- **Database toolkit (SQLx):** **Medium** – queries are written in raw SQL, so moving to another SQL‑first toolkit (or even an ORM) is feasible with moderate effort.
- **Database system (PostgreSQL):** **Low** – changing the database would require a complex migration and likely break SQL‑specific features (JSONB, window functions, etc.). Schema and data would need to be transformed.

## Related Documents
- [`vision_and_mission.md`](../piggypulse_product_alignment_docs/vision_and_mission.md) – product philosophy and core values
- [`dashboard_strategy.md`](../piggypulse_product_alignment_docs/dashboard_strategy.md) – product‑alignment principles
- [`ai_agent_implementation_prompt.md`](../piggypulse_product_alignment_docs/ai_agent_implementation_prompt.md) – existing mention of Rust backend separation
- [`AGENTS.md`](../AGENTS.md) – repository layout and product‑thesis references