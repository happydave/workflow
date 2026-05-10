# Agent Registry

## Purpose
This document is the central registry for the AI agents defined in this framework. It maps **Procedures** (how to work) to **Agents** (the automated actors), enabling both humans and other AI to discover and invoke the correct capability for any workflow step.

## Agent Directory

| Agent | Target Procedure | Role | Location |
|---|---|---|---|
| **Work Item Agent** | `WorkItem.md` | Intake and clarify work; create `workitem.md`. | `agents/WorkItem.md` |
| **Planner Agent** | `Plan.md` | Transform work items into actionable `plan.md` documents. | `agents/Plan.md` |
| **Plan Reviewer Agent** | `PlanReview.md` | Critically assess `plan.md` for sufficiency; create `planreview.md`. | `agents/PlanReview.md` |
| **Code Reviewer Agent** | `CodeReview.md` | Evaluate implemented code; run `Complete` action. | `agents/CodeReview.md` |
| **SideQuest Agent** | `SideQuest.md` | Execute one-off tasks with minimal overhead. | `agents/SideQuest.md` |
| **Implementer Agent** | `Implement.md` | Implement code from plan; maintain implementation log. | `agents/Implement.md` |
| **Documenter Agent** | `Document.md` | Verify and update documentation post-implementation. | `agents/Document.md` |
| **Reflector Agent** | `Reflect.md` | Capture lessons learned and output `reflect.md`. | `agents/Reflect.md` |
| **Archiver Agent** | `Archive.md` | Move completed work items to `archive/`. | `agents/Archive.md` |
| **Dispatch Agent** | `Dispatch.md` | Package context and instructions into `briefs/`. | `agents/Dispatch.md` |
| **Tester Agent** | `Test.md` | Execute tests and verify results; create `test.md`. | `agents/Test.md` |
| **Discover Agent** | `Discover.md` | Research domains and APIs; create `discover.md`. | `agents/Discover.md` |
| **Claude Agent** | (External) | Delegate complex tasks to Claude Code CLI. | `agents/ClaudeCode.md` |
| **Explore** | (Read-only) | Fast codebase exploration and Q&A. | (Built-in) |

*Planned Agents: None.*

## Interaction Principles

### 1. Progressive Context Disclosure
To optimize for AI context limits (e.g., 100-line initial peeks), this registry provides only the "What" and "Where." Full "How" logic is deferred to specific agent files, which in turn reference Procedural source files only when active.

### 2. Standard Handoffs
Agents are workflow-aware. Upon successful completion of a task, an agent SHALL indicate the next logical step and suggest the appropriate agent to the user (e.g., "**Work Item** complete. You may now invoke the **Planner Agent**"). Handoffs typically end with the **Reviewer Agent** executing the **Complete** action.

### 3. Actor-Agnostic Design
Agents are designed to follow the procedures in `procedures/*.md` exactly as a human would. This ensures that the framework's core logic remains the single source of truth, regardless of whether a human or AI is performing the step.

## How to Invoke
In supported editors (like VS Code), agents can be invoked by name (e.g., `@Work Item Agent`) or via the command palette. Each agent is configured to automatically read the relevant `Procedure` and `Guideline` documents upon activation.
