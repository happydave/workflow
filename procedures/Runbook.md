# Runbook

## Intent

Record an operational procedure a project repeats across work items — deploying a branch to a
test environment, driving a live end-to-end test, rolling back, rebuilding a fixture — as one
executable, dated document owned by the project. A work item's `test.md` records what one run saw;
the runbook records how to run it, and is corrected by every run.

Without one, each live verification re-derives the procedure from the last work item that happened
to run it, and the parts that drifted since (a renamed service, a moved node, a tool flag that stopped
working) are rediscovered as apparent defects.

## Three Layers

Keep these apart; mixing them is what makes procedures unfindable:

| Layer | Lives in | Example |
|---|---|---|
| Environment fact | the knowledge store (`knowledge/` for generic facts, the site overlay's `knowledge/` for site facts, or the platform store a project names) | which node pins ACKs, how a VPN assigns addresses |
| Project procedure | `docs/projects/<slug>/runbooks/<name>.md` | how *this* project deploys and drives its live test |
| Per-run evidence | the running work item's `test.md` | what run N observed, with timestamps |

A runbook links to environment facts and is linked from evidence. It copies neither.

## Document Storage & Naming

`docs/projects/<slug>/runbooks/<name>.md`, one file per operation, `lowercase-kebab-case`. Helper
scripts the runbook invokes sit beside it in `runbooks/<name>/`. The project record lists its
runbooks under an optional **Runbooks** heading (see `Project.md`).

## Required Content

- **Purpose** — the operation, in one sentence, and what it is *not* for.
- **Preconditions** — one line each, **every one paired with the command that checks it**. A
  precondition without a check is an assumption; the image-drift lesson (a shared namespace
  redeployed under a test) is what happens to assumptions.
- **Steps** — copy-pasteable, one operation per step, in execution order. A step that varies by run
  names the variable (`<TAG>`, `<NODE_IP>`) and the step that produces it.
- **Verification** — what a successful run looks like and *where to read it* (a log line, a document
  field, a command's output), not "confirm it works".
- **Known traps** — one line each, **dated**, stating the tell and the fix. A trap is a step that
  once read as a defect.
- **Last executed** — date, revision or image under test, outcome, and a link to the `test.md`
  that holds the evidence. A runbook not executed for a long time is read as suspect, and this line
  is what makes that visible.

## Procedure

### 1. Create

Write the runbook when one of these happens, not before — a runbook written ahead of a real run is
invented guidance:

- A `Test.md` execution is about to record a repeatable environment/driving procedure inside
  `test.md`. Write the runbook instead; `test.md` records the run and points at it.
- A `Discover`, `Design` or `Reflect` step names an operation the project will repeat.

Every command in a new runbook was run in the execution that wrote it.

### 2. Execute

Run the steps in order. Record in the work item's `test.md`: the runbook name and revision, every
deviation, and every new trap. Then update the runbook **in place** — correct the step that drifted,
add the dated trap, refresh **Last executed**. Never append narrative ("on 9/11 we found…"); the trap
line carries the date, the `test.md` carries the story.

### 3. Retire

When the operation no longer exists (the service is gone, the environment is decommissioned),
delete the runbook and note it in the project's Decisions. Do not leave a runbook that cannot be run.

## Guidance

- Directive, not narrative — `skills/comments.md` and `skills/authoring-skills.md` apply to the prose.
- Transitional steps (a cutover that leaves an old deployment scaled to zero) name the condition that
  removes them.
- A runbook that needs a credential names the tool that holds it (`skills/tooling.md`); it never
  contains one.
- Aim under 150 lines. Split by operation before growing past it.
