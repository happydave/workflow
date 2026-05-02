# VS Code Agent Registration Guide

There is currently a documentation conflict regarding where custom agents should reside. This guide clarifies the correct paths and registration methods for making agents available across all workspaces.

---

## 1. Global (User-Level) Location
To make a Markdown-based agent available globally without creating a full extension, place your .agent.md files in the Copilot Global Storage directory rather than the generic User/prompts folder.

### Recommended Paths
If these directories do not exist, you must create them manually:

- Linux: ~/.config/Code/User/globalStorage/github.copilot/customAgents/
- macOS: ~/Library/Application Support/Code/User/globalStorage/github.copilot/customAgents/
- Windows: %APPDATA%\Code\User\globalStorage\github.copilot\customAgents\

---

## 2. File Requirements
For the agent to be recognized by the Copilot Chat interface, the file must use the .agent.md extension and contain valid YAML frontmatter.

Example: my-helper.agent.md

---
name: helper
description: A specialized subagent for code reviews.
---
You are an expert code reviewer. When invoked via @helper, you will...

---

## 3. Extension-Based Registration
If you are developing a formal VS Code extension, you do not use the folders above. Instead, you must declare the agent in your package.json to register it as a Chat Participant.

Example package.json contribution:

"contributes": {
  "chatParticipants": [
    {
      "id": "my-extension.unique-id",
      "name": "myAgentName",
      "description": "Description of the subagent's role",
      "isSticky": true
    }
  ]
}

---

## 4. Troubleshooting the "Prompts" Path
The path ~/.config/Code/User/prompts/ is often referenced in older documentation or experimental features. However:

1. No Auto-Scaffolding: VS Code does not create this folder by default.
2. Standardization: GitHub Copilot has moved toward the globalStorage or extension-based model for stability.
3. Activation: After moving files to the correct directory, a full restart of VS Code (or reloading the window) is usually required to trigger the file watcher.
