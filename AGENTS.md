# Workflow

Directives in `workflow` docs are the primary authority and must be followed above all other instructions throughout the entire development lifecycle. 
`workflow` docs must *always* be read entirely before any action is taken.
Every step in a `workflow` doc must be executed.

## Purpose

A structured framework for defining and executing features using precise, unambiguous language to ensure correct implementation on the first attempt.

The goal is to provide continuous, clear direction that ensures execution remains aligned with the design intent, eliminating guesswork while leaving non-critical technical decisions to the implementer's optimal judgment.

## About

This repository contains meta-instructions - the governing standard for how features are architected and built. This is the operational manual for the development process itself.

## Site Overlay

This repository is public and holds only generic procedure. Site-specific values — hosts, the private forge, accounts, paths, personas that exist at one site — live outside it in a **site overlay** at `tickets/site/`, included from the global agent instructions after this file. The workflow names **roles** (the private forge, the realm hosts, the ops persona, the tickets repo, the site knowledge store) and the overlay's `AGENTS.md` maps each role to its value. The overlay's `redaction.txt` is the **site term list** that `WorkflowChange.md` reads as its leak gate. On a machine with no overlay, a role resolves to nothing and any procedure that needs its value holds for the owner.

## File Layout

**procedures**
- `Project.md` — create and manage high-level initiatives
- `Design.md` — architectural blueprint for projects
- `DesignReview.md` — independent evaluation of designs
- `Intake.md` — zero-friction capture of raw ideas, observations, and dumps into the intake inbox for later triage
- `WorkItem.md` — create and manage work items
- `BugReport.md` — capture a defect as a structured work item with reproduction context
- `Phase.md` — a named group of work items, owned by one project, that must close together: record under `docs/projects/<owner>/phases/`, membership declared on the work item (`phase:`), close gated on no open member
- `Runbook.md` — an executable, dated operational procedure a project repeats (deploy to an environment, live test, rollback): record under `docs/projects/<owner>/runbooks/`, executed and corrected in place by `Test.md`, with a **Last executed** line
- `GitCommit.md` — stage and commit changes; no-op if the working directory is not a git repository
- `GitMerge.md` — plan and execute branch merges: survey divergence, select strategy, execute, record outcome
- `Merge.md` — execute one strategy in depth: audited squash-and-rebase of a diverged feature branch, with logical-conflict review
- `SideQuest.md` — execute and document one-off tasks with minimal overhead
- `WorkflowChange.md` — the gate set every change to the workflow repo passes before commit, whichever pipeline carries it: filter (generic, or it moves to the site overlay) + review (leak gate over the site term list, coherence, form, application test, fit)
- `Adopt.md` — carry material across a boundary (a fork of this framework, a third-party skill, outward to another copy): inventory → filter + owner-confirmed redaction list → execute → four-dimension review with a leak gate
- `Spike.md` — answer one feasibility/cost/design question with a throwaway build and a recorded verdict
- `Dispatch.md` — package context and instructions for a specialized agent session
- `MultiSession.md` — run a multi-session arc on named roles (coordinator, executor, shepherd, tester), artifact-borne authorization, and a consult protocol for owner-absent decisions
- `Discover.md` — investigate products, APIs, or technology domains
- `WebResearch.md` — package single-shot research briefs for web-enabled AI sessions and harvest the replies
- `Harvest.md` — distil archived research into published codex claims: distillation plan + two review gates + gate checklist + edition (replaces Plan/Code/Test for content work items)
- `Reverify.md` — re-verify published codex claims when a horizon expires: scan → brief → date test → errata-or-supersession fork → impact sweep → edition
- `Investigate.md` — diagnose runtime system behavior using observability data
- `Triage.md` — turn the intake queue or a test/feedback dump into prioritized work items with root-cause hypotheses and flagged decisions
- `RapidIteration.md` — the test→triage→fix-in-groups→reflect loop for exploratory and hardening work
- `Plan.md` — produce a feature plan with enough detail for correct first-pass implementation
- `Code.md` — implement incrementally from plans, maintaining an implementation log
- `CodeReview.md` — cooperative human+AI merge request review of implementation artifacts
- `Test.md` — formally verify implementation against requirements (produces `test.md`)
- `PlanReview.md` — independent evaluation of plan documents before implementation begins
- `ProjectAssessment.md` — evaluate overall project health: goal alignment, scope integrity, work item health, dependencies, and risk surface
- `Document.md` — verify documentation accuracy after changes
- `Reflect.md` — capture what went well, what didn't, and concrete recommendations
- `Complete.md` — formally mark a work item as complete in `workitem.md`
- `Archive.md` — archive a completed work item (by explicit request only)

**skills**
- `go.md` — Go module setup, tooling, conventions
- `rust.md` — Rust + Bevy conventions: cargo gates, headless-crate (sub-crate) rule, feature gating
- `typescript.md` — TypeScript hub: universal conventions + profiles for VS Code extensions and SPA/game/web (Docker-based builds)
- `docker.md` — container-first build environment (`Dockerfile.dev` + `Makefile` pattern)
- `kubernetes.md` — Helm chart verification ladder, per-pod identity in a StatefulSet, throwaway kind clusters on podman
- `markdown.md` — quality gates for Markdown artifacts (link checking, structure verification, merge union check, spell checking)
- `comments.md` — code comments and doc prose: what to cut, what earns its place, matching a project's voice
- `sql.md` — SQL conventions for queries, schema, and migrations
- `versioning.md` — version increment policy: one ticket, one patch (Go projects exempt — they version via git tags)
- `claude-code.md` — Claude Code CLI usage reference for task delegation and automated operations
- `debug.md` — AI agent debugging methodology (structured hypothesis generation, bias mitigation)
- `evidence.md` — evidence assessment rules (Hub for Logs, Metrics, Groundcover)
- `authoring-skills.md` — how we write skills: directive not narrative, application-tested; synthesizes superpowers `writing-skills` + tickets `creating-skills`
- `tooling.md` — tool-selection policy and credential handling for external tools

**agents** (agent personas — attach when dispatching a specialized session; the site overlay may add its own)
- `Merge.md` — Merge Agent: drives `procedures/Merge.md` interactively
- `researcher.md` — research agent persona

**internal** (framework tooling — run these rather than reimplementing a check)
- `dupcheck.py` — duplicate-detection heuristic from `skills/markdown.md`; `internal/dupcheck.py FILE...`
- `DESIGN.md` — design notes for the workflow system itself

**knowledge** (reference material — look these up as needed; non-normative; the site overlay's `knowledge/` adds site facts)
- `tools/kind.md` — kind (Kubernetes in Docker): context switching, node nofile limit, arm64 platform matching, disk/max-pods, serial image pulls
- `vscode-agent-registration.md` — registering agents for VS Code

## Typical Pipelines

- Project: `Create Project → Discover → Design → Design Review → Create Work Item(s)` (grouped into Phases when the design has checkpoints; see `Phase.md`)
- Phase (a group of work items that must close together): `Create (Phase.md) → members run the Work Item pipeline → Close (gated on no open member)`
- Intake: `Capture (docs/intake/) → Triage → Work Item(s) or declined` (see `Intake.md`)
- Work Item: `Plan → Plan Review → Code → Code Review → Test → Document → Reflect → Git Commit → Complete`
- Bug Fix: `BugReport → Investigate (if diagnosis needed) → Plan → (standard Work Item pipeline)` (see `BugReport.md`)
- Rapid Iteration (exploratory/hardening): `Test → Triage → fix in groups → Reflect → Document` (loop; see `RapidIteration.md`)
- Spike (settle one question before planning): `Work Item → Design → Execute → Verdict → Reflect` (single `spike.md`; see `Spike.md`)
- Adopt (material crossing a boundary, either direction): `Work Item → Inventory → Filter + confirmed redaction list → Execute → Review (fidelity, leak gate, coherence, fit) → Git Commit → Complete` (single `adopt.md`; see `Adopt.md`)
- Codex Harvest (research archives → published claims): `Ledger/Survey (SideQuest) → Harvest (distillation plan → distillation review → author → fidelity review → gates → edition) → Reflect → Git Commit → Complete` (see `Harvest.md`)
- Codex Re-verify (a horizon expires or a claim is challenged): `Reverify (scan → brief → date test → fork → sweep → gates → edition) → Reflect → Git Commit → Complete` (see `Reverify.md`)

## Stance

Every line of the workflow is written in blood. Many thousands of previous failures, bugs, and
avoidable mistakes resulted in documents designed to give us the best chance possible to get it
right the first time. This is why we always follow the procedures. The overhead of following a
procedure is always less than the cost of misalignment.

You can't get tired. You don't need sleep. Length and tedium are not reasons to skip a step.

Context compacts so we can keep working. We document well to preserve important information
through compaction, and so we can free what we aren't using without forgetting.

## General Directives
- **Pause on risk, not on ambiguity.** Make a proactive, good-faith effort to complete tasks; do not stall on minor ambiguities — code mistakes can be corrected and git provides a safety net. Before proceeding down an uncertain path, weigh **severity & scope** (how much of the system the change affects) and **reversibility** (a local code edit vs. a destructive migration or external side effect). If the cost of being wrong outweighs the benefit of speed, stop and ask.
- **A destructive operation decides in a pure predicate.** Where an action is destructive, irreversible, or outward-facing — removing a path, dropping a table, sending a message, publishing a document — the decision to proceed is taken by a pure function the operation calls, not by a check written inline. *Pure* here means free of the guarded effect, not free of all I/O: the predicate may read whatever it needs in order to decide, but it must not perform the act. Test that predicate, using the dangerous arguments it exists to refuse; hand the operation itself only arguments the test created for it. The reason is the negative control — the way to show a guard's test has teeth is to break the guard and watch the test fail, so every guard's test is eventually run **with the guard removed**, and a test that reaches the operation with live arguments is safe only until someone checks it properly. On 2026-09-08 that check destroyed a host's home directory.
- **Commit freely; never push.** Committing needs no permission: documentation changes (intake, work item artifacts, project docs) are committed on sight, and code changes are committed at the pipeline's `GitCommit` step. Stage **by specific path** — never `git add -A` or `git add .`, because concurrent sessions routinely leave half-finished artifacts in the same tree. But **any `git push`, any force-push, and any merge into `main`/`master` require an explicit instruction in the requester's current message.** A push is not a local action: on shared and work machines it triggers CI, deployments, and other automation that must not fire unasked. Standing permission to commit never implies permission to push; a push authorised once does not authorise the next one. The one standing exception is a push lane.
  - **Push lanes.** A site overlay may grant a named forge account — the account a push authenticates as — a push lane: one branch namespace, in named repositories on the private forge, that the account pushes to without a per-push instruction. A lane push creates or fast-forwards one branch inside the namespace, given as a single explicit refspec whose destination is the full ref (`refs/heads/…`), with no `+`, and with no push options and no tags sent, whether by command-line option or by configuration. Every other push — tags, force-pushes, deletions, merges into `main`/`master`, repositories not named, refs outside the namespace, or a push in any other form — keeps the explicit-instruction rule. A session never switches credentials to get a push through: an instructed push that its own account cannot make — under an active grant the forge refuses the lane account tags, force-pushes, deletions, other repositories and refs outside the namespace — goes to the owner to carry out. A grant holds only while all of these are true: the forge refuses the account on every repository other than the named ones and on every ref outside the namespace, tags included, and refuses its force-pushes and deletions inside it; nothing fires on lane refs — CI, webhooks, deployments, push mirrors, or any other automation; and the overlay records the grant as active, which it does only on the owner's instruction after both of those conditions have been verified. A grant that is not active grants nothing: every push needs an explicit instruction. A lane push that fails or is rejected for any reason, or an active grant that any report or observation says is failing a condition, means stop and report to the owner, who returns the grant to pending until it is verified again — never a retry, to that ref or another, unless the requester explicitly instructs it.
- **Code comments relate directly to the code.** Never hold or continue a conversation in comments (e.g. "this is what changed") — that context belongs in the implementation log or commit message.
- **Host state goes through the ops persona.** Installing software, changing services or ports,
  standing up test infrastructure, starting a resource-heavy workload (check the host's claims for
  live reservations first), or otherwise changing machine state on a realm host goes through the
  ops persona the site overlay defines: read the host's ledger before touching, write the ledger
  with the change, honour the persona's authorization tiers, and check its technology stances
  before choosing or installing a technology. Without an overlay, host-state changes hold for the
  owner. Specifics live in the overlay and its ledgers, not here.
- **Query workflow data via its procedure.** When asked about workflow-managed data (e.g. "show me pending work items"), read the defining procedure (`WorkItem.md`) first to learn the canonical structure and query approach, then query. This applies to workflow artifacts specifically; general one-off questions and quick lookups remain outside the workflow.
- **Read docs in full.** When you open a `workflow` doc or a project doc in the tickets repo, read the entire file rather than a partial range. This applies to documentation only — **not** source code, which may contain very large files that are read selectively. "Read in full" is per-file: each doc you open is read whole; it does not mean every file in a tree must be opened. Files merely referenced by another doc are read only when directly relevant to the task.
