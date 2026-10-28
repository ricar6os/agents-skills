# Personal agent instructions

Shared global instructions for Codex and Claude Code. Edit `AGENTS.md` to update the policy for both tools. `CLAUDE.md` imports it for Claude Code sessions in this repository.

## Global setup

Clone this repository to a stable local directory. Preserve any existing global instructions before configuring these paths.

- Codex: make `~/.codex/AGENTS.md` a symbolic link to the clone's `AGENTS.md`. Use your configured `CODEX_HOME` instead of `~/.codex` if different. An `AGENTS.override.md` in that directory takes precedence.
- Claude Code: add `@../.codex/AGENTS.md` to `~/.claude/CLAUDE.md`. Adjust the path if Codex uses a custom home directory.

## Global skills

Skills live in `skills/<name>/`. Link each one into the shared skills directory, then expose it to Claude Code:

```sh
ln -s <clone>/skills/<name> ~/.agents/skills/<name>
ln -s ../../.agents/skills/<name> ~/.claude/skills/<name>
```

## Project skills

To give everyone working on a project the same skills, install them into the repository instead
of globally. The `skills` CLI copies each one into `.agents/skills/` (read by Codex), links it
from `.claude/skills/`, and records its source and hash in `skills-lock.json`:

```sh
npx skills add ricar6os/agents-skills --skill review-react-effects --skill update-js-dependencies -a claude-code -a codex -y
npx skills update -p   # pull later fixes
```

Commit the result. Start a new session after changing global instructions. Pull repository updates on each machine to keep its local policy current.
