---
name: github-issue
description: Start working on a GitHub issue — by reading the ticket and proceeding with full planning and implementation
triggers:
  - when the user wants to start working on a GitHub issue
  - when the user mentions an issue number and wants to implement it
  - when the user invokes /github-issue or /issue
---

# GitHub Issue Skill

You are an issue workflow assistant. When triggered, you help the user start working on a GitHub issue by offering two modes.

## Step 1 — Identify the issue

If the user provided an issue number, use it. If not, ask: **"Which issue number would you like to work on?"**

The GitHub organization and repository are inferred from the current project context.

Fetch the issue using the `mcp__github__issue_read` tool with `method: "get"`.

## Step 2 — Present the issue and offer two modes

Display a concise summary of the issue:

```
## Issue #<number> — <title>
<body (truncated to ~300 chars if long)>
```

Then ask the user to choose a mode:

```
How would you like to proceed?

**[1] Branch only** — Create a branch for this issue and switch to it. No implementation yet.
**[2] Full workflow** — Read the ticket, plan the implementation, then proceed with coding.

Reply with 1 or 2.
```

Wait for the user's response before continuing.

---

## Mode 1 — Branch only

### Branch naming

Derive the branch name from the issue using the format:
```
issue/<number>-<short-slug>
```

Where `<short-slug>` is a kebab-case summary of the issue title (max 5 words, lowercase, no special chars).

Example: issue #12 "Introduce tests on server side" → `issue/12-introduce-tests-server-side`

### Steps

1. Present the proposed branch name and ask for confirmation: **"Create and switch to branch `<branch>`? (yes / edit / cancel)"**
2. On **yes**: run `git checkout -b <branch>` and confirm success.
3. On **edit**: ask for the preferred name, then proceed.
4. On **cancel**: stop and inform the user.

---

## Mode 2 — Full workflow

### Step A — Deep-read the issue

Fetch the issue comments via `mcp__github__issue_read` with `method: "get_comments"` to gather any additional context or decisions made in the thread.

### Step B — Plan

Use the `Plan` subagent (via the Agent tool) to design the implementation plan based on:
- The issue title and body
- Any comments fetched
- The current codebase architecture (as described in CLAUDE.md)

Present the plan to the user with a summary of:
- Files to create or modify
- Key decisions and trade-offs
- Any clarifying questions before starting

Ask: **"Does this plan look good? (yes / edit / cancel)"**

Wait for confirmation before proceeding.

### Step C — Create the branch

Apply the same branch naming convention as Mode 1.
Run `git checkout -b <branch>` before writing any code.

### Step D — Implement

Implement the plan step by step, following the project's architecture and conventions defined in CLAUDE.md

### Step E — Commit

Invoke the `conventional-commit` skill to commit the changes.

---

## Important Constraints

- **Never start coding without user confirmation of the plan (Mode 2).**
- **Never skip branch creation** — always work on a dedicated branch.
- **Never push** the branch unless the user explicitly asks.
- If the issue is unclear or too vague, surface the ambiguity before planning.
- If the issue description raise questions or invite to think, be a source of help.
