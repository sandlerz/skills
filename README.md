# sandlerz/skills

My personal collection of [Agent Skills](https://agentskills.io) for Claude Code and other AI coding agents.

## Install all skills

```bash
npx skills add sandlerz/skills --all
```

## Install a specific skill

```bash
npx skills add sandlerz/skills --skill <skill-name>
```

## List available skills

```bash
npx skills add sandlerz/skills --list
```

## Skills

| Skill | Description |
|-------|-------------|
| [git-flow](skills/git-flow/SKILL.md) | Git + GitHub workflow manager. Flags: `--save` / `-s` (add+commit+push), `--update` / `-u` (merge base branch + push), `--pr` (create PR with title, description and template support), `--update-pr` / `--upr` (rewrite PR description from branch commits), `--status` / `-st` (current state overview) |

## Compatible agents

These skills follow the open [Agent Skills specification](https://agentskills.io) and work with Claude Code, Codex, Cursor, and more.
