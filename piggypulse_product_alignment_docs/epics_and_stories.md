# Epics & Stories – Dashboard Alignment

## Epic 1 – Period Status Refactor

Goal:
Make period clarity the central dashboard element.

### Stories

- Implement unified period summary endpoint (BE)
- Calculate tolerance-adjusted alignment status (BE)
- Replace red warning copy with neutral descriptive language (FE)
- Add remaining days contextual display (FE)
- Ensure edits to periods dynamically affect calculated metrics (BE)

Definition of Done:
- Period status derived only from current time frame.
- No hard-coded month assumptions.
- Clear neutral language for over/under states.

---

## Epic 2 – Alignment Consistency Metric

Goal:
Surface long-term behavioral signal instead of isolated variance.

### Stories

- Compute alignment consistency percentage (BE)
- Add rolling N-period calculation (BE)
- Display alignment trend card (FE)
- Allow optional projection toggle (future planned enhancement)

Definition of Done:
- KPI derived from historical periods.
- Tolerance buffer applied globally (e.g., 5%).
- No coaching copy.

---

## Epic 3 – Savings Evolution (Optional Enhancement)

Goal:
Provide reflective long-term financial trajectory without coaching.

### Stories

- Aggregate net difference across savings accounts (BE)
- Display trend over time (FE)
- Keep neutral framing (e.g., “Savings change over selected range”)

Definition of Done:
- No goal-based celebratory UI.
- Pure descriptive visualization.

---

## Epic 4 – Dashboard Simplification

Goal:
Reduce cognitive noise.

### Stories

- Remove recent transactions card (FE)
- Remove redundant top categories chart (FE)
- Ensure dashboard focuses on period + trend + balance only

Definition of Done:
- Dashboard reflects product thesis.
- No filler components.
