# Archive

## Intent

Formally archive a completed work item by moving its folder from `docs/pending/` to `docs/archive/`. This action cleans up the active work directory and moves the auditable trail of the work item to long-term storage.

Archive follows the logical `Complete` action.

## When to Archive

- The work item's status is `complete`.
- **Explicit Request**: The user or orchestrator explicitly requests archival. This action should NOT be performed automatically at the end of the `Complete` procedure.
- No further near-term reference to the work item in the active directory is required.

Archive is NOT appropriate when:
- The work item is still in a `pending` or `in-progress` state.
- Ongoing implementation or documentation work is expected in the same folder.

## Roles & Responsibilities

**Reviewer** — verifies the `complete` status and executes the folder move.

## Procedure

### 1. Verify Completion Status

Verify that the work item's status is `complete` by reading the `Status` field in `workitem.md`. If the status is not `complete`, stop and inform the user that the work item must be finalized via the `Complete` action first.

### 2. Execute Move

Move the entire work item folder from `docs/pending/` to `docs/archive/`. Individual files must not be copied or moved separately — the entire folder moves together.

## Guidance

- **Request Only**: Unlike `Complete`, which is a natural part of every work item's lifecycle, `Archive` is typically performed in batches or when the active workspace becomes cluttered.
- **Permanent Record**: Once archived, a work item is considered a "Closed Book." Any further work related to the same topic should typically start with a new work item that references the archived one.
- **Referenceability**: Archived work items remain fully searchable and referenceable; they simply reside in a different directory to distinguish "Done" from "Doing."
