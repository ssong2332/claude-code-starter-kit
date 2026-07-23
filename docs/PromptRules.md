# Prompt Rules — {{project-name}}

Owner: User. How to invoke the seven agents. Contract details: AGENTS.md.

## Pipeline with Approval Gates
```
planner → [user reviews PRD + answers Open Questions]
→ ux-design (if the project has a user-facing UI) → [user approves flows/screens; optionally generates UI mockups in Claude Design using the prompts in docs/UX.md — mockup deviations go back to ux-design to sync the spec]
→ architect → [user approves design]
→ implementer (one Tasks.md ID at a time)
→ reviewer → quality-assurance (sequential — QA's gate includes reviewer's APPROVED)
→ (issues? → implementer again / design defect? → architect first / UX defect? → ux-design first)
→ [GO: implementer marks the task `done`] → [orchestrator/user merges the task branch into main — never a subagent, see docs/GitWorkflow.md]
→ docs
```

## Invocation Table
| When | Say | Agent invoked |
|---|---|---|
| Project start / new requirements | "planner 에이전트로 요구사항 정리해줘" | planner |
| PRD approved, project has a UI | "ux-design 에이전트로 UX 설계해줘" | ux-design |
| PRD (and UX, if applicable) approved, need design | "architect 에이전트로 아키텍처 설계해줘" | architect |
| Design approved, build task | "implementer로 Tasks.md의 T{{n}} 구현해줘" | implementer |
| After implementation | "reviewer로 방금 변경분 리뷰해줘" | reviewer |
| Before marking done | "quality-assurance로 T{{n}} 검증해줘" | quality-assurance |
| After merge / release | "docs 에이전트로 문서 동기화해줘" | docs |

## Always
- Explain assumptions before acting on ambiguous input.
- Cite modified files (path + line) in every report.
- Produce a suggested commit message after code changes.

## Never
- Guess requirements — list them as Open Questions instead.
- Modify files unrelated to the current task.
- Rewrite large files when a small diff suffices.

## Rules
- One implementer invocation = one task ID. Never batch tasks in one prompt.
- Always pass the reviewer/QA report verbatim when re-invoking implementer for fixes.
- The verbatim rule applies to ANY relay of one agent's findings to another, not just fix re-invocations — quote, never paraphrase. A paraphrase can silently turn a narrow note into a broader instruction, and the receiving agent may then attribute the change to the wrong source. Agents cite provenance only from text they can verify.
- Approval gates are the user's job — agents report and stop; they never self-approve.
- If an agent's report contains Open Questions, answer them before invoking the next agent.
