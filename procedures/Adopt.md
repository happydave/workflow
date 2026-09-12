# Adopt

## Intent

Carry material across a boundary — a sibling fork of this framework, a third-party skill set, or
this framework outward to another copy — deciding per item what crosses, with source-specific
identifiers stripped and the result verified before it is committed. A SideQuest variant: it keeps
`workitem.md` and one summary artifact, and adds a filter pass and a review, because an adoption is
a bulk edit of authoritative documents whose errors are silent.

Both directions use this procedure, and the redaction list is confirmed in both. An adoption that
looks inbound is also outbound whenever its target can be published without another review.

## When to Use

- Merging generic changes from a fork of this framework, in either direction.
- Importing a skill, procedure, or reference document written elsewhere.
- Not for: content harvested into the codex (`Harvest.md`), or code (`Plan.md` → `Code.md`).

## Procedure

### 1. Work item

Create `workitem.md` (`WorkItem.md`) naming the source, the target, and the direction. All other
record-keeping goes in one `adopt.md` beside it (template below).

### 2. Inventory

Enumerate every source item against the target. For a fork, that is the diff of shared files plus
the source-only and target-only lists; for a fresh source, a file list. Every item gets a row in
the inventory table, including the ones that will be declined — a decline that is not written down
is indistinguishable from an oversight.

### 3. Filter and redaction list

Mark every row **take**, **take with redaction**, or **decline**, each with a one-line reason.
Then, before any edit, write the **redaction list**: the concrete identifiers that must not cross.
Include employer and product names, hosts, repository and group paths, home directories, ticket
and merge request numbers, commit hashes, and people. A measurement or version observed on the
source machine is redacted or re-verified on the target, never copied as fact. Name every conflict
between adopted text and a local directive (a different container runtime, a different host
policy) with its resolution. **The owner confirms the redaction list** — in both directions — and
the confirmation is recorded in `adopt.md` with its date before step 4 begins.

### 4. Execute

Apply the take rows only. Indexes, cross-references, and pipeline listings (`AGENTS.md`,
`README.md`, a skill's Source line) move in the same pass as the content. Write each redacted item
from the source with the listed identifiers replaced, and record the replacement.

### 5. Review

Run in-session; the dimensions carry the stance. Record each result in `adopt.md`. Fidelity and
fit are this procedure's own; the rest are shared with `WorkflowChange.md`.

- **Fidelity** — every take row landed (a file taken as-is is byte-identical to the source:
  `cmp`), every decline row is absent, and `git diff --cached --name-only` lists only the files
  the inventory names.
- **Leak gate** — the redaction list from step 3 over the staged diff, output pasted into
  `adopt.md`, must be empty. When the target is the workflow repo, the site term list is appended
  to the redaction list and the gate runs as `WorkflowChange.md` specifies; otherwise:

  ```sh
  git diff --cached | grep -inE 'term1|term2|term3'
  ```

  Placeholders only in the example: a real term written here trips the gate on this procedure.

- **Coherence** — `skills/markdown.md` build gates on every touched document (links resolve,
  structure holds) and `internal/dupcheck.py` over them; every new procedure or skill has its
  index line. For the workflow repo, the coherence, form and application-test dimensions of
  `WorkflowChange.md` apply in full.
- **Fit** — every conflict named in step 3 is resolved in the text, not left for the reader.

A failed dimension returns to step 4; the review is repeated in full after the fix.

### 6. Record, commit, complete

Complete `adopt.md`, commit by path (`GitCommit.md`), and set the work item to `complete`
(`Complete.md`). Publishing the result is a separate act under the push directive in `AGENTS.md`.

## The `adopt.md` Template

```markdown
# Adopt: <Work Item Title>

**Source:** <path or URL, revision>   **Target:** <path, revision>   **Direction:** <source → target>

## Inventory
| Item | Decision | Reason |
|---|---|---|

## Redaction list
- <term> — <why it is source-specific>
Confirmed by <owner> on <date>.

## Review
- Fidelity / Leak gate (command + output) / Coherence / Fit

## Audit Trail
- `path`: <what changed and why>
```

## Guidance

- Take an item whole or decline it. Where only part is generic, the redacted rewrite is the item.
- Adopted prose keeps the source's voice where it is directive; its history is not adopted.
