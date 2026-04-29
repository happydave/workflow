# Archive

## Intent

Formally archive a completed work item by moving its folder from `docs/pending/` to `docs/archive/`. This action cleans up the active work directory and moves the auditable trail of the work item to long-term storage.

Archive is a physical file system operation that follows the logical `Complete` action.

## When to Archive

- The work item's `Status` in `ticket.md` is set to `complete`.
- The user or orchestrator explicitly requests archival to clean up the `docs/pending/` directory.
- No further near-term reference to the work item in the active directory is required.

Archive is NOT appropriate when:
- The work item is still in a `pending` or `in-progress` state.
- Ongoing implementation or documentation work is expected in the same folder.

## Roles & Responsibilities

**Reviewer** — verifies the `complete` status and executes the folder move.

## Procedure

### 1. Verify Completion Status

Check the `ticket.md` inside the work item folder. Verify that the `Status` field is set to `complete`. If the status is not `complete`, stop and inform the user that the work item must be finalized via the `Complete` action first.

### 2. Execute Move

Move the entire work item folder atomically from `docs/pending/` to `docs/archive/`:

```
mv docs/pending/<id>-<name>/ docs/archive/
```

This is a single directory-level operation. Individual files must not be copied or moved separately. The entire work item folder — including all artifacts produced during the workflow — moves together.

## Guidance

- **Request Only**: Unlike `Complete`, which is a natural part of every work item's lifecycle, `Archive` is typically performed in batches or when the active workspace becomes cluttered.
- **Permanent Record**: Once archived, a work item is considered a "Closed Book." Any further work related to the same topic should typically start with a new work item that references the archived one.
- **Referenceability**: Archived work items remain fully searchable and referenceable; they simply reside in a different directory to distinguish "Done" from "Doing."
