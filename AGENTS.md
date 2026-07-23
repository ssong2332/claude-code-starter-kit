# AGENTS.md — Agent Contract

This document is the contract between the seven project agents. It defines what each agent consumes, produces, may modify, and which documents it owns.

## Prohibitions (override all other rules)
- No agent modifies a document or file it does not own (see Ownership table).
- planner, architect, ux-design, docs: never modify source code.
- reviewer, quality-assurance: never modify any file. Output is a report only.
- No agent invents requirements. Unclear requirements become Open Questions in the report.
- No agent waits for approval mid-run. Report items needing approval, then finish.

## Pipeline

```
planner → [user approval] → ux-design (if the project has a user-facing UI) → [user approval]
        → architect → [user approval] → implementer
        → reviewer → quality-assurance → (fixes: implementer again) → docs
```

- ux-design is skipped for projects with no user-facing UI (CLI, library, headless API) — ux-design itself reports "not applicable" in that case rather than the user having to know to skip it.
- reviewer runs before quality-assurance, never in parallel: the DoD Gate checklist that quality-assurance verifies includes "reviewer Status: APPROVED", so QA cannot gate until the review verdict exists.
- reviewer emits Status: APPROVED/REJECTED; quality-assurance emits Release Recommendation: GO/NO-GO. On REJECTED or NO-GO, their Action Items for Implementer go back to implementer, who re-implements and resubmits — implementer does not need to re-read the full report, just the Action Items.
- Design-level defects go back to architect first, then implementer. UX-level defects go back to ux-design first.
- implementer must satisfy the docs/DefinitionOfDone.md Gate checklist before handing off to reviewer/quality-assurance.
- Status `done` is set by implementer as soon as quality-assurance returns GO (all DoD Gate items passing). The docs step then completes the DoD Closure items (documentation sync, docs/CHANGELOG.md); this happens after `done` and does not require re-invoking implementer.
- docs never edits documents it doesn't own. When docs finds one stale (PRD/UX/Architecture/API/Database/DECISIONS), it appends a row to docs/UpdateRequests.md naming the owning agent, so the request survives even if no one acts on it in the same session; the user (or the pipeline) routes that request to planner/ux-design/architect as appropriate, and the owning agent marks the row resolved once handled.

## Authority

Each agent acts only within its authority. An agent must never perform work outside its authority unless explicitly instructed by the user.

| Agent | Authority |
|---|---|
| planner | Planning only |
| ux-design | UX/UI design only |
| architect | Technical design only |
| implementer | Code only |
| reviewer | Review only |
| quality-assurance | Validation only |
| docs | Documentation only |

## Agent Contract Table

| Agent | Input (consumes) | Output (produces) | May modify |
|---|---|---|---|
| planner | User request, docs/DECISIONS.md | docs/PRD.md, docs/Tasks.md, Open Questions report | docs/PRD.md, docs/Tasks.md only |
| ux-design | docs/PRD.md | docs/UX.md (if the project has a UI; includes a Claude Design Prompts section for generating UI mockups externally), design report | docs/UX.md and docs/UX-archive.md (append-only) only |
| architect | docs/PRD.md, docs/UX.md (if present) | docs/Architecture.md, docs/API.md (if required), docs/Database.md (if required), docs/DECISIONS.md entries, docs/adr/ records, design report | docs/Architecture.md, docs/API.md, docs/Database.md, docs/DECISIONS.md, docs/adr/ only |
| implementer | docs/PRD.md, docs/Architecture.md, docs/CodingRules.md, docs/GitWorkflow.md, docs/Tasks.md, docs/API.md, docs/Database.md, docs/UX.md, docs/DefinitionOfDone.md | Source code, implementation report | Source code; the Status column of its own task row in docs/Tasks.md only (recommend other doc updates; never silently change them) |
| reviewer | git diff (preferred), files explicitly specified by the caller, project documentation, docs/GitWorkflow.md, docs/UX.md (if present) | Review report (Status: APPROVED / REJECTED) | Nothing |
| quality-assurance | git diff (preferred), files explicitly specified by the caller, project documentation, docs/DefinitionOfDone.md, docs/UX.md | Test report (Release Recommendation: GO / NO-GO) | Nothing |
| docs | Project changes (git diff), README.md, docs/CHANGELOG.md, other project documentation (read-only, to detect drift) | Updated README.md/docs/CHANGELOG.md, rows appended to docs/UpdateRequests.md for documents it doesn't own, documentation summary | README.md, docs/CHANGELOG.md, docs/UpdateRequests.md (append-only) |
| planner / ux-design / architect | (in addition to their existing inputs) docs/UpdateRequests.md rows naming them as Owning Agent | (in addition to existing outputs) resolved Status on the rows they acted on | (in addition to existing scope) docs/UpdateRequests.md Status column, rows they own only |

## Document Ownership

| Document | Owner (creates/updates) | Everyone else |
|---|---|---|
| CLAUDE.md | User | Read-only |
| AGENTS.md | User | Read-only |
| README.md | docs | Read-only |
| docs/PRD.md | planner | Read-only. docs reports drift as an Update Request; only planner edits |
| docs/Tasks.md | planner | Read-only, except implementer may update the Status column of its own task row |
| docs/UX.md | ux-design | Read-only. docs reports drift as an Update Request; only ux-design edits |
| docs/UX-archive.md | ux-design (append-only) | Read-only. Overflow archive for old Deprecated UX entries; archived IDs stay reserved |
| docs/Architecture.md | architect | Read-only. docs reports drift as an Update Request; only architect edits |
| docs/API.md | architect | Read-only. docs reports drift as an Update Request; only architect edits |
| docs/Database.md | architect | Read-only. docs reports drift as an Update Request; only architect edits |
| docs/DECISIONS.md | architect | Read-only. docs reports drift as an Update Request; only architect edits |
| docs/adr/ | architect | Read-only (ADRs are immutable once accepted) |
| docs/CodingRules.md | User (or architect on request) | Read-only |
| docs/GitWorkflow.md | User | Read-only (implementer follows) |
| docs/DefinitionOfDone.md | User | Read-only (implementer/QA enforce) |
| docs/PromptRules.md | User | Read-only |
| docs/CHANGELOG.md | docs | Read-only |
| docs/UpdateRequests.md | docs | Read-only, except the named Owning Agent (planner/ux-design/architect) may flip a row's Status from `open` to `resolved` |
| Source code | implementer | Read-only |
| .claude/agents/*.md | User | Read-only. An agent's own definition file is not something it edits at runtime |
| .claude/memory-protocol.md | User | Read-only |

## Document Priority

Each agent's own file defines which documents are Required (always read if available) vs. Optional (read only when relevant to the current task) — this keeps agents from burning context on documents outside their role. When documents an agent actually reads conflict, the higher-priority one below takes precedence:

1. CLAUDE.md
2. AGENTS.md
3. docs/PRD.md
4. docs/UX.md
5. docs/UX-archive.md
6. docs/Architecture.md
7. docs/DECISIONS.md
8. docs/adr/
9. docs/CodingRules.md
10. docs/GitWorkflow.md
11. docs/DefinitionOfDone.md
12. docs/Tasks.md
13. docs/API.md
14. docs/Database.md
15. docs/PromptRules.md
16. README.md
17. docs/CHANGELOG.md
18. docs/UpdateRequests.md

README.md and docs/CHANGELOG.md rank below the spec documents because they are derived from them (docs keeps them synchronized) — a derived document must never win a conflict against the spec it was derived from.

## Project Structure

```
project/
├── CLAUDE.md
├── AGENTS.md
├── README.md
└── docs/
    ├── PRD.md
    ├── UX.md
    ├── UX-archive.md
    ├── Architecture.md
    ├── Tasks.md
    ├── CodingRules.md
    ├── Database.md
    ├── API.md
    ├── CHANGELOG.md
    ├── DECISIONS.md
    ├── DefinitionOfDone.md
    ├── GitWorkflow.md
    ├── PromptRules.md
    ├── UpdateRequests.md
    └── adr/
        └── 0001-....md
```
