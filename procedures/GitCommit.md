# GitCommit

## Intent

Stage and commit changes in the current working directory with a clear, accurate commit message.

This procedure commits only. **It never pushes.** Whether a push may follow is decided by the "Commit freely; never push" directive in `AGENTS.md`: an explicit instruction from the requester, or a push lane the site overlay grants.

## When to Use

- A change is ready to be recorded in version control.
- Explicitly requested by the requester.

## Procedure

### 1. Verify Git Repository

Run `git status` to confirm the working directory is inside a git repository.

If it is not a git repository, notify the requester and stop — the procedure is complete.

### 2. Survey Staged and Unstaged Changes

Identify what will be committed:

- Review `git status` output: staged files, unstaged modifications, and untracked files.
- Run `git diff` (unstaged) and `git diff --cached` (staged) to understand the substance of each change.
- Confirm no unintended files are included (secrets, build artifacts, editor temp files).
- Once everything is staged, where the diff adds prose that reports a result or a survey finding, verify, label or delete each claim that the thing under test is fine — *The Claims That Need Checking Are the Comfortable Ones* in `skills/evidence.md`, including its grep over the added Markdown lines, and *State the Sample a Claim Rests On* for a fix claim.

Stage additional files as needed, by specific path — never `git add .`, `git add -A` or `git commit -a` (`AGENTS.md`, *Commit freely; never push*).

### 3. Draft Commit Message

Write a commit message that reflects the *why*, not just the *what*:

- Subject line: imperative mood, under 72 characters, no trailing period.
- Body (optional): include only if the subject alone would leave the rationale unclear.
- Do not reference internal task IDs, ticket numbers, or ephemeral context unless the project convention requires it.

### 4. Commit

Create the commit. Do not skip hooks (`--no-verify`) unless the requester explicitly instructs it.

If a pre-commit hook fails, fix the underlying issue and retry — do not amend a prior commit to recover.

### 5. Confirm

Run `git log -1` to confirm the commit was recorded with the correct message and author.

Report the commit hash and subject line to the requester.
