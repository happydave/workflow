# Dispatch

## Intent
Package all necessary context, instructions, and guidance into a single, comprehensive briefing for a specific workflow task. This ensures that the downstream session (VS Code Chat, Claude Code CLI, etc.) has exactly what it needs to succeed with minimal context window waste.

Dispatching is the bridge between **orchestration** (tracking what needs to be done) and **execution** (actually doing the work).

## When to Dispatch
- Before starting a new non-trivial task (Plan, Code, Test, etc.).
- When handing off work from an orchestrator to a specialized model/agent.
- When starting a new session to ensure continuity without manual context re-attachment.

## Briefing Storage & Naming
Briefings are persisted in the work item's folder to provide a durable audit trail of what instructions were given.

- **Path**: `docs/pending/<id>-<desc>/briefs/<procedure>-brief.md`
- **Example**: `docs/pending/42-fix-auth-bug/briefs/plan-brief.md`

## Procedure

### 1. Identify Target Task & Environment
Determine exactly which procedure is being dispatched (e.g., `Plan.md`, `Code.md`) and where it will be executed (e.g., `Claude CLI`, `VS Code Chat`).

### 2. Assemble Context
The Dispatcher SHALL determine the minimum necessary files required for the task. This typically includes:
- The authoritative **Procedure** (`procedures/Plan.md`, etc.).
- The **Work Item** (`workitem.md`).
- The **Project** or **Design** (if applicable).
- Relevant **Code** or **Config** files (provide full absolute paths).
- Any **Guidelines** (`skills/go.md`, etc.) that apply to the task.

### 3. Construct the Prompt
Generate a comprehensive, single-block prompt in Markdown format. The prompt must:
- State the **Primary Objective**.
- List all **Reference Material** by providing full absolute file paths.
- Explicitly mention the **Expected Outcome** (e.g., "Create plan.md in the current directory").
- Include any specific constraints or logic determined by the Dispatcher.

### 4. Persist the Briefing
Create the `briefs/` directory if it does not exist and write the prompt to the correctly named file. 

### 5. Transition
Once the briefing is persisted, the orchestrator should trigger the execution using the content of that briefing as the initial input.

## Required Content in the Briefing Artifact
- **Target Procedure**: The name of the procedure being executed.
- **Timestamp**: When the briefing was generated.
- **The Prompt**: The full text to be passed to the execution agent.
- **File List**: A clear list of files the execution agent is expected to read, with absolute paths.
