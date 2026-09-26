# Code Review

## Intent

Verify that the changes produced by the Code procedure correctly implement the plan before proceeding to Test. The output is a structured set of findings — blocking issues that must be resolved, non-blocking observations worth noting, and confirmation that the implementation is ready to proceed.

Code Review is a cooperative process: the Agent performs exhaustive mechanical scanning and evaluates changes against the plan; the Reviewer exercises judgment on ambiguous findings and decides what must be resolved before Test begins.

## When to Use

- The Code procedure has completed and `code.md` is up to date.
- Before proceeding to the Test procedure.

## Roles

**Reviewer:**
- Clarifies findings the Agent flags as uncertain
- Judges which findings are blocking vs. acceptable given intent and context
- Decides whether to fix issues in-place or proceed to Test with known non-blocking observations

**Agent:**
- Reads the full diff without fatigue or anchoring bias
- Evaluates changes against the plan's requirements, invariants, and acceptance criteria
- Surfaces mechanical issues that are easy to miss at scale: misspellings, shadowed variables, name mismatches, copy-paste errors, inconsistent changes across similar files
- **Defaults to action**: if a finding has an obvious, good solution, applies the fix directly and notes it in the findings summary
- For findings without an obvious solution, weighs risk before deciding how to proceed (see step 4)
- Stops the pipeline and escalates to the Reviewer only for high-risk decisions where the cost of a wrong call exceeds the cost of pausing

## Procedure

### 1. Agent: Read Context

Read the following before reviewing the diff:

- `plan.md` — the authoritative specification: requirements, invariants, acceptance criteria, and applicable guidelines
- `workitem.md` — original intent and constraints
- `code.md` — what was done, decisions made, and any deviations from the plan

If `code.md` is missing or contains only empty scaffolding, this is a **blocking finding** — report it and do not proceed until it is provided.

Follow any document references in plan.md or workitem.md that are relevant to evaluating correctness (e.g., design docs, linked specifications).

### 2. Agent: Review the Diff

Obtain the local diff (e.g., `git diff` for unstaged, `git diff --cached` for staged, or the set of files modified during the Code procedure).

Read all changed files in full. Do not skim. For each file, identify:

- What category of change this is: substantive (logic, behavior, structure) or mechanical (formatting, whitespace, comment wording, generated code)
- Whether the change corresponds to a requirement or decision recorded in `plan.md` or `code.md`

Produce a brief **change summary** grouping files by category. Flag any changes that are not traceable to the plan or `code.md` — these may be unintended or out-of-scope.

A diff too large to read well in one context may be divided by independent area among helper sessions, each given the plan's requirements for its area. The review stays the Agent's: read directly the paths the plan's invariants and the Safety dimension run through, run the searches for callers and readers that cross areas, verify every delegated finding in the code before acting on it, and record in `codereview.md` which areas were delegated and to what brief. A delegated area that reports nothing is recorded as read by a helper, not as clean (WI 1756).

### 3. Agent: Evaluate

Assess all substantive changes across these dimensions:

**Correctness** — does the implementation match what the plan specifies? Check requirements, invariants, acceptance criteria, and each item of the plan's Edge Cases explicitly — an edge case that specifies behavior is a requirement under a quieter heading. Where the change adds a branch, an early return, or a state a value can newly take, find every path that reaches it and every reader of what it leaves behind — unchanged code included, found by search rather than from the plan's list — and check each: a new branch reached by a caller nobody enumerated is the regression review exists to catch (WI 1563). Note any gaps between what was planned and what was built. Per `skills/evidence.md`, a passing metric is not proof a result is correct. Wherever a criterion's evidence is a count, a summary or a test result standing in for a behavior — whatever the criterion's tag — examine what it stands for: the artifact, the file's contents, the items that were posted and in what order. The examination is discharged by an assertion that fails when that content is wrong; where no assertion can be written, by an inspection recorded in `codereview.md` naming what was looked at and what it showed. An `[agent]`-tagged criterion (see `Plan.md`) is always verified this way. A test that checks only a count where the behavior is about content — three posts where the plan says every item once, in order — is a Tests finding (WI 1674).

**Safety** — does the change introduce risk? Consider: security vulnerabilities, data loss scenarios, race conditions, inconsistent state, irreversible side effects. Where an operation is destructive, irreversible, or outward-facing, check that the decision to proceed is separable from the act — a pure predicate the operation calls, per the directive in `AGENTS.md` — and that everything the predicate decides on is computed inside it or passed in by a caller that decides nothing. A condition worked out in the caller is part of the decision made outside the predicate, and a control on it drives the operation (WI 1827).

**Clarity** — is the code readable and maintainable? Naming, structure, and whether a future reader would understand the intent without needing to ask the author. A comment's voice — length, phrasing, density — is `Voice.md`'s step, not review's; its truth is review's. A comment asserting a behavior the code does not have, such as a guarantee of all-or-nothing on a batch that can publish part of itself, or a doc comment an insertion has left describing a different declaration, is a Correctness finding (WIs 1595, 1674).

**Tests** — are new behaviors covered? Are existing tests still meaningful? Absence of tests for plan-specified behaviors is a blocking finding. An assertion of absence must establish the presence it qualifies, and a guard on a destructive, irreversible, or outward-facing operation is tested at the predicate that decides rather than by driving the operation — see `Test.md`, *Negative Testing* and *Run the Negative Controls*. Where the product code checks the same thing at more than one site, name the guarantee each site gives before touching anything, reading it from the plan's invariants and from where in the code the check sits, which the call graph alone does not show. A site whose guarantee is its own for some input that reaches it — a batch refused whole before its first item goes out, where the per-item check publishes some of them first — stays, and gets the case only its removal would break where none exists. A site whose guarantee a survivor gives at the same point, before any effect, on every input that reaches it — a check re-made in a caller whose callee refuses identically — may be deleted, with a test that reaches its path. A site whose guarantee cannot be named is left standing and recorded as a gap in the suite: the silence of a control settles nothing by itself, and is never a licence to delete. Source: WIs 1646–1651, 1668, 1670, 1671.

**Scope** — does the change stay within what the plan specifies? Unrelated or opportunistic changes should be flagged.

**Consistency** — for changes applied across multiple similar files, verify the pattern is applied uniformly. Inconsistencies across similar files are a common source of subtle bugs.

**Workflow repo** — when the diff changes the workflow repo, the gates in `WorkflowChange.md` (filter; leak gate, coherence, form, application test, fit) are part of this evaluation and are recorded in `codereview.md` under **Workflow change gates**.

Also scan exhaustively for mechanical issues regardless of change category:

- Misspellings in identifiers, comments, log messages, or documentation
- Shadowed variables or identifier reuse that may mask intent
- Variable or parameter name mismatches
- Copy-paste errors in repeated blocks

### 4. Agent: Report Findings

For each finding, apply one of three responses before reporting. Whichever response applies an edit, the finding is recorded as resolved only after the file's diff has been read: a script's output can show that an edit failed, never that it succeeded. An edit that did not land leaves the finding open, and a failed precondition is evidence about the text — re-open the file before retrying (WI 1657).

**Fix directly** — if there is an obvious, good solution: apply it, then include the finding in the summary as resolved. This is the default for linter-class issues, typos, name mismatches, and any substantive issue where the correct fix is unambiguous.

**Decide and proceed** — if there is no obvious solution but the risk is low: make a reasoned choice, apply it, document the rationale, and flag it in the summary for Reviewer awareness. Low risk means the decision is reversible via git and does not affect external contracts, security, or data integrity. A contract the plan itself specifies is the plan, not a change to it: implementing it, or choosing which of two colliding plan edge cases wins, is decided and recorded here (WI 1715).

**Stop and escalate** — if there is no obvious solution and the risk is high: halt and surface to the Reviewer before proceeding. High-risk decisions include: security vulnerabilities, data loss scenarios, breaking changes to external APIs or contracts, and scope changes that meaningfully deviate from the plan.

After the last edit this review makes, re-run the build and test steps of the plan's Applicable Guidelines. The disposition rests on that run: a red run is a finding, and a review that leaves the tree red does not proceed to Test.

Organize the findings summary into three tiers:

- **Escalations** — decisions stopped for Reviewer input, and any blocking finding the review cannot resolve itself; include what was found, why it is high-risk or blocking, what options exist, and which one is recommended and why
- **Resolved** — issues found and fixed, including both direct fixes and decided-and-proceeded cases; mark each finding that was blocking, and include the rationale for any judgment calls
- **Observations** — non-blocking notes the Reviewer may want to be aware of but that do not require action before Test

Write the findings summary to `codereview.md` in the work item folder (see Document Storage).

### 5. Reviewer: Act on Escalations

Review any escalated findings:

- Provide the missing context or make the call the Agent could not
- The Agent then applies the resolution and updates the findings summary

If there are no escalations, no Reviewer action is required — proceed to Test.

If escalations exist and no Reviewer is available, the work item **holds at the escalation**: record the escalation and a `held` disposition in `codereview.md`, notify the owner, and do not proceed to Test until the owner acts. Do not self-resolve an escalation. Escalations are defined as high-risk-only precisely so they are rare; a reviewer resolving its own escalations collapses the tier into "decide and proceed" and the distinction stops meaning anything.

## Document Storage

The findings summary from step 4 is written to `codereview.md` in the work item folder. It contains: the **change summary** from step 2, any delegation of the reading, the three findings tiers (**Escalations**, **Resolved**, **Observations**), the result of the final gate run, and a final **Disposition** — proceed to Test, or held at an escalation with the owner notified. The file's presence is what makes a completed review visible; other procedures rely on the name (`Spike.md` cites `codereview.md` as one of the pipeline artifacts its single spike document replaces).

## Guidance

- The plan is the primary evaluation standard. A change that works but doesn't match the plan is a finding; a plan gap discovered during review should be noted in `code.md`.
- A finding accepted from a reviewer is still a change: it is applied and then verified by the same gates as any other change, not exempted because it arrived with a reviewer's endorsement. Where the reviewer proposes a *fix* rather than reporting a defect, the fix carries the author's burden of proof — including measuring its cost when it touches a failure path.
- Default to action. Software can be rewritten and git provides a rollback path — the cost of a recoverable wrong decision is almost always lower than the cost of stopping unnecessarily.
- Escalate sparingly. Escalation is for decisions where recovery would be expensive or impossible: security holes, data loss, broken external contracts, significant unplanned scope. Everything else is low risk by default.
- Distinguish escalations from observations clearly. Treating every finding as a stop trains reviewers to ignore findings; burying real escalations in observations leads to proceeding with unresolved high-risk decisions.
- A short review of a small, well-scoped change is not a failure of rigor — it reflects a change that was ready. Match depth to complexity.
