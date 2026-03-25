# Project Planning Framework

## Purpose

This framework provides a structured approach for planning and documenting features with precise, unambiguous language that enables correct implementation on first attempt.

The goal is to produce plans so detailed and clear that an AI (or another human) can implement features correctly without guesswork or misalignment, while leaving non-critical decisions open for optimal choices.

## About

Important: This repository contains meta-instructions. It is a guide for creating feature plans, not a finished software product. We are currently refining this framework to ensure AI agents can understand and implement features correctly from the outset. Think of it like building the instruction manual before assembling the machine. At this stage, we are concerned with defining a consistent, unambiguous process — not specific feature requirements yet.

## Document Structure

This framework includes guidelines and core documents for creating plans and descriptions:

- **This README** — Overview of the entire system
- `docs/actions/Ticket.md` — Lightweight intake for work items
- `docs/actions/Plan.md` — Complete workflow instructions and methodology
- `docs/actions/Implement.md` — Implementation process (incremental implementation, implementation log)
- `docs/actions/Reflect.md` — Post-implementation reflection and recommendations
- `docs/actions/Document.md` — Verify documentation is accurate and consistent after changes
- `docs/templates/feature.md` — Instructions for writing lean, outcome-oriented feature plans
- `docs/guidelines/Go.md` — Go-specific guidelines (module setup, tooling, conventions)
- `docs/guidelines/Docker.md` — Container-first build environment guidelines

### Consumer Project Structure

When using this framework in a project, the project repository contains:

- `docs/pending/<ticket-id>/` — active work items, each containing `ticket.md`, `plan.md`, `implementation.md` as they progress
- `docs/completed/<ticket-id>/` — finished work items (moved from `docs/pending/` on completion)

## Using This Framework

### For Project Documentation

Important: This repository contains the framework itself. It is intended to be referenced from inside other projects where the work is actually being done. 

The `docs/templates/` and `docs/guidelines/` folders live in this framework repository. 

When you start a new project (the consumer repository), you should scaffold `docs/pending/` and `docs/completed/` directories there to hold the active execution state.

When documenting an actual project's plans in your consumer repository:

1. **Create tickets for work items**  
   For each feature or task, create a folder in `docs/pending/` with a `ticket.md` inside.  
   Follow the guidance in the framework's `docs/actions/Ticket.md`.

2. **Create feature plans**  
   For each ticket that needs planning, add `plan.md` to the ticket's folder.  
   Follow the guidance in the framework's `docs/templates/feature.md` to include only what is necessary: objective, invariants, behaviors & verifications, edge cases, explicit AI freedom, etc.  
   Avoid code; describe outcomes and constraints clearly while granting freedom on non-critical decisions.

3. **Follow the workflow**  
   Research & elaborate → draft plan → test descriptively → critically assess.  
   Refine until the Critical Assess step confirms the document is unambiguous and complete.

## Plan Style

### Language & Tone

- Plain Markdown only
- Concise, objective, strictly descriptive
- Minimal rhetorical flourish
- No code blocks unless explaining plan format
- Explicitly grant AI freedom in non-critical areas (see `docs/templates/feature.md`)

## Expected Output Quality

A healthy feature plan satisfies:

- Clear objective and non-negotiable invariants
- Behaviors and verifications sufficient to confirm correctness without guesswork
- Edge cases and error responses covered where significant
- No unstated assumptions or contradictions
- Critical decisions stated; non-critical ones left open
- Human and AI remain collaboratively involved throughout refinement

## Current State

This framework is evolving:

- `docs/templates/feature.md` newly added as the single source of truth for feature plans and descriptions
- No full example projects yet demonstrating the workflow end-to-end
- No retrospective documenting lessons learned

The framework can be used ad hoc, but its full value requires committed application with iteration.

## License

This project is released into the public domain under the [Unlicense](https://unlicense.org/).  
You are free to use, modify, distribute, and do whatever you want with it — no restrictions, no attribution required.