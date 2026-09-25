# Complete

## Intent

Formally mark a work item as complete by updating its status. This represents the conclusion of all creative and technical work (planning, implementation, review, documentation) and signals that the work item is ready for archival.

Completion is the logical end of the active workflow. It ensures that the state of the work item is explicitly recorded within the artifact itself.

## When to Complete

- All implementation and documentation work for the work item has been verified complete.
- A Reflect action (if executed) has been finalized.
- The work item meets all acceptance criteria defined in `workitem.md` and `plan.md`.

Complete is NOT appropriate when:
- Unresolved blocking findings remain from a Code Review or Plan Review.
- `voice.md` is absent.
- The work item claims to have fixed an intermittent failure and the claim does not state the run count it rests on against the failure's prior rate, and whether the attribution rests on those counts or on the mechanism — a single green run is not that sample (`skills/evidence.md`, *State the Sample a Claim Rests On*; WI 1491 was credited from one green gate and corrected after WI 1494 found two more paths).

## Procedure

### 1. Verify Artifact Presence

Check the work item folder for required artifacts:

- [ ] `code.md` — must be present if the work item involved implementation.
- [ ] `reflect.md` — should be present for non-trivial work items.
- [ ] `voice.md` — must be present; one line, `Nothing to trim.`, when the work item added nothing in scope.

A single-artifact procedure's document (`sidequest.md`, `spike.md`, `adopt.md`) stands in for all three, here and under *When to Complete*.

### 2. Update Status

Update the `Status` field in `workitem.md` to `complete`.

If a `Completed Date` or similar metadata field is used in the project, update that as well.

### 3. Update Project

Update the work item status in the project doc (if a project is specified).

If this completion **closes a design line or milestone** (its last work item is now complete),
move that line's completed rows and version-history prose from the project's anchor document
into the project's history log **in the same edit**, leaving the anchor a one-line summary +
history link. Anchors carry open work; history carries done work — offloading at the closure
boundary keeps the anchor from silently re-accumulating (the failure mode is rows accreting
until a project assessment forces a cleanup).

If the work item declares a `phase` and this completion leaves that phase with no open member,
say so in the completion report and point at the Close step of `Phase.md`. **Do not close the
phase from here**: a phase may carry exit criteria beyond membership, and closing it is its own
gated step.
