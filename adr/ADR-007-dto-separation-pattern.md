# ADR-007: DTO Separation Pattern – Explicit Request/Response Types

## Context
PiggyPulse's Rust API handles sensitive financial data with strict consistency requirements. The API boundary separates internal domain models (which evolve with business logic) from external representations (which must remain stable for consumers). Leaking internal structure through the API creates tight coupling between domain evolution and contract stability, and risks exposing internal fields (audit metadata, soft‑delete flags, internal IDs) to consumers.

## Decision
Adopt the following DTO separation strategy:
- **Strict DTO separation:** Domain types are never serialized directly to API responses or deserialized directly from API requests
- **Explicit mapping:** Every API endpoint maps between domain types and dedicated request/response DTOs
- **Direction‑specific DTOs:** Separate structs for input (create/update requests) and output (responses) — no shared "model" struct serving both directions
- **No derive‑based auto‑exposure:** Domain types do not derive `Serialize`/`Deserialize` for API use; only DTOs do

## Alternatives Considered

- **Expose domain types directly:** Simplest approach — derive `Serialize` on domain structs and return them from handlers. Eliminates mapping boilerplate, but tightly couples the API contract to internal representation. Any domain refactor risks breaking the API, and internal fields are easily leaked.
- **Shared input/output DTO:** A single DTO struct used for both requests and responses. Reduces struct count, but conflates creation fields (e.g., password) with response fields (e.g., created_at), requiring `Option` wrappers and runtime validation for fields that are contextually required.
- **DTO with `#[serde(skip)]` on internal fields:** Domain types with selective field hiding. Fragile — a forgotten skip annotation silently leaks data. The "safe by default" property is inverted: fields are exposed unless explicitly hidden, rather than hidden unless explicitly exposed.
- **Auto‑mapping libraries (e.g., `derive_more`, custom macros):** Reduce mapping boilerplate via procedural macros. Adds compile‑time magic that obscures the mapping, making it harder to audit what's exposed.

*Explicit DTO separation with manual mapping was chosen because it makes the API surface fully intentional — every exposed field is a deliberate choice in a dedicated struct. The mapping code is verbose but auditable, and Rust's type system ensures that domain changes that affect DTOs produce compile‑time errors rather than silent contract breakage.*

## Consequences

### Positive
- **Intentional API surface:** Every field in a response is a deliberate choice, not an accident of internal modeling.
- **Independent evolution:** Domain types can be refactored without affecting the API contract, and vice versa.
- **Security by default:** Internal fields (soft‑delete flags, audit columns, internal IDs) are never accidentally exposed.
- **Compile‑time safety:** Changes to domain types that affect the mapping produce compiler errors, not runtime surprises.
- **Validation boundary:** Input DTOs serve as the validation boundary — constraints are expressed in the DTO, not the domain type.

### Negative
- **Boilerplate:** Every endpoint requires dedicated request/response structs and explicit `From`/`Into` implementations.
- **Struct proliferation:** The number of types grows with the API surface, increasing codebase size.
- **Mapping maintenance:** Changes to domain types require updating the corresponding DTOs and mapping logic.

## Reversibility
- **DTO separation pattern:** **Medium** – collapsing DTOs back into domain types is mechanically straightforward but would sacrifice the safety properties and require auditing every exposed field.

## Related Documents
- [`ADR-001-backend-technology-stack.md`](ADR-001-backend-technology-stack.md) – backend technology decisions (Rust type system context)
- [`ADR-005-api-versioning-and-contract-strategy.md`](ADR-005-api-versioning-and-contract-strategy.md) – API contract stability and versioning
- [`AGENTS.md`](../AGENTS.md) – coding conventions ("Domain types are never exposed directly")
- [`SECURITY.md`](../SECURITY.md) – security design principles
