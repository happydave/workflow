# GitMerge

## Intent

Plan and execute a branch merge with full awareness of divergence, conflict, and strategy trade-offs. Produce a durable artifact (`gitmerge.md`) recording the chosen strategy and outcome so the merge decision is auditable.

## When to Use

- Two branches must be combined and the divergence is non-trivial.
- Context about which commits matter, what to squash, or whether to rebase would otherwise be lost.

## Direction and Naming

Every operation here has a direction, and the guidance below changes with it. Fix the names before
choosing a strategy:

- **target** — the branch you have checked out. Its ref moves; it carries the result.
- **source** — the branch named in the command (`git merge <source>`, `git rebase <source>`). It is
  not modified.

Both directions are common, and they are not symmetric:

| Direction | Typical reason |
|---|---|
| feature → mainline | Landing finished work. The target is mainline. |
| mainline → feature | Refreshing a long-lived branch so its merge request reflects a current base. The target is the feature branch. |

Record the direction explicitly at the top of `gitmerge.md` — for example, "integrate
`origin/master` **into** `my-feature`". Guidance phrased in terms of "the feature branch" is
ambiguous as soon as the direction is mainline → feature; the strategy guidance below is phrased in
terms of source and target instead.

## Artifact Storage

Create `docs/pending/<id>-<name>/gitmerge.md` to record the plan and outcome.

## Procedure

### 1. Survey Branches

Identify the common ancestor and list what has diverged on each side:

- Direction: `<source>` → `<target>` (see **Direction and Naming**).
- Common ancestor commit (divergence point).
- Commits on the source branch not in target (count + summary).
- Commits on the target branch not in source (count + summary).
- Files changed on each side.

Record findings in `gitmerge.md` under **Survey**.

### 2. Identify Conflicts & Divergence

Determine what will conflict and why:

- Run a dry-run merge or diff to list conflicting files.
- Classify each conflict: independent changes (likely auto-resolvable), overlapping logic changes (require judgment), or structural conflicts (file renames, deletions).

If no divergence is found (branches share the same tip), record this and stop — no merge is needed.

Record findings in `gitmerge.md` under **Conflicts**.

### 3. Select Strategy

Choose one strategy and record the rationale:

| Strategy | Use when | Rewrites history of |
|---|---|---|
| **Merge** (`git merge <source>`) | Preserving branch history matters; commits on both sides are meaningful. | Neither branch. The only strategy that is always safe when either branch is published. |
| **Squash** (`git merge --squash <source>`) | The **source** branch's commits are noisy or WIP and a single clean commit on the target is preferred. Only meaningful when the source is a feature branch — there is no reason to squash mainline into a feature branch. | Neither branch, but the source's individual commits are not carried over. |
| **Rebase** (`git rebase <source>`) | Linear history is required **and the target has not been published** to a shared remote. | **The target** — the branch you are on, not the source. |

> **Rebase names the wrong branch if you read it as "the feature branch."** Rebase replays the
> target's commits onto the source, so the target is what gets rewritten. In the mainline → feature
> direction the target *is* the feature branch, and it is usually the published one. Check whether
> the **target** is published; whether the source is published never matters, and mainline always
> is.

Squashing the target's *own* noisy commits is a different operation — an interactive rebase, not the
Squash row above. Do it before integrating rather than as part of it, and use [`Merge.md`](Merge.md)
if it needs conflict resolution of its own.

Record chosen strategy and rationale in `gitmerge.md` under **Strategy**.

### 4. Execute

Perform the merge using the chosen strategy. For each conflict encountered:

- Resolve using the intent captured in the Survey and Conflicts steps.
- If a conflict cannot be resolved without additional context, record it and escalate rather than forcing an incorrect resolution.

Verify the result: confirm the target branch builds and relevant tests pass where applicable.

### 5. Record Outcome

Update `gitmerge.md` under **Outcome**:

- Final commit hash(es) on the target branch.
- Conflicts resolved and how.
- Any deviations from the planned strategy and why.
