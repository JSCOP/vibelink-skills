# VibeLink Skills

Agent skills published by [VibeLink](https://github.com/JSCOP/vibelink-releases).

## vibelink-memory

Lets any coding agent read and record the durable project facts VibeLink keeps for a workspace,
through the `vibelink memory` CLI. One shared store, every agent.

### Install

Pick the agents you actually use — never install to every agent at once, or you will get config
directories for agents you do not have.

```
npx skills add JSCOP/vibelink-skills --skill vibelink-memory --global --agent claude-code --agent codex
```

Valid `--agent` keys are printed by `npx skills add --agent x`. Common ones: `claude-code`,
`codex`, `cursor`, `opencode`, `gemini-cli`, `github-copilot`, `windsurf`, `zed`, `universal`.

VibeLink can also install this skill for you, offline, from Settings -> AI -> Memory. Both paths
write the same file.

### Update

```
npx skills update vibelink-memory --global
```

### What it writes

One file: `<agent home>/skills/vibelink-memory/SKILL.md`. It never edits an agent's config file.

## License

MIT
