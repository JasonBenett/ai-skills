---
name: pickup
description: Multi-repo issue complete workflow - fetches a GitHub issue, creates branches across relevant repositories, and launches brainstorming before implementation
triggers:
  - when the user wants to start working on a given GitHub issue from a given repository
  - when the user mentions an issue number from a repository and wants to implement it
  - when the user invokes /pickup with an issue number and its repository
  - when the user asks to fetch, fix, or work on an issue by number and repository
  - when the user says "proceed with" or "implement" or any synonym for an issue number and repository
---

# Pickup Skill

Extend the AI agent capabilities to handle end to end an issue/task from a given Github repository. From the analysis to the delivery including the testing and the documentation.
The goal is to make the agent more capable of handling complex tasks involving multiple repositories.

# Workflow

### Step 1 – Identify the issue, repository and its details

Display the following message:
```
# Pickup Step 1 - Identify the issue and the details
--
```

The GitHub organization and repository are inferred from the repository context.

Fetch the issue using the GitHub MCP tools (e.g., `mcp__github__issue_read` or equivalent).
Fetch the comments using fetches GitHub MCP tools (e.g., `get_comments` or equivalent).

Display a concise summary of the issue:
```
--
## Issue #<number> — <title>
<body (truncated to ~300 chars if long)>
```

### Step 2 – Identify the repositories involved

Display the following message:
```
# Pickup Step 2 - Identify the repositories involved
--
```

From the information gathered so far, identify which repositories have to be involved in this task.
Analyze the issue to determine which sub-projects are affected. Consider:
- Keywords
- Issue labels
- Issue details
- The nature of the change (e.g., a new API endpoint likely needs api + app changes)
- Default: if unclear, ask the user.

Display a concise list of the repositories and the reasons why they are involved + ask for user's confirmation:
```
<repository 1> - <reason>
<repository 2> - <reason>

Do you agree or do you want to change the list?
...
```

### Step 3 – Git branches and worktrees

Display the following message:
```
# Pickup Step 3 - Git branches and worktrees
--
```

Verify if the repositories are clean or if they have uncommited changes. 
If they are clean, use a branching workflow. If they have uncommited changes, use a worktree workflow.

Glossary derived from the issue:
* <type>`: either feature or bugfix.
* <number>`: isse number
* <short-slug>`: a kebab-case summary of the issue title (max 5 words, lowercase, no special chars).

Example:
- issue #12 "Introduce functional tests on server side" → type is `feature`, `number` is 12 and short-slug is `introduce-tests-server-side`.

#### Branching workflow

Derive the branch name from the issue using the format:
```
<type>/<number>-<short-slug>
```

Create the branches from the latest main branch and switch to them on all the relevant repositories.
Don't ask for confirmation.

#### Worktrees workflow

Derive the worktrees name from the issue using the format:
```
/.worktrees/<repository>-<number>-<short-slug>
```

Create the worktree from the latest main branch on all the relevant repositories.
Don't ask for confirmation.

### Step 4 – Superpowers

Display the following message:
```
# Pickup Step 4 - Superpowers
--
```

Trigger the /brainstorming command from the superpowers plugin and provide the context you gathered so far to the command.

Pass to the brainstorming skill:
- The issue title, body, and comments
- The list of repositories involved and why
- The branch names/woktrees created
- Any CLAUDE.md conventions relevant to the affected sub-projects

Additional constraints and instructions to Superpowers:
* Each repository involved has its own CLAUDE.md file. Read it before doing anything.
* Use the skill /conventional-commit to generate commit messages.
* Do not commit yourself immediately but ask interactively for user's confirmation first.

### Step 5 – Self validation

```
# Pickup Step 5 - Self validation
--
```

Make sure the standards of each repository is followed and that the rules and tolling defined by the CLAUDE.md are followed.
If violations are found, fix them.

### Step 6 – Final validation

```
# Pickup Step 6 - Final validation
--
```

Gather user's validation for the next step:
- Push the branch?
- Create a pull request?
- Anything to modify?

#### Creating a pull request

Use the Github MCP Server to create a pull request.
Comment the pull request by explaining what you did and why. what is in the scope, what is not in scope, and what is left to do.
Mention also in the pull request the testing plan (checkboxes pre-ticked or not depending on if the tests were automated or if they require manual testing).

## Important Constraints

- **Never push** branches unless the user explicitly asks.
- **Never skip reading a sub-project's CLAUDE.md** before working in it.
- If the issue spans repos ambiguously, ask the user to confirm which repos are in scope.
- Update during each step of the flow, using Github MCP Server, the Github ticket status.
