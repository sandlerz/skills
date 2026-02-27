---
name: git-flow
description: "Manages git and GitHub workflow. Use when the user wants to commit, push, update their branch, create a PR, or do any git/GitHub operation. Handles: saving work (add+commit+push), updating branch from its base, creating PRs, and full workflows."
argument-hint: "[save|update|pr|sync] [commit message]"
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

## Scenarios

Detect what the user wants from `$ARGUMENTS` or their message. If unclear, ask.

---

### Scenario: save
**Trigger**: user says "save", "commit", "guardar", "subir cambios", or provides a commit message.

Steps:
1. Run `git status` to show what will be staged
2. Run `git add -A`
3. If no commit message provided, show the diff summary and ask for one (suggest one based on the changes using conventional commit format)
4. Commit: `git commit -m "type: description"`
5. Push: `git push` (if branch has no upstream yet: `git push -u origin HEAD`)
6. Show the result

---

### Scenario: update
**Trigger**: user says "update", "actualizar", "update branch", "traer cambios de main".

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
5. Show a summary: `git log --oneline origin/$BASE..HEAD`

---

### Scenario: pr
**Trigger**: user says "pr", "pull request", "abrir pr", "crear pr".

Steps:
1. Check if branch has been pushed: `git status`
2. If unpushed commits exist, push first: `git push` or `git push -u origin HEAD`
3. Check if a PR already exists: `gh pr view 2>/dev/null`
4. If no PR exists: `gh pr create` — use the last commit message as title suggestion, ask for description or use `--fill`
5. If PR exists: show its URL and status with `gh pr view`

---

### Scenario: sync
**Trigger**: user says "sync", "sincronizar", "update and push".

Steps:
1. Run the **update** scenario (merge from base branch)
2. If update succeeded without conflicts, push: `git push`
3. Show final status

---

### Scenario: status
**Trigger**: user says "status", "estado", "qué tengo".

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
