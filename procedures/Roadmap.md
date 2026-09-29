# Roadmap

## Intent

Record a project's **lines**: the unit between a project, which never finishes, and a phase, which
is one checkpoint. A line is what goes through `Discover.md`, `Design.md` and `DesignReview.md` and
has phases filed from it; `Complete.md` and `Phase.md` call it a design line when it closes. The
roadmap is the list of a project's lines, done, active and still to come, with what each needs
before it can start, so a reader answers "what must be unlocked to work on X" from one table rather
than from decomposition paragraphs, sequencing prose, intake files and the decision log. A roadmap
lists lines, never work items: a line small enough to be one work item is a work item.

## When to Keep a Roadmap

- A project has, or will have, more than one design, or names work it intends before designing it.
- Intake holds material that is a future line of the project rather than a work item; `Triage.md`
  dispositions it onto the roadmap.
- A project shares lines with another party (Coordination, below).

Roadmaps are opt-in; a project with one design and one decomposition needs none.

## Document Storage & Naming

One file per project: `docs/projects/<slug>/roadmap.md`, linked from the project record beside its
designs (`Project.md`, Optional Content).

## Format

A table, one row per line, columns in this order:

| Column | Content |
|---|---|
| Line | the line's name, a kebab-case slug, stable once written; other rows' Needs cite it |
| Purpose | one sentence: what is true when the line is done |
| Status | one of `idea`, `researched`, `designed`, `active`, `done`, `dropped` |
| Needs | `none`, or a comma-separated list of line names and `WI <id>` tokens that must be done before this line can start |
| Pointers | its discovery, research archives, design, review and phases, as links, as each appears |

Below the table: **Order**, prose for the owner's preferences among unlocked lines; **Coordination**
when any line is shared with another party; **Decisions**, dated and append-only (`Design.md`).

## The Line

- **Status** is manual, like a project's. `idea`: named, nothing else. `researched`: a discovery or
  a research archive exists. `designed`: its design is approved. `active`: a phase filed from it is
  active, or a member work item is open. `done`: every phase filed from it is complete and its
  design names no unfilled phase. `dropped`: the owner ended it; the row stays, with a dated
  Decisions entry saying why. A line may have phases and no design when the owner files them
  directly, a hardening loop for instance; it is `active` from its first open member.
- **Needs** are modelled, as `blocked_by` is on a work item, where sequencing between one line's
  phases stays prose in its design (`Phase.md`). A need is a hard gate on starting, not a
  preference: a line is **unlocked** when every line it names is `done` and every work item it
  names has a terminal status (`Retire.md`). Preferences among unlocked lines are prose under
  Order. A line names only lines on the same roadmap and work items; a need on another project's
  line is a line here that waits on it, so a tool reads one file.
- **Authority.** The roadmap is authoritative for a line's existence, status and needs. A phase's
  state stays derived from its record and members; a tool that finds a line `designed` while a
  phase filed from it is active reports the disagreement, and the row is corrected by hand.

## Procedure

1. **Create** — write the table from the project's designs (one line per design, its phases as
   pointers), the phases filed without a design, and the future lines the owner has named. Link it
   from the project record.
2. **Add a line** — a name, a purpose and `idea`, from the owner's word, a triage disposition, or a
   design that splits into more than one line. Name its needs when known; `none` otherwise.
3. **Advance** — the session that records an event moves the status: a discovery or archive lands
   (`researched`), a design is approved (`designed`), a phase is activated or a member opened
   (`active`), the last phase closes (`done`; `Complete.md` and `Phase.md` point here).
4. **Drop** — set `dropped` and append the dated reason. Never delete a row: a roadmap that loses
   lines silently is the failure it exists to prevent.
5. **Coordinate** — for a line shared with another party, the Coordination section names the line,
   the seam (a format, a protocol, a spec, lessons), which side owns it, and where the shared
   artifact lives. Material crosses the boundary through `Adopt.md`; a project's Distribution line
   (`Project.md`) decides what may cross at all, and the section restates it.

## Guidance

- Keep the row a line. A purpose that needs a paragraph is a design's Context; write the design.
- Needs are what must be true, not what would be nice; an empty Needs cell is a claim that the
  line can start now, so write `none` deliberately.
- The tech tree is drawn from the table by a listing tool; the table carries no dates or estimates.

Source: a project that ran four lines informally before the record existed (WIs 1980, 1981).
