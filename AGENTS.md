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

This repository is public and holds only generic procedure. Site-specific values — hosts, the private forge, accounts, paths, personas that exist at one site — live outside it in a **site overlay** at `tickets/site/`, included from the global agent instructions after this file. The workflow names **roles** (the private forge, the realm hosts, the ops persona, the tickets repo, the site knowledge store) and the overlay's `AGENTS.md` maps each role to its value. The overlay's `redaction.txt` is the **site term list** that `WorkflowChange.md` and `Adopt.md` read as their leak gate. On a machine with no overlay, a role resolves to nothing and any procedure that needs its value holds for the owner.

## File Layout

**procedures**
- `Project.md` — high-level initiatives
- `Design.md` — a project's architectural blueprint
- `DesignReview.md` — independent evaluation of a design
- `Intake.md` — zero-friction capture of raw ideas, observations, and dumps for later triage
- `WorkItem.md` — create and manage work items
- `BugReport.md` — a defect captured as a work item with reproduction context
- `Phase.md` — a group of work items, owned by one project, that must close together
- `Runbook.md` — a dated operational procedure a project repeats, corrected by every run
- `GitCommit.md` — stage by path and commit; never pushes
- `GitMerge.md` — plan and execute a branch merge, recording the strategy
- `Merge.md` — audited squash-and-rebase of a diverged branch, with a logical-conflict review
- `SideQuest.md` — one-off tasks with a single audit document
- `WorkflowChange.md` — the gates every change to this repo passes before commit
- `Adopt.md` — carry material across a boundary: filter, confirmed redaction list, review
- `Spike.md` — settle one question with a throwaway build and a recorded verdict
- `Dispatch.md` — package context and instructions for another session
- `MultiSession.md` — a multi-session arc on named roles and artifact-borne authorization
- `Discover.md` — investigate a product, API, or technology domain
- `WebResearch.md` — single-shot research briefs for web-enabled sessions, and their harvest
- `Harvest.md` — distil archived research into published codex claims
- `Reverify.md` — re-verify published codex claims when a horizon expires
- `Investigate.md` — diagnose runtime behavior from observability data
- `Triage.md` — turn intake or a feedback dump into prioritized work items
- `RapidIteration.md` — the test → triage → fix in groups → reflect loop
- `Plan.md` — a feature plan detailed enough for a correct first pass
- `PlanReview.md` — independent evaluation of a plan before implementation
- `Code.md` — implement from the plan, keeping an implementation log
- `CodeReview.md` — review the implementation against the plan
- `Test.md` — verify the implementation against its requirements (produces `test.md`)
- `Voice.md` — trim and voice the comments and prose a work item added (produces `voice.md`; mandatory)
- `Document.md` — verify documentation accuracy after changes
- `Reflect.md` — what went well, what did not, and recommendations
- `ProjectAssessment.md` — evaluate a project's overall health
- `Complete.md` — mark a work item complete
- `Retire.md` — close a work item that will not be completed; defines the status values
- `Archive.md` — archive a closed work item, on request only

**skills**
- `go.md` — Go tooling, testing, and conventions; `go/` holds its sub-files for concurrent tests, mocks, long-lived handlers, and lint
- `rust.md` — Rust and Bevy conventions
- `typescript.md` — TypeScript hub, with profiles for VS Code extensions and web apps
- `live-stack-testing.md` — tests that drive a running multi-process system
- `docker.md` — container-first build environment
- `kubernetes.md` — Helm chart verification, StatefulSet identity, throwaway kind clusters
- `markdown.md` — quality gates for Markdown artifacts
- `comments.md` — code comments, doc prose, and review replies: what to cut, what to keep
- `sql.md` — SQL conventions
- `versioning.md` — one ticket, one patch (Go exempt)
- `claude-code.md` — Claude Code CLI reference
- `debug.md` — debugging methodology for agents
- `evidence.md` — confidence labels and evidence rules for any empirical claim
- `measurement.md` — what a plan requires of a measurement or a property harness
- `authoring-skills.md` — how we write skills
- `tooling.md` — tool selection and credential handling
- `documenting-architecture/` — a repository's root `ARCHITECTURE.md` code map

**agents** — attach when dispatching a specialized session; the site overlay may add its own
- `Merge.md` — drives `procedures/Merge.md` interactively
- `researcher.md` — web research persona

**internal** — run these rather than reimplementing a check
- `dupcheck.py` — duplicate detection from `skills/markdown.md`
- `DESIGN.md` — design notes for the framework itself

**knowledge** — reference material, non-normative; the site overlay's `knowledge/` adds site facts
- `tools/kind.md` — operating a kind cluster
- `vscode-agent-registration.md` — registering agents for VS Code
- `negative-controls.md` — how a full negative-control sweep runs, and the worked cases behind the rules

## Typical Pipelines

- Project: `Create Project → Discover → Design → Design Review → Create Work Item(s)` (grouped into Phases when the design has checkpoints; see `Phase.md`)
- Phase (a group of work items that must close together): `Create (Phase.md) → members run the Work Item pipeline → Close (gated on no open member)`
- Intake: `Capture (docs/intake/) → Triage → Work Item(s) or declined` (see `Intake.md`)
- Work Item: `Plan → Plan Review → Code → Code Review → Test → Voice → Document → Reflect → Git Commit → Complete`
- Bug Fix: `BugReport → Investigate (if diagnosis needed) → Plan → (standard Work Item pipeline)` (see `BugReport.md`)
- Rapid Iteration (exploratory/hardening): `Test → Triage → fix in groups → Reflect → Document` (loop; see `RapidIteration.md`)
- Spike (settle one question before planning): `Work Item → Design → Execute → Verdict → Reflect → Git Commit → Complete` (single `spike.md`; see `Spike.md`)
- Adopt (material crossing a boundary, either direction): `Work Item → Inventory → Filter + confirmed redaction list → Execute → Review (fidelity, leak gate, coherence, fit) → Git Commit → Complete` (single `adopt.md`; see `Adopt.md`)
- Codex Harvest (research archives → published claims): `Ledger/Survey (SideQuest) → Harvest (distillation plan → distillation review → author → fidelity review → bless → gates → voice → edition) → Document → Reflect → Git Commit → Complete` (see `Harvest.md`)
- Codex Re-verify (a horizon expires or a claim is challenged): `Reverify (scan → brief → date test → fork → sweep → gates → voice → edition) → Reflect → Git Commit → Complete` (see `Reverify.md`)

## Stance

Every line of the workflow is written in blood. Many thousands of previous failures, bugs, and
avoidable mistakes resulted in documents designed to give us the best chance possible to get it
right the first time. This is why we always follow the procedures. The overhead of following a
procedure is always less than the cost of misalignment. A line is added only when a failure has
repeated, or once when it caused a loss — data, a host, published content (`Reflect.md`).

You can't get tired. You don't need sleep. Length and tedium are not reasons to skip a step.

Context compacts so we can keep working. We document well to preserve important information
through compaction, and so we can free what we aren't using without forgetting.

## General Directives
- **Pause on risk, not on ambiguity.** Make a good-faith effort to complete the task; do not stall on minor ambiguities — mistakes can be corrected and git is the safety net. Before an uncertain step, weigh severity and scope against reversibility; if the cost of being wrong outweighs the benefit of speed, stop and ask.
- **An owner decision carries a recommendation.** Wherever a decision is left to the owner — in conversation, at a gate, or as an open question in an artifact — name the option you recommend and why, beside the alternatives, and what would change it. The owner decides; a recommendation is never acted on as though it were the decision.
- **A destructive operation decides in a pure predicate.** Where an action is destructive, irreversible, or outward-facing, the decision to proceed is a pure function the operation calls — free of the guarded effect, though it may read what it needs. Test the predicate with the dangerous arguments it exists to refuse, and hand the operation only arguments the test created: every guard's test is eventually run with the guard removed, and one that reaches the operation with live arguments performs it (`knowledge/negative-controls.md`; the 2026-09-08 loss).
- **Commit freely; never push.** Documentation changes are committed on sight and code at the pipeline's `GitCommit` step, staged by specific path — never `git add -A` or `git add .`, since concurrent sessions leave half-finished artifacts in the same tree. Any `git push`, force-push, or merge into `main`/`master` requires an explicit instruction in the requester's current message: a push fires CI and deployments, and one authorised push does not authorise the next. The one standing exception is a **push lane** the site overlay grants a named forge account: one branch namespace in named repositories, where a lane push creates or fast-forwards one branch as a single explicit refspec to a full `refs/heads/…` ref, with no `+`, and no push options and no tags sent, whether by command-line option or by configuration. Every other push keeps the instruction rule, and a session never switches credentials to get a push through — a push its own account cannot make goes to the owner. The overlay states a grant's conditions and whether it is active; a grant that is not active grants nothing, and a lane push that fails, or a condition reported failing, means stop and report to the owner, who returns the grant to pending — never a retry unless the requester instructs it.
- **Code comments relate directly to the code.** Never hold a conversation in a comment; that context belongs in the implementation log or the commit message.
- **Host state goes through the ops persona.** Installing software, changing services or ports, standing up test infrastructure, or starting a resource-heavy workload on a realm host goes through the ops persona the site overlay defines: read the host's ledger first, write it with the change, and honour the persona's authorization tiers and technology stances. Without an overlay, host-state changes hold for the owner.
- **Query workflow data via its procedure.** Asked about workflow-managed data ("show me pending work items"), read the defining procedure (`WorkItem.md`) first, then query. One-off questions and quick lookups stay outside the workflow.
- **Read docs in full.** A `workflow` doc or a tickets project doc you open is read whole, never in ranges; source code is read selectively. The rule is per file: a file another doc merely references is opened only when directly relevant.
