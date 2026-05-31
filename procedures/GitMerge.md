# GitMerge

## Intent

Plan and execute a branch merge with full awareness of divergence, conflict, and strategy trade-offs. Produce a durable artifact (`gitmerge.md`) recording the chosen strategy and outcome so the merge decision is auditable.

## When to Use

- Two branches must be combined and the divergence is non-trivial.
- Context about which commits matter, what to squash, or whether to rebase would otherwise be lost.

## Artifact Storage

Create `docs/pending/<id>-<name>/gitmerge.md` to record the plan and outcome.

## Procedure

### 1. Survey Branches

Identify the common ancestor and list what has diverged on each side:

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

| Strategy | Use when |
|---|---|
| **Merge** | Preserving branch history matters; commits on both sides are meaningful. |
| **Squash** | Source branch has noisy/WIP commits; a single clean commit on target is preferred. |
| **Rebase** | Linear history is required; source branch has not been published to a shared remote. Do not rebase published branches — this rewrites history visible to other actors. |

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
