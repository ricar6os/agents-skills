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

Start a new session after changing global instructions. Pull repository updates on each machine to keep its local policy current.
