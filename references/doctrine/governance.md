# Spec Governance

## Purpose
Ensure design and behavior are defined before implementation.

---

## Roles
- **Spec Owner**: Defines behavioral contracts.
- **Reviewer**: Validates coherence and consistency.
- **Implementer**: Builds strictly from spec.
- **Design Agent**: Generates mock HTML following spec.

---

## Repository Structure

```
designs/v1/
  <feature>/
    <feature>_spec.md
    <feature>_wireframe.md
    piggypulse-<feature>.html
adr/
  ADR-NNN-<title>.md
references/
  agents/
  doctrine/
  figma/
assets/
designs/
  prototypes/
```

---

## Spec Lifecycle

1. Draft
2. Review
3. Lock
4. Version (v1.0, v1.1, etc.)

Implementation must reference locked version.

---

## Change Process

- Changes require ADR or spec amendment.
- No silent behavior changes.
- Diff must explain WHY.

---

## Versioning

Each major UX shift:
- Increment minor version.

Structural model change:
- Increment major version.

---

## Review Flow

- No parallel feature specs.
- One spec completed before next.
- UI mock must align with spec before implementation begins.

---

## Philosophy

Spec first.
Design second.
Implementation third.
