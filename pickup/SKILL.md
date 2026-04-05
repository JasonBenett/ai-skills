---
name: pickup
description: Multi-repo issue complete workflow - fetches a GitHub issue, creates branches across relevant repositories, and launches brainstorming before implementation
triggers:
  - when the user wants to start working on a GitHub issue
  - when the user mentions an issue number and wants to implement it
  - when the user invokes /pickup
  - when the user asks to fetch, fix, or work on an issue by number
  - when the user says "proceed with" or "implement" or any synonym for an issue
---

# Pickup Skill

Extend the AI agent capabilities to handle end to end an issue/task. From the analysis to the delivery.

# Workflow

### Step 1 - Identify the issue and its details

If the user provided an issue identifier, use it. If not, ask: **"Which issue would you like to work on?"**

The GitHub organization and repository are inferred from the repository context.

Fetch the issue using the GitHub MCP tools (e.g., `mcp__github__issue_read` or equivalent).
Fetch the comments using fetches GitHub MCP tools (e.g., `get_comments` or equivalent).

Display a concise summary of the issue:
```
## Issue #<number> — <title>
<body (truncated to ~300 chars if long)>
```

### Step 2 - Identify the repositories involved

From the information gathered so far, identify which repositories have to be involved in this task.
Analyze the issue to determine which sub-projects are affected. Consider:
- Keywords
- Issue labels
- The nature of the change (e.g., a new API endpoint likely needs api + app changes)
- Default: if unclear, ask the user.

Display a concise list of the repositories and the reasons why they are involved :
```
<repository 1> - <reason>
<repository 2> - <reason>
...
```

### Step 3 - Git branches

Derive the branch(es) name(s) from the issue using the format:

```
<type>/<number>-<short-slug>
```

Where:
* <type>` is either feature or bugfix.
* <short-slug>` is a kebab-case summary of the issue title (max 5 words, lowercase, no special chars).

Example: issue #12 "Introduce tests on server side" → `feature/12-introduce-tests-server-side`.

Create the branches from the latest main branch and switch to them on all the repositories where it's relevant based on Step 2.

Note: Always ask for user confirmation for each repository:

```
Create and switch to branch <branch-name> on <repository>?
```

### Step 4 - Superpowers

Trigger the /brainstorming command from the superpowers plugin and provide the context you gathered so far to the command.

Pass to the brainstorming skill:
- The issue title, body, and comments
- The list of repositories involved and why
- The branch names created
- Any CLAUDE.md conventions relevant to the affected sub-projects

Additional constraints and instructions to Superpowers:
* Use the skill /conventional-commit to generate a commit message.
* Do not commit yourself immediately but ask interactively for user's confirmation first.
* Each repository involved has its own CLAUDE.md file. Read it before doing anything.

## Important Constraints

- **Never create branches without showing them to the user first.**
- **Never push** branches unless the user explicitly asks.
- **Never skip reading a sub-project's CLAUDE.md** before working in it.
- If the issue spans repos ambiguously, ask the user to confirm which repos are in scope.
