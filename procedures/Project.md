# Project

## Intent

Define a high-level goal or initiative that requires multiple work items to achieve. A project provides the context, vision, and architectural direction that individual work items must align with. It serves as an orchestration layer rather than a place for code changes.

## When to Create a Project

- A goal is too large or complex for a single work item
- An initiative involves changes across multiple repositories
- A "discovery" phase reveals a significant new feature area that needs structured decomposition
- Any time high-level design and architectural oversight are needed before breaking down work

Projects are "never finished" in the traditional sense; they exist as long as the initiative is active. A record is archived on the terms under **Status Tracking**.

## Document Storage & Naming

Projects are documented in a central location (e.g., a `tickets` repository), not within individual code repositories. Each project has a folder at `docs/projects/<slug>/` which holds the project record and prose artifacts:

- `design.md`: The architectural design for the project (see `Design.md`).
- `phases/`: One flat file per phase this project owns (see `Phase.md`).
- `runbooks/`: One file per operational procedure the project repeats — deploy to an environment,
  live test, rollback (see `Runbook.md`).
- `archive/`: A folder where completed work items associated with this project are moved.

A project whose output must never leave the host it is made on — derived from material licensed for
local use but not redistribution, say — takes the slug prefix `sandbox-` on the owner's decision, and
so do its repositories, so the constraint shows in every path, link and commit message a session
meets before it acts. *Sandbox* means contained, not throwaway: such a project can be long-lived and
serious. It carries the **Distribution** line under Required Content. The prefix and the line are
added or dropped together, by a dated decision in the record, and dropping or narrowing them waits on
a check of each repository's history for the material that would newly leave; a check that finds
some holds the change until the owner chooses the remedy. A project that plainly qualifies but that the owner
has not marked is raised with the owner before its folder is created.

## Required Content

Every project record must include:

- **Title** — a concise name for the project.
- **Status** — `active`, `inactive`, or `archived`. Starts as `active`. The field holds the word
  alone; an archived record carries the dated reason described under Status Tracking.
- **Purpose** — the high-level "why" and "what" of the project.
- **Scope** — the boundaries of the project, including which repositories or systems are involved.
- **Distribution** — on a `sandbox-` project only: `internal-only`, on the line directly below
  Status, naming the host, what the constraint derives from, and what it covers — by default all the
  project's output: its repositories, artifacts, and data derived from the licensed material.
  Covered output never leaves that host: it is not pushed, published or uploaded, nor copied or
  served to another host, the site's own included; a copy or a loopback view on the same host is not
  a departure. A narrower
  scope — tool code without derived data, say — is the owner's decision to change the line, which an
  instruction to act is not, and is recorded as a dated entry superseding any decision it
  contradicts, with the record's other statements of the scope brought into line. The line narrows
  `AGENTS.md`'s push directive and grants nothing: no push lane covers a `sandbox-` repository, and
  an instruction that would move covered output off the host is answered by quoting the line. The
  record and the project's work items live in the tickets repo, which is pushed, so they carry
  planning and never covered output.

## Optional Content

- **Backlog** — a list of Work Item IDs associated with this project and their current status.
- **Phases** — the phases this project owns, each a record under `phases/` (see `Phase.md`), with their declared status. Membership is derived from the work items' own `phase` fields, never listed here as authority.
- **Runbooks** — the operations this project repeats, each a record under `runbooks/` (see
  `Runbook.md`), with its **Last executed** date. Work items that verify live run these rather than
  re-deriving the procedure.
- **Decisions** — dated, append-only decision entries. Settled decisions recorded in a project
  doc follow the Decision Records convention in `Design.md`: supersede by appending a new dated
  entry that names what it replaces and why — never by rewriting the stamped original.

## Procedure

1. **Initiate** — create `docs/projects/<slug>/project.md` with the Title, Purpose, Scope, and Status set to `active` — and, for a `sandbox-` project, its Distribution line and the dated decision that set it.
2. **Discovery (Optional)** — if the project requires research before design, follow `Discover.md`.
3. **Design** — produce a high-level design following `Design.md`.
4. **Design Review** — subject the design to a formal review following `DesignReview.md`.
5. **Decomposition** — break the design down into individual Work Items following the WorkItem procedure, linking each back to the project. When the design has checkpoints or milestones whose work items must close together, create one phase per checkpoint following `Phase.md` and have each decomposed work item declare its phase.
6. **Execution** — manage the execution of work items.

## Status Tracking

- **active**: Project is being actively worked on.
- **inactive**: Project is paused or deprioritized, and may be picked up again.
- **archived**: Project has ended and no further work is planned — whether the goal was met, another
  project absorbed it, its premise was disproved, or it was never started.

The status field holds the single word, so a lister can filter on it. The reason sits where a reader
meets the status first: on the status line itself where the record states its status in prose, or in
a paragraph of its own immediately below the frontmatter where it does not. It is a dated clause
naming why the project ended and, where one exists, the successor that holds its work — the form is
`**Archived (YYYY-MM-DD).** Goal met: …`, `**Archived (YYYY-MM-DD).** Superseded by …`,
`**Archived (YYYY-MM-DD).** Never started: …`, and so on for whichever ending applies. Date it the
day the record was archived; name any earlier end date inside the clause, at whatever precision is
known. The word alone leaves a reader unable to tell a delivered project from an abandoned one, and
a record that ends without the reason has it re-derived later. Work items still open under an
archived project stay listed in its backlog: the status states an intent and closes none of them,
and what becomes of each is a separate decision.

Status updates are manual.
