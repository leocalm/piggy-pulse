# AGENTS.md

## Purpose

This repo is the PiggyPulse product, architecture, ADR, design, and agent-reference repo. Agents should use it for product intent, design specs, governance, and architectural rationale, while treating AgentBrain as durable memory and implementation repos as source of truth for shipped code.

## Memory-first workflow

AgentBrain is the durable project memory source. Before meaningful work:

1. Read the memory manifest.
2. Read PiggyPulse project context.
3. Read relevant project memory files.
4. Search decisions.
5. Check open questions.
6. Inspect this repo.

If AgentBrain, this repo, and implementation code disagree, stop and report the mismatch. Do not silently update specs to match assumptions.

## Required memory reads

- Always: `Context.md`, `ArchitectureOverview.md`, `ProductPrinciples.md`, `DesignSystem.md`, `KnownIssues.md`.
- Architecture/ADR work: `Backend.md`, `Frontend.md`, `APIConventions.md`, `SecurityModel.md`, `DataModel.md`, `Deployment.md`, relevant decisions.
- Design/spec work: `DesignSystem.md`, `ProductPrinciples.md`, relevant `Features/*`.
- Backend/frontend/mobile coordination: the relevant repo memory under `Repos/` plus feature files.

## Memory write-back rules

After meaningful work, record durable decisions, open questions, interaction/session summaries, and requested daily/global summaries.

Use MCP tools if available: `record_decision`, `upsert_open_question`, `record_interaction`, `append_daily_log`, `append_global_daily_summary`.

If MCP is unavailable, write to `AgentBrain/10_Projects/PiggyPulse/`. Do not store secrets, `.env` contents, credentials, private keys, tokens, signing material, or raw chain-of-thought.

## Repo overview

Product/reference repo containing ADRs, design specs, wireframes, doctrine, agent role references, architecture notes, and product alignment docs for PiggyPulse.

## Important directories

- `adr/` - architecture decision records.
- `designs/v1/` - v1 product specs, wireframes, HTML mocks, and tracking.
- `references/doctrine/` - product doctrine, governance, interaction language, and keyboard navigation.
- `references/agents/` - agent role contracts.
- `references/figma/` - Figma-related references if present.
- `docs/plans/` - implementation/design plans.
- `piggypulse_product_alignment_docs/` - vision, mission, epics, and dashboard strategy.
- `ARCHITECTURE.md`, `SECURITY.md`, `README.md` - overview docs.

## Commands

No package manager, build, test, lint, or format tooling is verified in this repo.

### Install

No install command is verified.

### Development

No development command is verified.

### Build

No build command is verified.

### Test

No test command is verified.

### Lint / format

No lint or format command is verified.

### Database / migrations

Not applicable.

### Mobile platform commands

Not applicable.

## Conventions

- Keep specs concise and tied to implementation repos or AgentBrain references.
- Product tone is descriptive, not prescriptive; informative, not judgmental; no shame-based financial language.
- Design direction is calm Nebula finance UI with soft purple, dusty rose, and muted blue accents.
- For UI work, read `designs/v1/design-system.md`, the relevant feature spec, and `references/doctrine/interaction_language.md`.
- ADRs should record decisions, context, consequences, and affected repos.
- Do not treat old design mocks as implementation truth when they conflict with AgentBrain or current code.

## Testing expectations

For docs/spec changes, review affected cross-references and verify links/filenames. If a spec claims implementation behavior, inspect the relevant implementation repo or mark it as a TODO/open question.

## Security / privacy rules

- Never commit secrets or `.env` values.
- Do not include signing keys, provisioning profiles, keystores, API tokens, service-account files, passwords, or private certificates.
- Public-facing security/privacy claims must be checked against `SecurityModel.md` and implementation repos.
- Follow `PrivacyRules.md` only if the task touches personal/career/user memory; otherwise project security rules apply.

## Environment variables

None documented for this repo.

## When to stop and ask/report

Stop and report if memory contradicts source, implementation repos contradict specs, a public API/security claim is uncertain, required commands are missing, legal/privacy text needs owner approval, or the requested change conflicts with recorded decisions.

## Completion checklist

Before final response, verify:

- [ ] Relevant AgentBrain memory was read.
- [ ] Relevant source files were inspected.
- [ ] Existing decisions and open questions were checked.
- [ ] Commands run are listed.
- [ ] Tests/lint/build were run where appropriate, or skipped with reason.
- [ ] Durable decisions were recorded or proposed.
- [ ] Open questions were recorded or proposed.
- [ ] No secrets or `.env` values were exposed.
- [ ] Any memory/code contradictions were reported.
