# Workflow

## Purpose

A structured approach for planning and documenting features with precise, unambiguous language that enables correct implementation on first attempt.

The goal is to produce plans so detailed and clear that they can be implemented correctly without guesswork or misalignment, while leaving non-critical decisions open for optimal choices.

## About

This repository contains meta-instructions — a guide for creating feature plans, not a finished software product. Think of it as the instruction manual before assembling the machine: we define a consistent, unambiguous process first.

The workflow can be supported by **Agents** — automated actors that help execute specific procedures. See [AGENTS.md](AGENTS.md) for the full registry of available capabilities.

## File Layout

**procedures**
- `Project.md` — create and manage high-level initiatives
- `Design.md` — architectural blueprint for projects
- `DesignReview.md` — independent evaluation of designs
- `WorkItem.md` — create and manage work items
- `BugReport.md` — capture a defect as a structured work item with reproduction context
- `GitMerge.md` — plan and execute branch merges: survey divergence, select strategy, execute, record outcome
- `SideQuest.md` — execute and document one-off tasks with minimal overhead
- `Dispatch.md` — package context and instructions for a specialized agent session
- `Discover.md` — investigate products, APIs, or technology domains
- `Investigate.md` — diagnose runtime system behavior using observability data
- `Plan.md` — produce a feature plan with enough detail for correct first-pass implementation
- `Implement.md` — implement incrementally from plans, maintaining an implementation log
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
- `typescript.md` — VS Code extension / Node.js conventions (Docker-based builds)
- `docker.md` — container-first build environment (`Dockerfile.dev` + `Makefile` pattern)
- `markdown.md` — quality gates for Markdown artifacts (link checking, structure verification, spell checking)
- `sql.md` — SQL conventions for queries, schema, and migrations
- `versioning.md` — version increment policy: one ticket, one patch
- `claude-code.md` — Claude Code CLI usage reference for task delegation and automated operations
- `debug.md` — AI agent debugging methodology (structured hypothesis generation, bias mitigation)
- `evidence.md` — evidence assessment rules (Hub for Logs, Metrics, Groundcover)

**templates**
- `implementation-template.md` — rich skeleton template for implementation logs

## Typical Pipelines

- Project: `Create Project → Discover → Design → Design Review → Create Work Item(s)`
- Work Item: `Plan → Plan Review → Implement → Code Review → Test → Document → Reflect → Complete`

### Workflow Architecture

A detailed Mermaid diagram is available for visual representation here: [Workflow Architecture](knowledge/WorkflowArchitecture.md).

## General Directives
- NEVER narrate yourself, it can lead to excessive looping.
- ALWAYS use the `todo` tool (when available) rather than chat.
