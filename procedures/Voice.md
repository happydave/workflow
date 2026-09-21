# Voice

## Intent

Trim and voice what a work item added — code comments, documentation prose, and drafted replies —
once, after the implementation is finished and verified, so the pass sees the complete picture.
Comments written while a change is still being worked out argue the case the author was making to
themselves; once the branch has settled those sentences are dead, and the ones that remain can be
judged against the defects the tests have paid for. `skills/comments.md` is the skill this step
applies.

## When

After `test.md` exists and before Document, for every work item. The step is never skipped: when
nothing in scope was added or changed, the artifact is one line.

## Procedure

### 1. Identify Scope

Take the diff of this work item's changes: `git diff <base>` for uncommitted work, staged and
unstaged; `git diff <base> <last commit>` once it is committed, so later commits by others stay out.
The base is the merge-base with the target branch, or, for work committed directly on the trunk,
the last commit before the item's first change. Where other work is interleaved on the trunk, diff
the item's own commits one by one instead of the range. From the diff collect:

- comment lines added or changed
- documentation prose files added or changed
- reply drafts this work item wrote to be posted, quoted in `code.md` or `codereview.md` — the
  quoted text itself, not files it refers to; a work item's test fixtures are never in scope

Changes outside git (a file in the central tickets folder) are listed by path with the changed lines
named. Record the base and the list. If the list is empty, go to step 5.

### 2. Read Reference Files

Per `skills/comments.md` step 1: two or three files in the same layer you would be happy to have
written; when the layer is a directory of like documents, its neighbours are the reference. When
the work item touched every file in its layer, the reference is their own base revisions and the
nearest layer out; when neither has enough to read, `voice.md` says so and the skill alone decides.
Once per work item, before touching anything — this is the step that pays.

### 3. Apply the Skill

Sentence by sentence over the scope; the category decides each call. A comment outside the scope is
left alone even when it deserves the same treatment. A kept comment that passes the keep-test stays,
whatever the Author's taste. Reply drafts are voiced under the skill's reply rules even when they will
not be posted — they are the text the owner pastes.

### 4. Rerun the Build Gates

When step 3 changed anything: those named in `plan.md`'s Applicable Guidelines — a formatter, a
link check. A gate that fails the same way at the item's base is out of scope: this step is
behaviour-neutral and leaves it. Read the base, never the edited tree: a worktree of the base, or
the base revisions of the touched files where the gate reads only those. A cut that breaks a gate
the base passed is fixed or reverted; rerun until clean. A reverted cut is noted in `voice.md`.

### 5. Write `voice.md`

Write it after step 3's cuts are in the files, from the diff that shows them: the working diff of
the touched files, since step 1's range is frozen at the item's last commit and holds none of this
pass's work. Record a cut only after confirming it landed there — a draft written from the cuts the
pass meant to make records edits that do not exist (WIs 1651, 1669).

In the work item folder: the base and scope from step 1, the reference files read, and only the cuts
a later reader might miss — a removed reason, a sentence moved to a test's assertion message, a
sentence kept with its argument cut away. Most runs are a few lines. When step 1 found nothing in
scope, the file is the single line:

```
Nothing to trim.
```

## Guidance

- Behaviour-neutral: comments and prose only, no code tokens. The one exception is a test's
  assertion message, where the skill lets cut substance land. Any other change belongs to a Code step.
- Document follows and verifies the trimmed text for accuracy: Document updates content; this step
  sets voice.
- This step is where `skills/comments.md` is applied, and it is applied to this work item's own
  changes. Reviewing another author's work does not include it.
