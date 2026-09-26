# Reflect

## Intent

After implementation completes, capture what happened so the process, documentation, and tooling improve over time. This is not a narrative — it is a structured record that feeds concrete changes back into the planning framework, feature templates, guidelines, and implementation prompts.

## When to Reflect

At the end of every implementation, or when implementation is interrupted and will be resumed in a new session.

## Document Storage

Create `reflect.md` in the work item's folder alongside `workitem.md`, `plan.md`, and `code.md` (e.g., `docs/pending/07-investigate-pipeline-jobs/reflect.md`).

## Procedure

### 1. Record What Went Well

Identify aspects of the implementation that worked effectively. These may include:

- Planning documents that were clear and sufficient
- Implementation order or sequencing that avoided rework
- Constraints that prevented mistakes
- Tooling or infrastructure that worked as expected
- Scope discipline that kept the implementation focused

The purpose is to reinforce what should be preserved or repeated.

### 2. Record What Could Have Gone Better

Identify friction, errors, and rework during implementation. For each item, note:

- What happened
- Why it happened (root cause if apparent)
- How it was resolved

If the work item ran a project runbook (`Runbook.md`), note whether it had drifted, and confirm the
correction landed in the runbook itself rather than only in `test.md`.

Focus on issues that originated from the process, documentation, or tooling — not incidental problems like a service being temporarily unavailable.

Record friction here on its first occurrence, even when it warrants no change: the reflection is where a later session finds that a problem has happened before.

### 3. Produce Recommendations

For each significant issue, propose a concrete change to prevent or reduce it in future implementations. Recommendations should target specific documents or artifacts:

- The plan (e.g., add a missing invariant to its Invariants section)
- Framework procedure documents (e.g., update `procedures/Plan.md` with additional plan sections)
- Framework guidelines (e.g., update `skills/docker.md`)
- Dispatch briefs (`Dispatch.md`)
- Project tooling (e.g., add a pre-flight check to the gate)
- New work items (if the issue warrants separate follow-up)

Avoid vague recommendations. "Improve documentation" is not actionable. "Add an invariant to feature 08 stating that `.vscode/tasks.json` commands must use Makefile targets" is.

A recommendation that changes the workflow repo — a procedure, a skill, a directive, or a new work item against it — is made for a problem experienced repeatedly: the same root cause, in any project, in this work item and in at least one earlier one, found by searching earlier reflections under `docs/pending/` and `docs/archive/` and named by work item ID; or once, when the occurrence caused a loss (data, a host, published content). Where an earlier reflection shows a rule already covers the problem, the recommendation goes to that rule or to its application, not beside it. A first occurrence stays in step 2, and a reflection that found no earlier one says what it searched for. Recommendations to the plan, the project's docs, or its tooling carry no such bar. Source: WI 1878.

### 4. Human Review

The human reviews the reflection and decides which recommendations to act on. Not every recommendation warrants a change — some may be one-off issues not worth codifying.

## Guidance

- Keep entries concise and factual.
- Separate problems caused by documentation gaps from problems caused by implementation errors. Both are worth recording, but they have different fixes.
- If the implementation was interrupted and resumed, note where the break occurred and whether the implementation log was sufficient to continue.
- The reflection is a companion to the implementation log — it exists to feed improvements back into the process, not to stand alone.
