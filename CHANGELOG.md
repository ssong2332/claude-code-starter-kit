# Changelog

All notable changes to this Starter Kit will be documented in this file.

This file tracks changes to the **kit itself**. It stays at the kit's root and is not copied into new projects — new projects get `docs/CHANGELOG.md` (a blank template) instead.

## [1.2.0] - 2026-07-24

### Added
- architect Security Design Checklist (auth, secrets, sensitive data, validation boundaries, attack surface, dependency risk, abuse cases) — outcomes recorded in Architecture.md's new Security section, which replaces the bare Authentication/Authorization section; every row requires a decision or an explicit "N/A — reason"
- planner Monetization capture — new PRD.md Monetization section (model / pricing intent / revenue constraints); planner never invents a business model: unstated-but-commercial becomes an Open Question, non-commercial products state "N/A — non-commercial" explicitly

## [1.1.0] - 2026-07-23

### Added
- Seventh subagent: **ux-design** (user flows, screen specs, Claude Design prompts for generating visual mockups externally)
- docs/UX.md and docs/UX-archive.md (overflow archive for retired UX entries; archived IDs stay reserved)
- docs/PRD.md "Planning Decisions" section — the authoritative record for planner's scope/priority decisions, separate from architect-owned docs/DECISIONS.md
- docs/DefinitionOfDone.md Gate/Closure split — Gate items block the `done` transition and are checked by quality-assurance; Closure items (docs sync, CHANGELOG) complete afterward at the docs step, closing a prior circular dependency
- `.claude/memory-protocol.md` — shared persistent-memory protocol referenced by planner/architect/implementer/ux-design, replacing ~140 duplicated lines per agent
- GitWorkflow.md "Who Merges" section — merges to `main` are the orchestrator's/user's job, not a subagent's; verify checkout before merging
- GitWorkflow.md merge policy decided: fast-forward preferred, merge-commit as fallback, never squash

### Changed
- docs agent granted read-only Bash (`git diff`/`status`/`log`) so it can execute its own documented workflow
- architect and implementer now treat docs/UX.md as Required-when-applicable instead of Optional; implementer likewise for docs/DefinitionOfDone.md and docs/GitWorkflow.md
- reviewer promoted to `opus` (was `sonnet`) so the verifier is never weaker than an `inherit`-model implementer; reviewer now always emits Action Items for Critical/Major findings regardless of APPROVED/REJECTED status
- Agent contract requires reviewer to run strictly before quality-assurance (QA's gate depends on reviewer's verdict)
- Document Priority reordered: README.md and docs/CHANGELOG.md now rank below the spec documents they're derived from
- Agent memory scope corrected from `project` to `user` (matches actual storage path); ux-design granted persistent memory to match the other design-phase agents
- PromptRules.md "pass reports verbatim" rule extended to every cross-agent relay, not just fix re-invocations

### Fixed
- ux-design's dependency on docs/DECISIONS.md corrected — planning-level exclusions live in PRD's Planning Decisions (planner-owned), not in architect-owned DECISIONS.md
- AGENTS.md Document Priority and Project Structure now list docs/UX-archive.md (was present in the ownership/contract tables but missing from these two)

## [1.0.0] - 2026-07-14

### Added
- Initial Claude Code Starter Kit
- Six project subagents
  - planner
  - architect
  - implementer
  - reviewer
  - quality-assurance
  - docs
- CLAUDE.md
- AGENTS.md
- Project documentation templates
- ADR template
- Decision Log
- Definition of Done
- Git Workflow
- Prompt Rules

### Changed
- Switched to project-local `.claude/agents`
- Standardized documentation under `docs/`
- Introduced Required / Optional document loading
- Added document priority rules
- Added Authority model
- Added Decision logging
- Added Git diff–based review workflow

### Notes
Project-local agents take precedence over user-global agents (`~/.claude/agents`) following Claude Code's documented behavior.
