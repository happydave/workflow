# Complete

## Intent

Formally mark a work item as complete by updating its status in `ticket.md`. This represents the conclusion of all creative and technical work (planning, implementation, review, documentation) and signals that the work item is ready for archival.

Completion is the logical end of the active workflow. It ensures that the state of the work item is explicitly recorded within the artifact itself.

## When to Complete

- All implementation, testing, and documentation work for the work item has been verified complete.
- A Reflect action (if executed) has been finalized.
- The work item meets all acceptance criteria defined in `ticket.md` and `plan.md`.

Complete is NOT appropriate when:
- Unresolved blocking findings remain from a Code Review or Plan Review.
- Implementation is incomplete or has not been verified against guidelines.
- The Author or Reviewer believes further changes are needed to meet the objective.

## Roles & Responsibilities

**Author** — confirms that the work meets the objective and acceptance criteria.

**Reviewer** — verifies that all mandatory workflow artifacts are present and of sufficient quality, then updates the status.

## Procedure

### 1. Verify Artifact Presence

Check the work item folder for required artifacts:

- [ ] `ticket.md` — must be present.
- [ ] `plan.md` — must be present if the work item was promoted to a plan.
- [ ] `implementation.md` — must be present if the work item involved implementation.
- [ ] `reflect.md` — should be present for non-trivial work items.

### 2. Update Status

Update the `Status` field in `ticket.md` from `pending` (or its current state) to `complete`. 

If a `Completed Date` or similar metadata field is used in the project, update that as well.

### 3. Notify Stakeholders

If the project uses external tracking (issue tracker, project board), update those systems to reflect that work is finished.

## Guidance

- **Completion is Logical, Archive is Physical**: Completing a work item records the fact that work is done. Archiving moves the files.
- **Verification is Mandatory**: Do not mark a work item as complete if verification steps (build/test) from applicable guidelines are failing.
- **Reflect Before Complete**: It is highly recommended to perform the `Reflect` action before `Complete`, as reflection often surfaces minor "polishing" tasks that should be done while the context is fresh.
