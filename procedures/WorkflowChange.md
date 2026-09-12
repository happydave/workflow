# Workflow Change

## Intent

The workflow repo is public and authoritative: a change to it is published to strangers and obeyed
by every later session. So every change to it passes one gate set before `GitCommit.md`, whichever
pipeline carries it — `SideQuest.md`, `Plan.md` → `Code.md`, or `Adopt.md`. This is a gate, not a
pipeline; the carrying procedure says where it runs and which artifact records it.

## Inputs

- The staged diff (`git diff --cached`), staged by path.
- **The site term list** — `tickets/site/redaction.txt` in the site overlay: one extended regular
  expression per line, `#` comments allowed, naming every identifier that must not be published
  (addresses, hostnames, accounts, home paths, the private forge, site-specific products). A missing
  list fails the leak gate; absence never reads as a pass.

## Procedure

### 1. Filter

Every hunk is generic or it moves. Generic means true at any site running this framework. A hunk
that names a host, address, account, home path, realm, private forge, internal tool, or a site's
own project is site-specific: rewrite it as a **role reference** — the private forge, the realm
hosts, the tickets repo, the ops persona, the site knowledge store — and put the value in the site
overlay (`tickets/site/`, whose `AGENTS.md` maps roles to values). Provenance may cite a work item
number; it may not cite a host. The term list is the authority for what *may not* cross; the
filter is the judgement for what *should not*.

### 2. Review

Five dimensions, run in-session; each result is recorded, and a failed dimension returns to the
edit with the whole set repeated afterwards.

- **Leak gate** — the term list over the **added** lines of the staged diff, output pasted, must
  be empty. Removed lines are excluded on purpose: deleting a leaked term is the cleanup, and a gate
  that reads removals fails the change that fixes the leak. The `+++` file headers are excluded so
  a file name cannot trip it.

  ```sh
  git diff --cached -U0 | grep -E '^\+' | grep -vE '^\+\+\+ ' | grep -nE -f tickets/site/redaction.txt
  ```

- **Coherence** — `skills/markdown.md` build gates on every touched document (links resolve,
  structure holds), `internal/dupcheck.py` over them, an index line in `AGENTS.md` for every new
  procedure, skill, knowledge, or agent file, a pipeline line in `AGENTS.md` and `README.md` for
  every new pipeline, the merge union check whenever content is consolidated, and — when the change
  removes or moves a file — a search of the tree for the old path, which the term list cannot see.
- **Form** — per `internal/DESIGN.md` and `skills/authoring-skills.md`: directive, not narrative;
  positive framing; provenance as one Source line; a procedure describes its own step and does not
  restate the broader workflow; procedures and skills aim under 100 lines, and one that lands over
  states its count and why.
- **Application test** — a new or changed procedure or skill is exercised before it is committed:
  by a live run in the same work item, or by a fresh subagent given only the document and a
  realistic task that makes its rules load-bearing. Record which, and what it found. A correctness
  gap found here is fixed and the set repeated.
- **Fit** — the change contradicts no directive in `AGENTS.md` and no existing procedure; where it
  supersedes one, the superseded text is removed in the same change. One authority per rule.

### 3. Record

Write each dimension's result under a **Workflow change gates** heading in the carrying artifact:
`sidequest.md`, `adopt.md`, or `codereview.md`. The leak gate entry carries the command and its
output. The commit follows only when all five are recorded as passed.

## Guidance

- Run the leak gate over the whole tree (`git grep -nE -f <list> HEAD`) when the term list itself
  changes: a new term is a claim about the tree, not only about this diff.
- A site-specific fact that a generic procedure genuinely needs is a sign the procedure should
  name a role and let the overlay supply the value — not a reason to publish the value.
