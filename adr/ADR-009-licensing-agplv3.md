# ADR-009: Licensing – GNU Affero General Public License v3

## Context
PiggyPulse is a SaaS platform deployed as a network service. The choice of open‑source license determines what obligations apply to third parties who modify and deploy the software, and signals the project's stance on commercial use, contribution, and source availability. For a SaaS product, the distinction between permissive licenses (MIT, Apache) and copyleft licenses (GPL, AGPL) is particularly consequential — permissive licenses allow proprietary forks to be deployed as competing services without sharing modifications.

## Decision
- **License:** GNU Affero General Public License v3 (AGPLv3)

## Alternatives Considered

- **MIT:** Maximum adoption potential — anyone can use, modify, and redistribute with minimal restrictions. However, a competitor could fork PiggyPulse, modify it, and deploy it as a competing SaaS product without publishing their changes. For a self‑funded product, this gives away the entire codebase with no reciprocity.
- **Apache 2.0:** Similar permissiveness to MIT with added patent protection. Same SaaS vulnerability — no obligation to share modifications deployed as a network service.
- **GPLv3:** Strong copyleft requiring source disclosure for distributed modifications. However, the "distribution" trigger does not cover SaaS deployment — running modified code as a network service is not considered distribution under GPLv3, so competitors could deploy proprietary forks without sharing.
- **BSL (Business Source License):** Source‑available with a time‑delayed open‑source conversion. Prevents commercial competition during the protection period, but is not OSI‑approved and may deter contributors who prefer recognized open‑source licenses.
- **SSPL (Server Side Public License):** Designed specifically for SaaS protection (used by MongoDB), but not OSI‑approved and its broad "service" definition creates legal uncertainty that may discourage adoption and contributions.

*AGPLv3 was chosen because it closes the "SaaS loophole" present in GPLv3 — any party that modifies PiggyPulse and deploys it as a network service must publish their modifications under the same license. This protects the project's competitive position while remaining a recognized, OSI‑approved open‑source license that encourages genuine open‑source contribution.*

## Consequences

### Positive
- **SaaS protection:** Competitors cannot deploy proprietary forks as competing services without publishing their changes.
- **OSI‑approved:** AGPLv3 is a recognized open‑source license, maintaining legitimacy in the open‑source ecosystem.
- **Contribution incentive:** Modifications must be shared, ensuring improvements flow back to the community.
- **Transparency signal:** Aligns with PiggyPulse's values of clarity and trust — the source is always available.

### Negative
- **Corporate adoption friction:** Some organizations have blanket policies against AGPL dependencies, limiting potential enterprise contributions or integrations.
- **Contributor hesitation:** Some developers avoid AGPL projects due to perceived legal complexity or concerns about copyleft obligations.
- **Dual‑licensing complexity:** If commercial licensing is desired in the future (e.g., offering a proprietary license to enterprises), all contributors must assign copyright or agree to relicensing — requiring a CLA.

## Reversibility
- **AGPLv3:** **Low** – relicensing requires consent from all copyright holders. If external contributions have been accepted without a CLA, relicensing may be legally impractical. Establishing a CLA early mitigates this.

## Related Documents
- [`AGENTS.md`](../AGENTS.md) – repository overview
- [`README.md`](../README.md) – project overview and license reference
