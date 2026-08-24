# VibeLink Skills

Agent skills published by [VibeLink](https://github.com/JSCOP/vibelink-releases).

Each skill is a single `skills/<name>/SKILL.md` document that teaches a coding agent a
contract. Install only the ones you want, for only the agents you actually use.

## Skills

| Skill | What it gives an agent |
| --- | --- |
| `vibelink-memory` | Read and record the durable project facts VibeLink keeps for a workspace, through the `vibelink memory` CLI. One shared store, every agent. |
| `vibelink-browser` | Drive a real browser through the `vibelink browser` CLI — the user's own signed-in Chrome, or a VibeLink in-pane page. |
| `worktree-first-development` | Work in a Git worktree of its own instead of the shared checkout, hand off a commit, and integrate one branch at a time. Needs no VibeLink install. |
| `vibelink-skill-creator` | Write, prove, publish, and register a new VibeLink skill without missing a step. |

`vibelink-memory` and `vibelink-browser` need VibeLink running, because they call its CLI.
`worktree-first-development` is plain Git and works in any repository.

## Install

Pick the agents you actually use. Never install to every agent at once — that creates
config directories for agents you do not have.

```
npx skills add JSCOP/vibelink-skills \
  --skill worktree-first-development \
  --global --agent claude-code --agent codex
```

Valid `--agent` keys are printed by `npx skills add --agent x`. Common ones: `claude-code`,
`codex`, `cursor`, `opencode`, `gemini-cli`, `github-copilot`, `windsurf`, `zed`,
`universal`.

VibeLink can also install these for you from Settings → AI → Skills, where each agent gets
its own switch per skill. Both paths write the same file.

## Update

```
npx skills update <skill-name> --global
```

VibeLink refreshes the copies it installed on its own, so a wording fix published here
reaches those agents without a new VibeLink release.

## What it writes

One file per skill: `<agent home>/skills/<skill-name>/SKILL.md`. It never edits an agent's
config file.

## License

MIT
