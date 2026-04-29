# Project Planning Framework

## Purpose

A structured approach for planning and documenting features with precise, unambiguous language that enables correct implementation on first attempt.

The goal is to produce plans so detailed and clear that an AI (or another human) can implement features correctly without guesswork or misalignment, while leaving non-critical decisions open for optimal choices.

## About

This repository contains meta-instructions — a guide for creating feature plans, not a finished software product. Think of it as the instruction manual before assembling the machine: we define a consistent, unambiguous process first.

The workflow is supported by **Agents** — automated actors that help execute specific procedures. See [AGENTS.md](AGENTS.md) for the full registry of available capabilities.

## File Layout

**Actions** — step-by-step procedures for each phase:
- `Ticket.md` — intake for work items (keep them short)
- `Discover.md` — investigate products, APIs, or technology domains before planning
- `Investigate.md` — diagnose runtime system behavior using observability data
- `Plan.md` — produce a feature plan with enough detail for correct first-pass implementation
- `Implement.md` — implement incrementally from plans, maintaining an implementation log
- `CodeReview.md` — cooperative human+AI merge request review of implementation artifacts
- `PlanReview.md` — independent evaluation of plan documents before implementation begins
- `Document.md` — verify documentation accuracy after changes
- `Close.md` — formally close a ticket by moving its folder to `docs/complete/`
- `Reflect.md` — capture what went well, what didn't, and concrete recommendations

**Guidelines** — language/tooling-specific rules:
- `Markdown.md` — quality gates for Markdown artifacts (link checking, structure verification, spell checking)
- `TypeScript.md` — VS Code extension / Node.js conventions (Docker-based builds)
- `Go.md` — Go module setup, tooling, conventions
- `Docker.md` — container-first build environment (`Dockerfile.dev` + `Makefile` pattern)
- `Evidence.md` — evidence assessment rules for investigations
- `Debug.md` — AI agent debugging methodology (structured hypothesis generation, bias mitigation)

## Typical Workflow

```
Ticket → Plan → Plan Review → Implement → Code Review → Document → Reflect → Close
```

1. **Ticket** — capture the work item in `docs/pending/<id>/ticket.md` (Assisted by: **Ticket Agent**)
2. **Plan** — produce `plan.md` following the guidance in `procedures/Plan.md` (Assisted by: **Planner Agent**)
3. **Plan Review** — independent evaluation of the plan document before implementation begins (Assisted by: **Reviewer Agent**)
4. **Implement** — build incrementally per plan, log decisions in `implementation.md` (Assisted by: **Implementer Agent**)
5. **Code Review** — cooperative human+AI review of merge requests after implementation
6. **Document** — verify documentation accuracy and consistency after changes (Assisted by: **Documenter Agent**)
7. **Reflect** — capture lessons in `reflect.md` (Assisted by: **Reflector Agent**)

*Discover and Investigate are optional procedures available at any point — before Plan, between steps, or in isolation. Research is assisted by the **Discover Agent**.*

### Consumer Project Setup

When using this framework in a project repo, scaffold `docs/pending/` and `docs/complete/` directories there. The full set of procedure documents (Ticket through Reflect) and guidelines live in this framework repository.
