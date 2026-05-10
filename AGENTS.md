# Global Agent Rules

This file governs how the agent operates in this workspace.
**Superpowers** is the primary workflow framework. Skills activate automatically — do not force them unless specified.

## Instruction Priority

1. Explicit user instructions in the current session
2. Repository-specific rules, documentation, and conventions
3. This `AGENTS.md`
4. Superpowers skill definitions

---

## Coding Standards

Follow all rules in `.github/copilot-instructions.md` for all Java files.
This covers: async patterns, naming conventions, REST design, error handling, Repository pattern, SQLTemplate usage, WebClient rules, and security.

---

## Project Context

> **Fill in this section for your project.**
> Example:
> - **Platform**: DHP (Digital Health Platform) — internal microservice hosting platform
> - **Stack**: Java, Vert.x, SQLTemplate, JdbcPool, WebClient
> - **Service name constant**: `SERVICE_NAME = "your-service-name"` (used in `x-efx-component` header)
> - **Key modules**: e.g. `OrderVerticle`, `PaymentVerticle`
> - **Key concepts**: e.g. what an Order lifecycle looks like in your domain

---

## Default Principles

### Shortest Path First

- Default to the shortest path that meets quality requirements.
- Assess parallelism first — prefer parallel execution when tasks are clearly independent; otherwise proceed sequentially.
- If a task can be completed and verified directly, do not escalate to a heavier process.
- Small tasks use lightweight planning. Large tasks use the full Superpowers flow.

### Task Classification

| Type | Description | Default Approach |
|---|---|---|
| **Read-only** | Analysis, explanation, code reading, architecture questions | Answer directly |
| **Lightweight** | Single-file change, small bug fix, config or copy update, minor test addition | Skip full brainstorming/planning; implement and verify directly |
| **Medium** | New endpoint, service-level change, multi-file refactor | Short brainstorming + concise plan |
| **Large** | New feature, cross-module change, schema change, shared contract update | Full `brainstorming → writing-plans → implementation` |

### Lightweight Task Rules

- Skip full `brainstorming`, `writing-plans`, `using-git-worktrees`, and heavy review chains.
- Ask **at most 1 clarifying question** on the first lightweight task; do not re-ask if context is already available.
- If no response is received and risk is low, state your assumption and continue.
- Do not generate standalone spec/plan files for lightweight tasks unless explicitly requested.

### Process Escalation / De-escalation

**Escalate** when: scope expands beyond initial assessment, public API/schema/persistence/shared logic is affected, requirements remain unclear, or the task has evolved into a medium or large implementation.

**De-escalate** when: change is local with clear boundaries, shared core logic is not affected, verification is straightforward, or the problem has converged to a single fix.

---

## Superpowers Workflow Integration

Superpowers skills activate automatically. The standard flow for implementation tasks:

```
brainstorming → writing-plans → implementation → verification-before-completion
```

Skill reference by task type:

- **New feature / behaviour change**: `brainstorming → writing-plans → implementation`
- **Debugging**: `systematic-debugging` — confirm root cause before fixing
- **Code review**: `requesting-code-review` / `receiving-code-review`
- **Pre-completion**: `verification-before-completion`
- **High-risk behaviour change**: `test-driven-development`
- **Session wrap-up**: `session-wrap`

> Superpowers skills override default agent behaviour, but this `AGENTS.md` always takes precedence over skills.
> Example: if a skill says "always use TDD" but this file or the user says otherwise, follow this file.

---

## Execution & Verification

### Step-by-Step Reasoning

- When requirements are ambiguous, clarify: goal, constraints, acceptance criteria, and edge cases first.
- For multi-step tasks, maintain a visible task list; only one task is `in_progress` at any time.
- Lead with conclusions, then provide supporting context and trade-offs.
- Revise prior judgements proactively when new information emerges.

### Command Verification Rules

- Never fabricate executed commands, exit codes, or verification results.
- If a critical verification cannot be run, state the reason explicitly.
- Do not claim "done", "passing", "ready to commit", or "ready to merge" without verification evidence.

### Change Delivery Gate

Before declaring completion, committing, pushing, or opening a PR, ensure:

1. Verification directly related to this change has been completed and results reported honestly
2. Applicable quality gates have been passed
3. If the repository requires heavier verification, follow repository rules
4. If critical verification cannot be run, state this explicitly and downgrade completion confidence

### Commit Convention

Format: `<type>(scope): <summary>`

- `scope` is optional
- `summary`: imperative verb, ≤ 50 characters, no trailing period
- Common types: `feat` / `fix` / `refactor` / `docs` / `test` / `chore`

Examples:
```
feat(order): add create order endpoint
fix(payment): handle null response from DHP service
refactor(repo): migrate to SQLTemplate for all queries
```

### Testing Strategy & Quality Gates

TDD is not enforced by default on all tasks. Apply based on: behaviour impact, shared scope, regression risk, and test value.

| Level | When to apply |
|---|---|
| **L0 — Targeted verification** | Local, low-risk, small change |
| **L1 — Regression test** | Small fix or localised behaviour change |
| **L2 — TDD** | New feature, explicit behaviour change, shared logic, high-risk change |
| **L3 — Code review** | Follow `requesting-code-review` / `receiving-code-review` |
| **L4 — Completion verification** | Follow `verification-before-completion` and Change Delivery Gate above |

---

## Engineering Practices

### Getting Started on a Task

1. Read relevant files, documentation, and recent commits — understand module boundaries first
2. If the user provides `plan2go=<path>`, treat that file as the execution source and keep it in sync
3. To understand architecture, call chains, data flow, entry points, and dependencies:
   - Prefer `mcp__ace-tool__search_context` for semantic understanding
   - Use `rg` / `grep` only for exact string location of known identifiers
   - Architecture conclusions are based on `ace-tool`; `rg` for confirmation only

### Documentation Maintenance

- When plans, goals, constraints, key decisions, lessons learned, or progress change — update the relevant documentation.
- Proven lessons worth reusing should be added to this `AGENTS.md`.
- Lesson entry minimum template:

```
**Title**: [short label]
**Trigger**: [when this applies]
**Root cause / constraint**: [why this matters]
**Correct approach**: [what to do]
**Verification**: [how to confirm]
**Scope**: [where this applies]
```

### Code Quality Hard Limits

- Function length: ≤ 50 lines
- File length: ≤ 300 lines
- Nesting depth: ≤ 3
- Positional parameters: ≤ 3
- Cyclomatic complexity: ≤ 10
- No magic numbers — use named constants

### Bug / Test / Refactor Rules

- Bug reports must include: symptom, trigger condition, expected vs actual behaviour, affected scope, severity, and logs/stack/environment.
- Real bugs default to `systematic-debugging` — confirm root cause before fixing.
- Tests should prioritise: critical paths, boundary conditions, and error paths. Assert `expected` before `actual`.
- Follow SOLID, DRY, separation of concerns, YAGNI. Name things clearly. Handle edge cases explicitly.
- Refactoring: preserve behaviour first, improve structure second. Add tests before refactoring where necessary. For larger refactors, plan first, then review and verify on completion.

---

## Safety Rules

- Do not run destructive commands (e.g. `git reset --hard`) unless explicitly instructed by the user.
- Do not manipulate `.git` with non-Git tools.
- Avoid dangerous delete commands unless scope is explicitly limited to temporary artefacts.
- Never hardcode secrets, credentials, or API keys into source code.
- All database access must use parameterised queries — no string concatenation.
- Never interpolate untrusted input into shell commands or SQL.
- Do not terminate processes not started by the current task unless explicitly instructed.

---

## Multi-Agent & Parallel Execution

> This section applies when using Claude Code or Codex with subagent support.
> GitHub Copilot does not support subagent dispatch — skip this section when working in Copilot.

### Parallel Admission Criteria

Only use parallel execution when tasks can be naturally split into 2–4 subtasks that:
- Have clearly defined and non-overlapping `scope_write`
- Can be independently verified
- Have no obvious same-file write conflicts

Do **not** parallelize when: changes are concentrated in 1–2 core files, shared contracts/types/schema are involved, root cause is unclear, or splitting significantly increases integration risk.

### Files That Must Be Handled Serially

`package.json`, lockfiles, root build/lint/test config, CI config, schema/migrations, shared contracts/types, route/app entry points, environment variable templates, shared adapters.

### Subagent Rules

- Every `spawn_agent` call must explicitly set `model` and `reasoning_effort`.
- A subagent must stop and escalate if it needs to modify files outside its `scope_write`, if a shared contract needs to change, or if a conflict is discovered.
- After all subtasks complete, always run a consolidation step: summarise changes, check for conflicts, analyse merge order, run final verification (test / lint / build / smoke), and produce a final merge plan.
- Never assume "subtask complete = project complete".

---

## Communication Style

- Respond in **English** by default. Technical terms remain in English.
- Code identifiers and commit messages in English.
- Code comments in English, concise and clear.

### Output Format by Task Type

**Execution tasks** (code changes, bug fixes, multi-step tasks):

```
🎯 Task: one-line description

📋 Plan:
  ✅ Done
  🔄 In progress
  ⏸ Pending

🛠️ Current progress: what is being done, what has been completed

⚠️ Risks / blockers: potential issues or blocking factors

📎 Reference: file:line
```

**Analysis tasks** (Q&A, code explanation, architecture, diagnosis):

```
✅ Conclusion: 1–2 sentence direct answer

🧠 Key analysis:
  1. Core point
  2. Evidence
  3. Trade-offs

🔍 Deep dive: (optional)
📊 Option comparison: (optional)
🛠️ Implementation suggestion: (optional)
⚠️ Risks and trade-offs: (optional)
```

### Technical Content Rules

- Use fenced Markdown code blocks with language identifiers for all multi-line code, config, and logs.
- Examples should focus on core logic — omit irrelevant boilerplate.
- Use `+ / -` diff notation when highlighting changes.
- Use tables only when they genuinely add clarity.
- For complex responses, end with a short summary and a concrete next step or action item.
