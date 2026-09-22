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

Run the identified test suites and perform manual verification. If any steps require human intervention (e.g., UI verification, hardware interaction), the AI agent must explicitly ask the user to perform these steps and report the results.

When the verification is a live or environment-driven procedure, execute the project's runbook for
it (`Runbook.md`) and record in `test.md` the runbook used, every deviation, and every new trap; then
correct the runbook in place and refresh its **Last executed** line. If no runbook exists and the
procedure is one the project will repeat, write it now and have `test.md` point at it — `test.md`
holds this run's evidence, not the recipe.

### 3. Run the Negative Controls

A test that passes against a deliberately broken implementation is not evidence. For each behavior
the plan specifies in its **Required Behaviors & Verifications**, remove or invert that behavior and
confirm the intended tests fail on the cases that target it. The scope is the plan's behaviors: not
every new test, and not every line of the change. Before breaking anything, read the first column of
the table below: a matching row adds to the obligations here, and a behavior that matches no row
owes only these. Worked cases are in `knowledge/negative-controls.md`.

Plan the set of controls:

- **Name where each observable can be seen**: the fixture feature that would differ if the behavior
  were violated — a channel, a field, a state — and the moment the assertion samples it. An
  observable the fixture cannot carry is untested however many controls fail, because a control
  trips only assertions the fixture lets differ. An effect that ends before the assertion samples —
  a goroutine that starts and returns, a flag a later step clears, input that stops before the
  broken state is reached — is asserted through the decision that produces it, with a test still
  reaching the call site, or the input is extended until the effect holds when the assertion
  samples. When a fixture changes, every assertion reading it is named again and its control re-run,
  not only the new ones.
- **One break per observable.** Read the observables from what the behavior claims rather than from
  the count of its verification bullets: split a claim wherever a plausible weaker implementation
  could satisfy one part of it and not the other.
- **Where another site checks the same rule**, break each site alone and then all of them together.
  A site an earlier refusal makes unreachable is one finding, not a split repeated for every
  observable whose path crosses it.
- **Two breaks are exempt from one-per-observable**: the combined break just described, and a break
  re-expressed because the first attempt tripped an assertion belonging to another behavior (the
  table's third row). A re-expression and the attempt it replaces are both recorded, each as its own
  row, naming the other.

Run them one at a time:

- **Restore the tree between controls** — a combined break is one control — and verify the
  restoration after the last one; a control left applied forges every result after it.

Read what came back:

- **Read which cases failed, not only that something did.** A break that fails more cases than
  expected, or fewer, has located a gap.
- **Where a behavior is reached by more than one entry point**, say which of them the suite covered
  it at, whether the break failed something or nothing.
- **Show the break is live** before reading a nil result as missing coverage: a temporary test that
  reads the mutated state, or its first effect, directly, deleted once it has answered, because a
  break that never executed reports the same nothing. Where the break is unobservable by
  construction — two checks of one rule that refuse identically — the combined break discharges
  this, and the unobservability is itself the finding.
- **A break that fails nothing is a finding for step 5**, with four readings to separate: the
  behavior has no test; the break ran but its effect ended before the assertion sampled, which the
  liveness probe tells apart from the last reading; another check already does the broken one's
  work; or nothing reaches what you broke. Name which one it was — or which ones, since a masked
  path is both unreached and covered by whatever masks it.
- **Read the multi-site split**: every single break passing while the combined one fails means the
  sites are redundant; a single break that fails on its own is the load-bearing one, and the sites
  that passed are either its redundant copies or unreachable, which the applied diff tells apart; a
  combined break that also fails nothing means the behavior is untested.

Record what you did:

- **Each control is a row in `test.md`**, in these columns: the break and where it was applied, the
  fixture feature and moment its assertion reads, the applied-state check, the tests it failed, the
  assertions that tripped, anything it failed or spared unexpectedly — including a failure at an
  assertion belonging to another behavior — the runs, and the reading where nothing failed, left
  blank where something did. One run completes a deterministic control, such as an error versus
  nil.
- **Where a control could not be run**, name the behavior and the reason, so an omitted control is
  never indistinguishable from one that passed. An observable the fixture cannot carry is recorded
  the same way, as untested, and is a finding for step 5.

| When the behavior or its control… | Then… | Source |
|---|---|---|
| guards a destructive, irreversible, or outward-facing operation | run the control against the predicate that decides, never by driving the operation — *A destructive operation decides in a pure predicate*, `AGENTS.md`. A control runs with the guard removed, so a control that reaches the operation performs it. Where the decision is not separable from the act, do not run the control: the non-separability is the finding, and the restructuring comes first. | WI 1378; the 2026-09-08 loss |
| could be passed by a wrong implementation by chance — a choice among waiters, scheduling, a random port, a race | one red run is a coin. Run the control repeatedly and record failed-of-total; the passes in that split, over the total, are the rate at which the test misses this defect, and the remedy is to restructure the control so a wrong implementation cannot pass it by luck — typically by extracting the decision and driving it directly, where one run then decides. | WI 1500 |
| fails at an earlier assertion, or before any assertion runs | that is not coverage of this behavior: the tripped assertion must be the one that targets it. The test has other teeth; this behavior may have none. Re-express the break to reach the assertion that targets this behavior — holding the earlier claim constant — and record both attempts. | WI 1486 |
| is broken by deleting a block | express the break so the artifact still builds — flip the condition (`if false && …`), assign the zero value, discard the result, return early — and delete only where the deletion leaves nothing unused. A deletion that strands a variable or an import fails the build, and a build failure reads as *the tests did not notice* while proving nothing about the suite. | WIs 1644, 1646, 1667–1669 |
| is broken by moving a derived value — a hash, a timestamp, an identifier the view under test does not carry | move the value the property is about, and confirm the test can tell the difference: a timestamp shifted by a second under a millisecond assertion, or a key changed so the record leaves the view entirely, reports *did not fail* about something else. | WIs 1648, 1670 |
| mutates a target that is not unique in the file, is applied by a replace helper or script, or runs against an artifact a stale build could answer for | verify the applied state — the diff, a fresh build of the mutated tree — before reading the result, and record that check in the break cell. The result is exactly what a broken control forges. | WIs 1401, 1403 |

This is `skills/evidence.md`'s "actively seek contradictory evidence" applied to the suite itself: a control that *should* fail is what distinguishes a test with teeth from a test that agrees with whatever it is given.

### 4. Produce `test.md`

Document the testing process and results in a `test.md` file within the work item folder.

#### Required Sections:
- **Test Summary**: High-level pass/fail status and summary of coverage. A claim that a fix ended an intermittent failure carries the run count it rests on against the prior rate (`skills/evidence.md`, *State the Sample a Claim Rests On*); `Complete.md` refuses the claim without it.
- **Automated Results**: Output or summary of test suite executions.
- **Manual Verification**: Description of manual steps taken and their outcomes.
- **Negative Controls**: a row per control in step 3's columns — or, where none were run, which behaviors went uncontrolled and why.
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
