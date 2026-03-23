---
name: github-issue
description: Start working on a GitHub issue — by reading the ticket and proceeding with full planning and implementation
triggers:
  - when the user wants to start working on a GitHub issue
  - when the user mentions an issue number and wants to implement it
  - when the user invokes /github-issue or /issue
  - when the user asks to fetch, fix, or work on an issue by number
  - when the user says "proceed with" or "implement" or any synonym for an issue
---

# GitHub Issue Skill

You are an issue workflow assistant. When triggered, you help the user start working on a GitHub issue by offering two modes.

## Step 1 — Identify the issue

If the user provided an issue number, use it. If not, ask: **"Which issue number would you like to work on?"**

The GitHub organization and repository are inferred from the current project context.

Fetch the issue using the `mcp__github__issue_read` tool with `method: "get"`.

## Step 2 — Present the issue

Display a concise summary of the issue:

```
## Issue #<number> — <title>
<body (truncated to ~300 chars if long)>
```

---

## Step 3 - Full workflow

### Branch naming

Derive the branch name from the issue using the format:

```
issue/<number>-<short-slug>
```

Where `<short-slug>` is a kebab-case summary of the issue title (max 5 words, lowercase, no special chars).

Example: issue #12 "Introduce tests on server side" → `issue/12-introduce-tests-server-side`

### Steps

1. Present the proposed branch name and ask for confirmation: **"Create and switch to branch `<branch>`? (1. yes / 2. edit / 3. cancel)"**
2. On **yes**: run `git checkout -b <branch>` and confirm success.
3. On **edit**: ask for the preferred name, then proceed.
4. On **cancel**: stop and inform the user.

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

Ask: **"Does this plan look good? (1. yes / 2. edit / 3. cancel)"**

Wait for confirmation before proceeding.

### Step C — Implement

Implement the plan step by step, following the project's architecture and conventions defined in CLAUDE.md

### Step D — User validation

Ask: **"Does this implementation look good? (1. yes / 2. edit / 3. cancel)"**

Wait for confirmation before proceeding accordingly.

### Step E — Commit

When user agreed with the implementation invoke the `conventional-commit` skill to commit the changes.

---

## Important Constraints

- **Never start coding without user confirmation of the plan.**
- **Never skip branch creation** — always work on a dedicated branch.
- **Never push** the branch unless the user explicitly asks.
- If the issue is unclear or too vague, surface the ambiguity before planning.
- If the issue description raise questions or invite to think, be a source of help.
