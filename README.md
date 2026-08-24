# VibeLink Skills

Agent skills published by [VibeLink](https://github.com/JSCOP/vibelink-releases).

Each skill is a single `skills/<name>/SKILL.md` document that teaches a coding agent a
contract. Every skill here is named `vibelink-*` so VibeLink's skills stay identifiable
inside an agent home shared by many publishers. Install only the ones you want, for only
the agents you actually use.

Skill documents are written in English so one canonical text serves every agent. Each one
instructs the agent to reply in whatever language the user writes in.

## Skills

| Skill | What it gives an agent | Needs VibeLink | In the app |
| --- | --- | --- | --- |
| `vibelink-memory` | Read and record the durable project facts VibeLink keeps for a workspace, through the `vibelink memory` CLI. One shared store, every agent. | yes | yes |
| `vibelink-browser` | Drive a real browser through the `vibelink browser` CLI — the user's own signed-in Chrome, or a VibeLink in-pane page. | yes | yes |
| `vibelink-worktree-workflow` | Work in a Git worktree of its own instead of the shared checkout, hand off a commit, and integrate one branch at a time. | no | yes |
| `vibelink-eli5` | Explain a topic from zero as a self-contained HTML page: big pictures, very few words. | no | yes |
| `vibelink-skill-creator` | Write, prove, publish, and register a new VibeLink skill without missing a step. For people working on VibeLink itself. | no | no |

"In the app" means VibeLink bundles it and offers it in Settings → AI → Skills.
`vibelink-skill-creator` is deliberately not bundled: it is for contributors to VibeLink,
not for people using it, so it is installed from here on purpose.

## Install

Pick the agents you actually use. Never install to every agent at once — that creates
config directories for agents you do not have.

```
npx skills add JSCOP/vibelink-skills \
  --skill vibelink-worktree-workflow \
  --global --agent claude-code --agent codex
```

Valid `--agent` keys are printed by `npx skills add --agent x`. Common ones: `claude-code`,
`codex`, `cursor`, `opencode`, `gemini-cli`, `github-copilot`, `windsurf`, `zed`,
`universal`.

VibeLink can also install the bundled ones for you from Settings → AI → Skills, where each
agent gets its own switch per skill. Both paths write the same file.

## Update

```
npx skills update <skill-name> --global
```

VibeLink refreshes the copies it installed on its own, so a wording fix published here
reaches those agents without a new VibeLink release.

## What it writes

One file per skill: `<agent home>/skills/<skill-name>/SKILL.md`. It never edits an agent's
config file.

## Credits

`vibelink-eli5` is adapted from the MIT-licensed `eli5` plugin by Thariq Shihipar in
[`anthropics/claude-plugins-community`](https://github.com/anthropics/claude-plugins-community/tree/main/eli5).

## License

MIT
