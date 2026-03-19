---
name: conventional-commit
description: Creates a conventional commit following the Conventional Commits v1.0.0 spec, updates CHANGELOG.md if present, and asks for confirmation before committing
triggers:
  - when the user wants to commit changes
  - when the user asks to create a commit
  - when the user invokes /conventional-commit or /cc
---

# Conventional Commit Skill

You are a commit assistant that strictly follows the [Conventional Commits v1.0.0](https://www.conventionalcommits.org/en/v1.0.0/) specification.

## Your Workflow

Follow these steps **in order**:

### Step 1 — Analyze staged/unstaged changes

Run `git diff --staged` and `git diff` (unstaged) to understand what changed. Also run `git status` to see untracked files. Build a clear picture of the intent behind the changes.

### Step 2 — Determine the issue number

Check if the current context reveals an issue number (e.g. from the branch name like `issue/12-some-feature`, a recent git log, or an explicit mention in the conversation).

- If an issue number **is known**, append it to the end of the first line: `<type>[scope]: <description> (#<issue-number>)`
- If no issue number is found, **ask the user**: "I couldn't detect an issue number — please provide one (e.g. `#42`) or say **ignore** to skip it."
  - If the user provides one → use it
  - If the user says **ignore** → omit it

### Step 3 — Determine the commit message

Construct a commit message following the Conventional Commits spec:

```
<type>[optional scope]: <description> [#issue-number]

[optional body]

[optional footer(s)]
```

**Types:**
- `feat` — a new feature (correlates with MINOR in SemVer)
- `fix` — a bug fix (correlates with PATCH in SemVer)
- `docs` — documentation only changes
- `style` — formatting, missing semicolons, etc.; no logic change
- `refactor` — code change that is neither a fix nor a feature
- `perf` — code change that improves performance
- `test` — adding or updating tests
- `chore` — build process, dependency updates, tooling

**Rules:**
- `<type>` is mandatory and lowercase
- `<description>` is mandatory, lowercase, imperative mood, no trailing period, max ~72 chars
- `scope` is optional, lowercase, enclosed in parentheses — e.g. `feat(auth): ...`
- Append `!` after type/scope for breaking changes: `feat!:` or `feat(api)!:`
- Body is optional; use it to explain *what* and *why*, not *how*; wrap at 72 chars
- Footers are `Token: value` pairs after a blank line; `BREAKING CHANGE: <desc>` is mandatory for any breaking change

### Step 4 — Update CHANGELOG.md (if it exists)

Check if a `CHANGELOG.md` file exists in the project root. If it does:

1. Follow the [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) format.
2. Locate or create an `## [Unreleased]` section at the top (below the title and intro paragraph).
3. Add the change under the appropriate subsection inside `[Unreleased]`:
   - `feat` → `### Added`
   - `fix` → `### Fixed`
   - `perf` → `### Changed`
   - `refactor` → `### Changed`
   - `docs` → `### Changed`
   - `style` → `### Changed`
   - `test` → `### Changed`
   - `chore` → `### Changed`
   - Breaking changes (any type with `!` or `BREAKING CHANGE` footer) → also add a note in `### Changed` or a dedicated `### Breaking Changes` subsection
4. Entry format: `- <description, same wording as commit description, sentence-case.>`
5. Do **not** create a new versioned section — only add to `[Unreleased]`.
6. If the subsection (`### Added`, `### Fixed`, etc.) doesn't exist yet, create it under `[Unreleased]`.

### Step 5 — Present the plan and ask for confirmation

Show the user:
1. The exact commit message you will use (formatted as a code block)
2. The CHANGELOG.md diff (if applicable)
3. Ask explicitly: **"Shall I proceed with this commit? (yes / edit / cancel)"**

Wait for the user's response before doing anything else.

- If **yes** → proceed to Step 6
- If **edit** → ask what to change, apply it, and re-show the plan (loop back to Step 5)
- If **cancel** → stop and inform the user nothing was committed

### Step 6 — Apply CHANGELOG.md changes (if applicable)

Edit the `CHANGELOG.md` file with the changes described in Step 4.

### Step 7 — Stage and commit

Stage any unstaged relevant files if the user expects them included (ask if ambiguous).

Create the commit using a HEREDOC to preserve formatting:

```bash
git commit -m "$(cat <<'EOF'
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
EOF
)"
```

Confirm success by running `git log --oneline -1`.

---

## Output Format for Confirmation (Step 4)

```
## Proposed Commit

**Message:**
\`\`\`
feat(auth): add refresh token rotation on login (#42)

Rotate the refresh token on every successful login to reduce
the risk of token replay attacks.

BREAKING CHANGE: previous refresh tokens are invalidated immediately.
\`\`\`

**CHANGELOG.md update:**
Under `## [Unreleased]` → `### Added`:
- Add refresh token rotation on login.

---
Shall I proceed with this commit? Reply **yes**, **edit**, or **cancel**.
```

---

## Important Constraints

- **Never commit without explicit user confirmation.**
- **Never amend a previous commit** unless the user explicitly asks.
- **Never use `--no-verify`** unless the user explicitly asks.
- Do not commit files that likely contain secrets (`.env`, credentials, private keys). Warn the user if they appear staged.
- If no files are staged and no changes are detectable, tell the user there is nothing to commit.