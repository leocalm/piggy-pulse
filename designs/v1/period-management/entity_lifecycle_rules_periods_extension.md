# ENTITY_LIFECYCLE_RULES_PERIODS_EXTENSION.md

This document extends entity lifecycle rules with period semantics.

## Period Lifecycle
- Period status is derived from time:
  - Upcoming: now < start
  - Open: start <= now < end
  - Closed: now >= end

There is no manual close or reopen.

## Edit Rules
- Period boundaries (start/end) are immutable after creation.
- Period name may be editable.
- Targets are editable anytime.
- Transactions remain editable anytime, regardless of period status.

## Consequence Disclosure
Any UI that edits a transaction whose date falls into a closed period must disclose:
- “Saving updates historical totals and charts.”
