# Test

## Intent

Formally verify the implementation against requirements, acceptance criteria, and quality standards. This procedure ensures that the changes work as intended, do not introduce regressions, and meet the technical requirements defined in the planning phase.

## When to Test

- Implementation is complete and documented in `code.md`.
- Code review has been finalized (or is being conducted in parallel with testing).
- The changes are ready for final verification before documentation and completion.

## Input & Guidance Priority

Test guidance may be found in multiple locations. When conflicting or overlapping guidance exists, follow this priority order:

1.  **Work Item Plan** (`plan.md`): Specific test cases, edge cases, and acceptance criteria defined for this work item.
2.  **Project README/Manifest**: Project-wide testing standards, required test suites, or environment configurations.
3.  **Language/Framework Guidelines**: General best practices for the specific technology stack (e.g., `Go.md`, `TypeScript.md`).

## Roles & Responsibilities

**Author** — executes automated tests, performs manual verification, and records results.

**Reviewer** — verifies that the test coverage is sufficient and that all findings have been addressed or triaged.

## Procedure

### 1. Identify Test Requirements

Gather testing instructions from the sources identified in the **Input & Guidance Priority** section. List all required:
- Unit tests
- Integration tests
- Manual verification steps
- Performance or security checks (if applicable)

### 2. Execute Tests

Cite each identified suite's result: Code's recorded run, with Code Review's runs over its own edits;
run a suite here only for an edit made since both, or where no earlier step ran it — an integration,
live, performance or manual check (`AGENTS.md`, *A test suite runs once for each tree it
certifies*). Perform the manual verification. If any steps require human intervention (e.g., UI verification, hardware interaction), the AI agent must explicitly ask the user to perform these steps and report the results.

When the verification is a live or environment-driven procedure, execute the project's runbook for
it (`Runbook.md`) and record in `test.md` the runbook used, every deviation, and every new trap; then
correct the runbook in place and refresh its **Last executed** line. If no runbook exists and the
procedure is one the project will repeat, write it now and have `test.md` point at it — `test.md`
holds this run's evidence, not the recipe.

### 3. Run the Negative Controls

A test that passes against a deliberately broken implementation is not evidence. A control removes
or inverts one behavior and confirms that a test fails at the assertion targeting it. `Code.md`
step 4 ran one for each clause as its test was written, so Test controls only what that pass cannot
have settled:

- **a guard on a destructive, irreversible, or outward-facing operation** — run against the
  predicate that decides, never by driving the operation with arguments the test did not create
  (*A destructive operation decides in a pure predicate*, `AGENTS.md`). Where the decision is not
  separable from the act, do not run the control: the non-separability is the finding, and the
  restructuring comes first;
- **each invariant** in the plan's **Invariants & Hard Constraints**;
- **a test added or changed since Code's break pass** — by review, by a fix in step 2, or by a
  fixture change, which re-opens every assertion that reads the fixture — and a clause `code.md`
  shows no break for;
- **a behavior that failed in the wild** — the defect this work item fixes, or one step 2 found.

A behavior outside this set is not controlled at Test and needs no row saying so.

**The full sweep** — a control for every behavior in the plan's **Required Behaviors &
Verifications** and **Edge Cases** — runs when the plan or the owner asks for it, on a
hardening pass (`RapidIteration.md`), and on a work item that touches code whose defects stay
silent until they are costly — consensus, replication, failover, durability. A sweep follows
`knowledge/negative-controls.md`: how the set is planned, the restore and cache rules between
controls, how a result is read, and the record it keeps.

For each control run here:

- commit step 2's fixes and the review's test changes first, then start each control from that
  commit and restore to it, with compiled-code caches off; when the last one is done, a diff
  against that commit shows nothing;
- express the break so the artifact still builds — flip the whole condition, assign the zero
  value, discard the result, return early (`skills/go.md` for Go) — and verify it applied before
  reading the result; a guard's control runs only the predicate's tests and the tests that hand
  the operation arguments they created;
- read which cases failed, not only that something did: the tripped assertion is the one that
  targets the behavior, or the behavior has no proven coverage;
- run a control a wrong implementation could pass by chance — a choice among waiters, a race, a
  random port — repeatedly, and the unbroken tree as often; record failed-of-total for both, and
  restructure the control so that one run decides;
- a break that fails nothing is shown live first — a temporary test reading the mutated state,
  deleted before the restore check — and is then a finding for step 5: no test holds the behavior,
  the effect ended before the assertion sampled, another check masks it, or nothing reaches it.
  Name which.

### 4. Produce `test.md`

Document the testing process and results in a `test.md` file within the work item folder.

#### Required Sections:
- **Test Summary**: High-level pass/fail status and summary of coverage. A claim that a fix ended an intermittent failure carries the run count it rests on against the prior rate (`skills/evidence.md`, *State the Sample a Claim Rests On*); `Complete.md` refuses the claim without it.
- **Automated Results**: Output or summary of test suite executions.
- **Manual Verification**: Description of manual steps taken and their outcomes.
- **Negative Controls**: one line per control — the behavior, the break and where it was applied, the applied-state check, the test and assertion that failed. A control that failed nothing, failed at another assertion, ran repeatedly, or could not be run carries the full record in `knowledge/negative-controls.md`'s columns, as does every control of a sweep. Say whether the full sweep ran.
- **Findings**: A clearly enumerated list of all problems found.

### 5. Handle Findings

All findings in `test.md` must be addressed:

- **Minor Issues**: Bugs, typos, or minor deviations should be corrected immediately. Re-run affected tests to verify the fix.
- **Significant Issues**: If a finding requires design changes, significant architectural changes, or falls outside the scope of the current work item/project design:
    - Do NOT implement the change immediately.
    - Document the issue clearly in `test.md`.
    - Suggest the creation of one or more new work items to address the issue.
    - Consult the user/reviewer to decide if the current work item can proceed to completion or if it is blocked.

## Guidance

- **Negative Testing**: Always include test cases for invalid input, error states, and boundary conditions. **An assertion of absence must first establish the presence it qualifies** — a test that is satisfied when the thing it guards never happens at all passes while proving nothing. Assert that the value reaches the output, then assert how it appears there. This matters most for escaping, injection, redaction, and secret-scanning checks, where a vacuous test reads as security coverage: raji-local-mcp's reflected-input test asserts the hostile value is echoed at all *before* asserting that it is echoed escaped, and says so in a comment. The timing case is the same rule — when a test asserts that a disabled or detached component receives *nothing*, precede the assertion with a wait for the last event that was expected to arrive, otherwise the assertion can pass simply because it ran before anything was delivered. The converse shape is a content scan, which is scoped to the artifact's code rather than its prose, so a check cannot trip on documentation of itself.
- **Evidence-Based**: Where possible, include logs, screenshots, or command output in `test.md`. Apply `skills/evidence.md`: report each result at the confidence the evidence supports, and remember that passing automated metrics does not verify a result until the artifact behind it is examined (a build that produces a file with the right shape can still produce the wrong file).
- **No Guesswork**: If it's unclear how to test a specific component, refer back to the `Discover` procedure or ask the user for clarification.
