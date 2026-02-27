---
name: git-flow
description: "Manages git and GitHub workflow. Use when the user wants to commit, push, update their branch, create a PR, update a PR description, or do any git/GitHub operation. Handles: saving work (add+commit+push), updating branch from its base and pushing, creating PRs with title and description, updating PR descriptions."
argument-hint: "[--save|-s] [--update|-u] [--pr] [--update-pr|--upr] [--status|-st] [commit message]"
allowed-tools: Bash
---

# Git Flow

Manage the user's git + GitHub workflow. Always use `gh` CLI for GitHub operations.

## Conventional Commits

ALL commit messages must follow this format exactly:
```
type: description
```

Valid types: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`, `build`, `ci`, `perf`, `revert`

**NEVER add a scope** — no `feat(something):`, only `feat:`.

---

## Flags

Parse `$ARGUMENTS` to detect the flag. Each scenario maps to one or more flags:

| Flag | Aliases | Scenario |
|------|---------|----------|
| `--save` | `-s` | save |
| `--update` | `-u` | update |
| `--pr` | | pr |
| `--update-pr` | `--upr` | update pr |
| `--status` | `-st` | status |

If no flag is provided, infer intent from the message context or ask.

Any text after the flag is treated as the commit message (for `--save`/`-s`).

---

### Scenario: save
**Flag**: `--save` / `-s`
**Trigger also when**: user says "save", "commit", "guardar", "subir cambios", or provides a commit message.

Steps:
1. Run `git status` to show what will be staged
2. Run `git add -A`
3. If no commit message provided, show the diff summary and suggest one based on the changes using conventional commit format
4. Commit: `git commit -m "type: description"`
5. Push: `git push` (if branch has no upstream yet: `git push -u origin HEAD`)
6. Show the result

---

### Scenario: update
**Flag**: `--update` / `-u`
**Trigger also when**: user says "update", "actualizar", "update branch", "traer cambios de main".

Steps:
1. Detect the base branch:
   ```bash
   BASE=$(gh pr view --json baseRefName -q .baseRefName 2>/dev/null)
   if [ -z "$BASE" ]; then BASE="main"; fi
   ```
2. Fetch latest:
   ```bash
   git fetch origin
   ```
3. Merge base into current branch:
   ```bash
   git merge origin/$BASE
   ```
4. If conflicts arise, list them clearly and wait for the user to resolve before continuing.
5. Once merged successfully, push:
   ```bash
   git push
   ```
6. Show a summary: `git log --oneline origin/$BASE..HEAD`

---

### Scenario: pr
**Flag**: `--pr`
**Trigger also when**: user says "pr", "pull request", "abrir pr", "crear pr".

Steps:
1. Check if branch has been pushed: `git status`
2. If unpushed commits exist, push first: `git push` or `git push -u origin HEAD`
3. Check if a PR already exists: `gh pr view 2>/dev/null`
4. **If PR already exists**: show its URL and status with `gh pr view` and stop.
5. **If no PR exists**:
   a. Build the title from the branch name or last commit message (conventional commit format)
   b. Build the description:
      - Get all commits in the branch: `git log --oneline origin/$BASE..HEAD`
      - Check for a PR template: look for `.github/pull_request_template.md` or `.github/PULL_REQUEST_TEMPLATE.md`
      - If a template exists, fill it using the branch changes as context
      - If no template exists, write a concise description summarizing what changed and why
   c. Create the PR:
      ```bash
      gh pr create --title "title here" --body "description here"
      ```
6. Show the PR URL

---

### Scenario: update pr
**Flag**: `--update-pr` / `--upr`
**Trigger also when**: user says "update pr", "update pull request", "actualizar pr", "actualizar descripción del pr".

Steps:
1. Verify a PR exists: `gh pr view 2>/dev/null`. If none, inform the user.
2. Get the base branch: `gh pr view --json baseRefName -q .baseRefName`
3. Get all commits in the branch: `git log --oneline origin/$BASE..HEAD`
4. Get the current PR description: `gh pr view --json body -q .body`
5. Check for a PR template: look for `.github/pull_request_template.md` or `.github/PULL_REQUEST_TEMPLATE.md`
6. Rewrite the description using:
   - The branch commits as source of truth for what changed
   - The template structure if one exists
   - The existing description as context (keep any manually added info that seems intentional)
7. Update the PR:
   ```bash
   gh pr edit --body "updated description here"
   ```
8. Show the PR URL

---

### Scenario: status
**Flag**: `--status` / `-st`
**Trigger also when**: user says "status", "estado", "qué tengo".

Show:
```bash
git status
git log --oneline -5
gh pr view 2>/dev/null || echo "No open PR for this branch"
```

---

## General Rules

- Always run `git status` before staging to show the user what's happening
- Never force push unless explicitly asked
- Never use `--no-verify`
- If on `main` or `master` branch, warn before committing directly and ask to confirm
- When pushing a new branch for the first time, always use `git push -u origin HEAD`
- After every operation, briefly confirm what was done (commit hash, branch, PR URL if applicable)
