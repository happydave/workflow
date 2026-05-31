---
name: claude-code
description: Claude Code CLI usage reference for task delegation and automated operations
---
# Claude Code CLI Guidelines

## Purpose
This guideline provides the technical reference and best practices for interacting with the Claude Code CLI (`claude`). It ensures consistent, efficient, and safe use of the CLI for task delegation and automated operations.

## Core Commands

### Interactive Session (Default)
```bash
claude
```
Starts an interactive agentic session. Use this for complex, multi-step tasks where human oversight is preferred.

### Non-Interactive (One-shot)
```bash
claude "your prompt" --print
```
Executes the prompt and prints the result to stdout before exiting. Ideal for automated checks or quick queries.

### Continuing a Session
```bash
claude -c "follow-up instruction" --print
```
Resumes the most recent conversation in the current directory.

## Key Options

| Option | Description | Recommended Usage |
|---|---|---|
| `--print` | Non-interactive output. | Always use when calling from scripts or other agents. |
| `--model <model>` | Specify the model (e.g., `sonnet`, `opus`). | Use `sonnet` (default) for speed/coding, `opus` for complex reasoning. |
| `--effort <level>` | `low`, `medium`, `high`, `xhigh`, `max`. | `medium` for standard tasks, `high` for complex refactoring. |
| `--permission-mode <mode>` | `acceptEdits`, `auto`, `bypassPermissions`, `default`, `dontAsk`, `plan`. | `acceptEdits` is usually the best balance for automation. |
| `--json-schema <schema>` | Validate output against a JSON schema. | Use when structured data is required for downstream tools. |
| `--ide` | Connect to an active IDE instance. | Use when working within VS Code to sync state. |

## Usage Patterns

### Concurrency & Isolation
By default, Claude Code resumes the "most recent" session in a directory. To prevent race conditions and ensure traceability:
- **Mandatory Session ID**: Use `--session-id <UUID>` (MUST be the VS Code session ID) for all operations unless explicitly instructed otherwise or when the ID is unavailable.
- **Stateless Execution**: Use `--no-session-persistence` ONLY for ephemeral, one-off tasks that specifically must not be recorded in session history.
- **Forking**: Use `--fork-session` when resuming an existing task but you want to preserve the original state as a checkpoint.

### Task Delegation
When a task is too complex or requires a specialized agent, delegate it to Claude Code:
```bash
claude "Implement the logic for X in file Y, following skills/go.md" --print
```

### Automated Code Review
```bash
git diff | claude "Review these changes for security issues and performance bottlenecks." --print
```

### Contextual Awareness
Claude Code automatically discovers `CLAUDE.md` and `.claude/settings.json`. Ensure these files exist in the project root to provide persistent context.

## Safety & Permissions
- Avoid `--dangerously-skip-permissions` unless in a strictly controlled sandbox environment.
- Prefer `acceptEdits` mode to allow Claude to make changes while still requiring a final "Enter" or "Y" if run interactively, or providing a clear audit trail.
- Be mindful of token costs when using `high` or `max` effort levels.
