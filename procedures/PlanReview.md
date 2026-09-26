# Plan Review

## Intent

Critically assess a plan document to ensure it is sufficient to guide correct implementation on first attempt. The output is a structured set of observations — concerns, questions, and recommendations — that identifies gaps, contradictions, or ambiguities before implementation begins.

Plan Review is preferably external; without a reviewer it is self-applied (`AGENTS.md`, *A review is never skipped for want of a reviewer*). See **Self-Applied Review** for how the two-party protocol collapses to one agent.

## Roles

**Reviewer:**
- **Prerequisites**: Access to the originating `workitem.md` (for Scope Alignment), access to applicable guideline documents identified during evaluation, and knowledge of any project-specific constraints stated in the plan's Dependencies & Context or Explicit AI Freedom sections.
- Evaluates the plan across five dimensions (step 3), resolves what can be resolved and stops on what cannot (step 4), and reports tiered findings and a Status (steps 5 to 7).

**Author:**
- Provides the plan and any context not captured within it (stakeholder constraints, technical limitations, etc.).
- Clarifies uncertain findings the Reviewer cannot resolve from the plan alone.
- Addresses findings and revises the plan until it reaches a Complete state.

## Self-Applied Review

Under self-application (`AGENTS.md`) the dimensions, the findings tiers, and the `planreview.md` artifact are identical to an external review; only the role assignments collapse:

- "Stops and notifies the requester" means surfacing to the human owner — the requester is the owner in both modes.
- The Revision Cycle Protocol applies unchanged: each self-revision is a real edit to `plan.md` addressing all Blocking findings, and three failed cycles still escalate to the owner as Significant Findings.
- Step 4's stop clause — intent only the Author can clarify — collapses the same way: the self-reviewer resolves the finding as step 5 says — revising `plan.md` so the uncertainty no longer exists — or holds it for the owner; it does not resolve it in the plan's favour.
- Step 6's Author-availability clause (confirm or dismiss uncertain findings from domain knowledge) is external-review only: a self-reviewer already holds all of the author's context, so uncertainty that survives it is genuine.
- The Precision spot-check SHALL be performed and its sample recorded in the artifact — which claims were checked against which opened files. It is the most bias-resistant step in the review because it is factual rather than judgmental.

## Procedure

### 1. Initiate Review

The Author provides the plan (typically `plan.md` in the work item folder).

If `plan.md` or the originating `workitem.md` cannot be located, stop, inform the requester, and record Status **Deferred** (step 7).

### 2. Structural Analysis

The Reviewer reads the plan in its entirety to identify:
- Presence and substance of each section (not just placeholders).
- Category of content: metadata, constraints, requirements, details, or meta-information.

Produce a **plan structure summary** confirming which sections are present and substantively filled.

A section contains **substantive content** when it provides information sufficient for an independent agent to evaluate whether the plan meets its stated objective without guessing missing details. Concrete indicators include:
- Specific conditions, constraints, or outcomes described in natural language.
- Named failure modes with specified responses (not just "handle errors").
- Scenario sequences that define inputs and expected outputs.

A section contains **placeholder content** when it consists only of headings with no body text, generic markers ("TBD", "[details to follow]"), single vague words ("appropriate", "sufficient"), or category names without elaboration ("see guidelines" with no specific guidance cited).

### 3. Evaluation

The Reviewer assesses the plan across these dimensions:

**Completeness** — are all sections that `Plan.md`'s **Plan Structure** requires present (open it)? Are there gaps in failure modes, security-relevant behaviors, or edge cases?

**Coherence** — are there internal contradictions? Check invariants against each other and against required behaviors. Ensure invariants are provable.

**Precision** — are terms specific and unambiguous? Flag vague language like "appropriate" or "as needed" in critical sections (Invariants, Behaviors, Edge Cases, Data Changes).

Precision is accuracy as well as clarity. Where the plan makes a concrete claim about existing code, content, or data, spot-check a sample of those claims against the artifact each one cites or describes — `Plan.md` requires the planner to have opened it (see its **Open-and-verify** rule), and this is the gate that checks they did. A claim that is precise but false is Blocking: unlike vagueness, which an implementer will notice and question, a confident wrong claim propagates silently into the implementation. Where a claim states a count or other quantity about an existing artifact, count it and record the number in the sample: confirming that the items match does not confirm how many there are (WI 1210: a plan's "six proven meals" passed its spot-check against a list of five).

For each condition or trigger that starts, keeps, repeats, or stops a behavior ("while X holds", "until Y", "at first start"), name one event that can change it or set it off. One that no event can change or set off is empty — Blocking, as ambiguity in a critical section — and the behavior it governs is not what the plan says (WI 1828).

**Scope Alignment** — does the plan stay within the mandate of the originating `workitem.md`? Distinguish reasonable deductions from purely opportunistic additions. A plan step that installs a binary or registers external tooling, in a work item that is not about installation, is scope inflation: name it as a step for the owner instead.

**Compliance** — does the plan conflict with any applicable `skills/[language].md` rules? Plans may override applicable guideline rules when explicitly documented in the plan itself. The Reviewer checks for two things: (a) whether a conflict exists, and (b) whether the plan explicitly acknowledges and justifies the override. Unjustified conflicts remain Blocking; explicit overrides with justification are Non-blocking structural observations unless they violate an Invariant.

### 4. Resolve Findings

Before reporting, the Reviewer SHALL attempt to resolve every finding identified in Step 3:

- **Fix it directly** — for Blocking and Non-blocking findings where the correct resolution is unambiguous (e.g., a typo, a trivially-wrong reference, a missing section whose content can be inferred from context), apply the fix to the plan document immediately. A finding is fixed when the file's diff shows the edit landed; an edit that did not land leaves it open, and the file is re-opened before a retry (WI 1657).
- **Stop on unresolvable findings** — if a finding cannot be resolved without information or decisions the Reviewer does not have (e.g., conflicting requirements, ambiguous intent that only the Author can clarify, scope questions that require stakeholder input), stop immediately and state:
  1. Which finding cannot be resolved.
  2. Why it cannot be resolved (what information or decision is missing).
  3. Who or what is needed to unblock it.
  4. The resolution you recommend, and why.

Do not proceed to Step 5 (Reporting) until all resolvable findings have been resolved or it is confirmed that unresolvable findings prevent proceeding.

### 5. Reporting

The Reviewer organizes findings into two tiers:
- **Blocking** — must be resolved before proceeding: a missing required section or placeholder-only content; an internal contradiction, such as conflicting invariants; a behavior that violates a stated invariant; scope inflation beyond the originating work item; ambiguity in a critical section (Behaviors, Invariants, Edge Cases) that could lead to incorrect implementation.
- **Non-blocking** — worth addressing but planning may continue: vague terms in non-critical sections (metadata, meta-information); partially-filled sections with minor gaps; an overly broad AI Freedom section that conflicts with no specific constraint; structural observations.

Flag findings as **uncertain** when they depend on intent or context the Reviewer cannot fully determine. An uncertain finding defaults to **Blocking** unless a human adjudicates it. Under a self-applied review, the author's presence in the session does not constitute adjudication: a Blocking uncertain finding is resolved the way any Blocking finding is — revise the plan so the uncertainty no longer exists, or hold it for the owner. It SHALL NOT be resolved in the plan's favor by default. The `planreview.md` output SHALL list all uncertain findings separately with their disposition so they are visible.

### 6. Judgment and Refinement

**Revision Cycle Protocol:**
- The Author may revise and resubmit up to **three times**.
- Each resubmission SHALL address all Blocking findings and note how Non-blocking findings were handled (addressed, deferred with rationale, or acknowledged).
- If three revisions have occurred and the plan still has unresolved Blocking findings, Status defaults to **Significant Findings** and the matter is escalated for human decision on whether to continue planning, reduce scope, or close the work item.

If the review is external and the Author is available to address uncertain findings (inoperative under self-application — see Self-Applied Review):
- Confirm or dismiss uncertain findings based on domain knowledge and project context.
- Identify anything the Reviewer missed (e.g., organizational nuances).

### 7. Status

Status is one of:

- **Complete** — no unresolved Blocking findings, and at most five Non-blocking findings with no pattern of systemic issues.
- **Significant Findings** — a Blocking finding remains, or the Non-blocking findings number more than five or affect more than half the plan's sections: systemic quality concerns that warrant revision.
- **Deferred** — a prerequisite artifact (`plan.md`, the originating `workitem.md`) is absent or cannot be located; a precondition failure, not an evaluation result, reported with which artifact is missing.

The Reviewer produces `planreview.md` in the work item folder, populated according to the artifact structure defined in this procedure. The file includes: **Status**, the **Review Mode** (`external` or `self-applied`), a **Structure Summary** of which plan sections are present and substantive, **Findings** organized by dimension (Completeness, Coherence, Precision, Scope, Compliance) with each tagged as Blocking or Non-blocking — under self-application the Precision findings include the recorded spot-check sample — an **Uncertain Findings** subsection listing all uncertain findings with their disposition, and **Free-form Feedback** for observations that don't fit structured categories.

## Notes

- **Evaluate Plan Quality, Not Merely Idea Merit**: Evaluate the planning artifact's quality, not just the technical merit of the idea.
- **Ambiguity as a Blocker**: If a term could lead to different implementation choices, it needs clarification. Ambiguity often masks hidden contradictions.
- **Match Depth to Complexity**: A short review for a small, well-scoped plan is appropriate.
- **Be Specific**: Reference section names and line numbers. "Possible race condition" is less helpful than naming the conflicting invariant and behavior.
- **Release When Ready**: If the plan can guide correct implementation, release it for the next step without delay.
