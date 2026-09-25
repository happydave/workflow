# Retire

## Intent

Close a work item that will not be completed: replaced by another, or no longer wanted. The reason
and any successor are recorded in the item itself, so the backlog shows only owed work and the
folder can be archived like a completed one.

## Statuses

A work item's `status` is one of:

- `pending` — owed. The default.
- `deferred` — still wanted, not now. Open: it counts as owed work everywhere a status is read.
- `complete` — this item's own criteria were checked and met (`Complete.md`), wherever the work was
  done. Work carried out under another item's change gets a short `sidequest.md` here pointing to
  that item's record, which stands in for this item's artifacts.
- `superseded` — another item's scope replaces this one's, so this item's criteria are not checked
  here: a duplicate (superseded by the one kept), or a replacement that takes over work not yet
  done.
- `declined` — it will not be done: obsolete, descoped or rejected.

The last three are terminal. The legacy values `completed` and `reverted` stay valid where they are
already recorded; new records use the five above. Any other value, such as `cancelled`, reads as
open until this procedure normalises it. A completed item whose work is later reverted stays
`complete`: the revert and any redo are new work, recorded by the items that do them. Waiting on
another item or on the owner is not a status: `WorkItem.md`'s `blocked_by` and `gate` fields
record it.

## Procedure

### 1. Decide

Retiring reverses the decision that the work should happen, so it is the owner's: raise it with a
recommendation (`AGENTS.md`), or act on the owner's own instruction, and record the owner's words
and date. Deferring is the owner's too. One case needs no ruling: a duplicate whose kept item is
complete and whose record names this one as closed by it. The session that finds it records it, citing that
record. So does normalising a status whose record already carries the owner's reason.

Before recording any retirement, search the tickets tree for the item's ID — a `blocked_by` naming
it, a phase counting it, a plan relying on it — and say what each hit loses. A retired blocker reads
as satisfied, so a `blocked_by` naming it is repointed to the successor on a supersession, and put
to the owner before a decline is recorded.

### 2. Record

In the frontmatter:

- `status: superseded` or `status: declined`.
- `superseded_by: <id>` on a supersession: the item that delivers or replaces it.
- `resolution: "YYYY-MM-DD: <why>"` — dated the day it is recorded; one line, with the owner's
  ruling (and its date, if earlier) and what was already done if the item was part delivered. Quote
  the whole value: single quotes if it contains a double quote, with any apostrophe inside doubled. A deferral records why, and what would bring it back, the same way.
  Leave the body as written.

### 3. Update the project

Update the item's row in the project doc (`Complete.md` step 3 applies, offload included). If the
item declares a `phase` and was its last open member, say so and point at `Phase.md`'s Close step.

### 4. Commit

Commit every file the retirement changed — the work item, the project doc, any dependent whose
`blocked_by` moved — by path (`GitCommit.md`). Archiving follows only on
request (`Archive.md`), as for a completed item.
