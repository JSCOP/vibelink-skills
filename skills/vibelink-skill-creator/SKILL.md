---
name: vibelink-skill-creator
description: >-
  Use when creating, editing, publishing, or removing a VibeLink agent skill —
  the Markdown documents VibeLink installs into coding agents' own skills
  directories and users switch on in Settings → AI → Skills. Covers proving a
  skill actually changes agent behavior, publishing it to JSCOP/vibelink-skills,
  bundling it in the desktop app, wiring its Settings row and search terms, and
  knowing which changes reach users without a new build. Triggers include "add a
  skill", "new VibeLink skill", "publish this skill", "register the skill in
  Settings", "update the skill wording", "스킬 추가", "스킬 등록", "스킬 만들어".
---

# Creating a VibeLink Skill

A skill is documentation that teaches an agent a contract. The capability itself
lives in the daemon behind the `vibelink` CLI. A skill with no capability behind
it does nothing; a capability with no skill leaves agents guessing syntax.

Three layers, kept separate: the daemon owns the capability, the `vibelink` CLI
is the execution surface, the skill is the teaching.

## Language

Every published skill is written in English, so one canonical text serves every
agent and the frontmatter stays greppable. Each skill also carries a short
`## Language` section telling the agent to answer in the language the USER is
writing in, keeping code, commands, paths, and error strings verbatim. Do not
translate a skill document; do not omit that section.

## Prove it before you ship it

**A skill nobody watched fail is a skill you cannot claim works.** The whole
value is that an agent behaves differently with it than without it, and that is
measurable in about ten minutes.

1. **RED.** Give a fresh subagent a realistic task that tempts the failure this
   skill exists to prevent. Include the pressure that causes it in real life —
   urgency, a teammate, a plausible shortcut. Record what it actually did and
   quote the reasoning it gave.
2. **GREEN.** Same scenario, fresh context, with the skill text supplied as an
   instruction document. Record what changed.
3. **Compare on artifacts, not self-report.** Inspect the end state yourself —
   the repository, the files, the output. An agent's summary of its own
   behavior is a claim, not evidence.
4. **REFACTOR.** A rationalization that survived goes in the skill's own
   rationalization table, in the agent's words. Rerun.

If the RED run does not exhibit the failure, there is nothing to teach — stop
and do not write the skill.

Do this for **one skill at a time**, all the way through publication. Batching
two skills' authoring hides which text caused which behavior change. Desktop
registration for several finished skills MAY be one batched change, because it
changes no skill semantics.

## Name it, and decide whether it ships

**Anything registered in the VibeLink desktop app MUST be named `vibelink-<topic>`.**
The name is the directory, the frontmatter `name:`, the `--skill` argument, and
the key users see in the generated install command — one prefix keeps VibeLink's
skills identifiable in an agent home shared by many publishers. A skill that is
published but never bundled MAY use any name, but prefer the prefix anyway.

Then decide the audience, because it decides the bundle:

| Audience | Publish to the repository | Bundle in the app |
|---|---|---|
| Everyone using VibeLink | yes | yes |
| Only people developing VibeLink itself | yes | **no** |

The app's Settings list is what every end user sees. A skill only a VibeLink
contributor would ever run — this one, for instance — belongs in the repository
so it can be installed deliberately, and NOT in `BUNDLED_SKILLS`, where it would
be permanent noise in a product surface.
## Write the document

Keep it short enough that an agent reads all of it. Aim for what a reader needs
and nothing else.

- `name:` on its own line, exactly matching the directory name. The desktop
  refuses a document whose frontmatter declares anything else.
- `description:` states **when to use it** — triggering conditions, symptoms,
  and trigger phrases including Korean ones. Do NOT summarize the workflow
  there: agents follow a workflow summary in the description instead of reading
  the body.
- Body: the rule first, then the steps, then a rationalization table and red
  flags if this skill enforces discipline under pressure.
- Every command must be one an agent can run as written. Verify it yourself
  first — a command that fails the way the document does not predict is worse
  than no command.

## Publish

Canonical location, in `JSCOP/vibelink-skills`:

```
skills/<skill-name>/SKILL.md
```

Publishing there is what lets a wording fix reach already-installed agents
**without a desktop release**: the app fetches the raw file, validates it,
caches it for six hours, and prefers it over its built-in copy. It is also what
makes the skill legal to name in the `npx skills add` command the app generates
— that command lists only skills the repository is proven to serve.

Update `README.md` in the same commit so the published set is discoverable, and
keep its install examples selective. Never publish a command that installs to
every agent; it creates config directories for agents the user does not have.

Constraints the app enforces on the published document — all four fail closed,
keeping the built-in copy instead:

| Constraint | Limit |
|---|---|
| Path | `skills/<name>/SKILL.md`, name matching the directory |
| Encoding | UTF-8; a BOM is tolerated |
| Frontmatter | starts `---`, has a closing `---`, contains `name: <name>` |
| Size | 256 KiB |

After pushing, confirm the raw URL actually serves it:

```
https://raw.githubusercontent.com/JSCOP/vibelink-skills/main/skills/<name>/SKILL.md
```

An HTML error page is not a skill — the frontmatter check is what stops one from
being written into agent homes.

## Bundle it in the desktop app

Only needed for a skill VibeLink itself should offer in Settings. Copy the
**byte-identical** file to:

```
vibelink-app/src-tauri/resources/skills/<skill-name>/SKILL.md
```

**Byte-identical is not a figure of speech.** The desktop decides "installed" vs
"stale" by SHA-256 against the published text. One CRLF working copy makes every
installed copy permanently stale and rewritten on every refresh. Both
repositories pin `eol=lf` through `.gitattributes` for this reason — check with
`git check-attr eol -- <path>` and compare the two files' SHA-256 before
believing they match.

Then, in `src-tauri/src/app/agent_skills.rs`:

- add a name constant beside the existing ones;
- **append** an entry to `BUNDLED_SKILLS` and widen its fixed array length.
  Append, do not insert: status is zipped positionally against the per-target
  skill list, so reordering silently mislabels which skill is installed;
- bump `VIBELINK_SKILLS_REVISION`. It is written beside each installed skill as
  metadata. Staleness itself is decided by content hash, so the bump does not
  mark unchanged skills stale.

Nothing else in the Rust needs to know. Install, uninstall, refresh, status, and
the generated CLI command all iterate the bundle.

## Wire the Settings row

An unregistered skill still renders — with a robot icon and its raw kebab-case
name. That is the silent defect to avoid.

- `src/components/AgentSkillSettings.tsx` — add a `skillCopy` entry: short
  label, a `lucide-react` icon, and one sentence saying what the user gets.
- `src/components/settings/sections.ts` — add a `settingsSearchEntryDefinitions`
  row under section `skills`, with English and Korean keywords. Korean terms
  belong in `keywords`, never in the rendered label.
- `src/components/SettingsDialog.tsx` — add the same label to the `labels` array
  on the Skills `SettingsSearchTarget`. That array is the scroll anchor: without
  it a search result opens the section but lands nowhere.

Keep the card's hint catalog-neutral. A hint that names each skill has to be
edited every time one is added, and eventually lies.

## Verify

Focused checks, not full suites:

- Rust: the bundle-reporting, per-skill install/uninstall, refresh-installs-
  nothing-missing, and generated-command tests in `agent_skills.rs`. Assert
  behavior for **every** bundle entry rather than a hardcoded count, so the next
  skill does not break unrelated tests.
- Frontend: the skill matrix, per-skill consent, and Settings search tests.
  Their fixtures list skills positionally — extend the fixtures, do not renumber.
- `git check-attr eol` plus a SHA-256 comparison of published and bundled files.
- The raw URL returns 200 with matching frontmatter.
- Watch the per-file line budgets in `scripts/check-quality-ratchets.mjs`; if a
  file legitimately grows past its budget, raise it in the same commit with a
  one-line reason.

Then the native check: one serialized run through the repository's existing DEV
lease. Confirm the new row renders with real copy, its switches are independent
per agent, an agent with no home stays disabled, and Settings search finds the
row by both English and Korean terms.

## What reaches users, and when

| Change | Reaches users |
|---|---|
| Published `SKILL.md` wording | Next refresh of an already-installed copy — no app build |
| New skill added to the bundle | Only after `pnpm tauri:build` and reinstall |
| Settings label, icon, or search terms | Only after rebuild and reinstall |
| A skill installed for an agent | That agent's next session |

Say which of these applies whenever you report the work. A committed skill that
needs a rebuild is not yet live for the user, and claiming otherwise is the
mistake this table exists to prevent.

## Consent rules that must not change

- A newly bundled skill is **off** until the user switches it on for an agent
  that already exists on their machine.
- Auto-update refreshes only skills already installed. It never introduces a new
  skill and never creates an agent home.
- Uninstall removes exactly one skill from exactly one agent.
- If the published copy cannot be fetched or fails validation, the built-in copy
  stays authoritative and nothing visibly breaks.

Anything that would install a skill the user did not ask for is a defect, not a
convenience.

## Rationalizations

| Thought | Reality |
|---|---|
| "The skill reads well, testing it is overkill" | Reading is not using. The RED run is where you find out the wording changes nothing. |
| "I will write both skills, then test both" | Then you cannot attribute a behavior change to either text. One skill, all the way through. |
| "The agent said it followed the skill" | Self-report is a claim. Inspect the repository, files, or output yourself. |
| "Publishing is enough, users have it now" | Publishing updates installed copies. A new skill needs a rebuilt, reinstalled app to appear at all. |
| "Copy-paste is byte-identical" | Not on Windows, where a checkout can rewrite line endings. Compare hashes. |
| "I will insert it in the natural alphabetical spot" | The bundle is zipped positionally against installed state. Append. |
| "Settings will render it anyway" | With a robot icon and a kebab-case name, and invisible to search. |

## Red flags

- writing a skill without having run the scenario that proves the failure exists
- a `description` that summarizes the workflow instead of naming triggers
- publishing without opening the raw URL
- a bundled copy whose SHA-256 differs from the published one
- reporting a new skill as available without saying a rebuild is required
- a test asserting a skill count instead of iterating the bundle
- a bundled skill whose name does not start with `vibelink-`
- bundling a contributor-only skill into the product's Settings list
- a skill document written in anything but English, or missing its `## Language` section
