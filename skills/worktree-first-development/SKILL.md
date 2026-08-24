---
name: worktree-first-development
description: >-
  Use when a task is about to change files tracked by Git, before the first
  edit, and when finished work has to reach the base branch. Especially when
  another agent, another pane, or the user is working in the same repository,
  when the checkout already has uncommitted changes, when several tasks run in
  parallel, or when a request sounds urgent ("be quick", "just fix it here",
  "small change"). Also use when rebasing a task branch, deciding who may merge,
  or removing a finished worktree. Triggers include "fix this", "add this
  feature", "hand this off", "merge my branch", "another agent is working in
  this repo", "워크트리", "브랜치 따서", "합쳐줘".
---

# Worktree-First Development

Edit in a linked worktree of your own. Hand off a commit. One agent at a time
moves the base branch.

Two agents sharing one checkout share one working tree, one index, and one
`HEAD`. Git does not protect them from each other: it locks refs and each
worktree's own index, and nothing else. Whoever saves a file last wins, and
whoever runs `git add -A` sweeps the other's edits into their commit. A linked
worktree costs one command and removes that entire class of failure.

## The rule

**Before the first change to a Git-tracked file, be in a linked worktree that
belongs to this task.** Not the base checkout. Not another task's worktree.

Urgency does not change this. "Be quick", "it is one line", and "just fix it in
this checkout" are the conditions under which shared-checkout collisions
actually happen, because that is when several agents are working at once.

Not required for: read-only investigation, running status commands, answering
questions, planning, or a directory that is not a Git repository.

## 1. Look before you create

```
git rev-parse --show-toplevel --git-common-dir --abbrev-ref HEAD
git worktree list --porcelain
```

Plus the base branch's SHA (step 2). Everything else — dirty files, an
in-progress merge — announces itself loudly the moment a git command touches it.

Read the repository's own instruction files (`AGENTS.md`, `CLAUDE.md`,
`CONTRIBUTING.md`) for worktree location, branch naming, and approval rules.
Repository and user rules outrank this skill. If they require approval before
creating a worktree, ask and wait.

**A `--git-common-dir` different from `--git-dir` means you are already in a
linked worktree.** Reuse it only if you created it for this task in this session,
or the user named it. Otherwise create your own — never nest a worktree inside a
worktree, and never borrow another task's.

## 2. Choose the base, locally

In order: the repository's declared base branch, then the branch the user named
for this task, then local `main`, then local `master`, then the local branch
that `refs/remotes/origin/HEAD` points at.

If none of those resolves — including when `refs/remotes/origin/HEAD` simply
does not exist locally — the base is ambiguous. Stop and ask. Never query or
mutate the remote to break the tie: no `fetch`, `pull`, `push`, or
`remote set-head` just to start work.

## 3. Create the worktree

```
git worktree add <path>/<task-slug>-<unique-suffix> -b <branch> <base-sha>
```

Branch name follows the repository's convention, else `task/<short-slug>`.
Path follows the repository's convention, else an existing worktree directory,
else a sibling directory outside the repository. Give the directory a
collision-proof suffix (a date, a short id) so that a second agent aiming at the
same task fails loudly on `git worktree add` instead of quietly landing in your
worktree. A worktree directory inside the repository MUST be ignored by Git —
check with `git check-ignore -q <dir>`.

Base it on the base branch's committed SHA. Uncommitted changes in the base
checkout are someone else's in-flight work: they stay exactly where they are.
Report them; never stash, reset, clean, move, or commit them.

Install dependencies and run setup only when the work you are about to do needs
them. There is no automatic full install and no baseline full test suite.

On Windows, keep the worktree path short — a deep worktree plus `node_modules`
or `target` hits the 260-character limit unless `core.longpaths` is on. Treat
path comparisons as case-insensitive when deciding whether two tasks overlap.

## 4. Work and hand off

Write only inside your task worktree. Nothing enforces that — not the
filesystem, not Git, not the build tool — so a write outside it is a rule you
broke, and the honest move is to stop and report it, not to fold it into your
commit.

When the change is done:

1. run a focused check — a **named command** whose result changes if your commit
   is reverted. If no such command exists, or it cannot run in a fresh worktree
   for environmental reasons, say that in the handoff. Never report verification
   you did not run, and never link or copy another checkout's `node_modules`,
   `target`, or `.env` to make a check run;
2. `git add` only the paths this task owns, then commit;
3. report branch, commit hash, the check you ran, and any file another task is
   likely to touch too.

**Then stop.** Do not merge into the base branch, do not push, and do not remove
your worktree. Committing your own work onto the base branch is integrating it —
that is the step where other agents' commits get lost.

Several agents at once: assign files or symbols before they start. That
assignment is advisory and made before anyone has read the code, so it is a
starting guess, not a guarantee. The authoritative overlap set is
`git diff --name-only <base>...<task-branch>` per queued branch; any path in two
of them means those tasks integrate in sequence, each rechecked. An agent that
finds it must touch a path assigned to another task stops and reports instead of
editing it.

## 5. Integration — one agent at a time

Exactly one agent moves the base branch. When several task agents are in flight,
that is a distinct named integration owner, and no task agent runs `merge`,
`push`, or `update-ref` against the base.

**Working alone is the normal case and it is allowed**: the same agent may
integrate its own work, but only after its handoff commit exists and it has
re-read the current base SHA. Being solo removes the second agent, not the
sequence.

Integrate from the checkout where the base branch is checked out, one handoff at
a time:

```
git -C <task-worktree> rebase <base>     # only if the task is behind
git -C <base-checkout> merge --ff-only <task-branch>
```

Rebase runs **inside the task's own worktree**. Running
`git rebase <base> <task-branch>` from the base checkout fails with
`fatal: '<task-branch>' is already used by worktree at ...`, and the flag that
appears to fix that rewrites the other worktree's `HEAD` underneath a live
agent. Do not reach for it; rebase where the branch actually lives, after that
agent has handed off and stopped.

**Any rebase invalidates the handoff proof.** Conflict or not, the tree being
integrated is no longer the tree the task verified — a clean rebase still lands
a renamed symbol next to its old caller. Rerun that task's focused check after
every rebase, and again after its fast-forward, on the integrated base. Failures
then belong to exactly one fast-forward instead of the whole queue.

Resolving a conflict is a content decision Git cannot check. Keep both sides'
intent; a whole-file `--ours`/`--theirs` silently reverts the other task.

What Git does and does not guarantee here:

- **Does:** a single branch-ref update is atomic, so two concurrent integrations
  cannot silently overwrite each other — one fails or stops being a
  fast-forward. `merge --ff-only` also refuses when it would overwrite
  uncommitted changes in that checkout, and unrelated dirty files survive it
  untouched.
- **Does not:** protect working-tree files, judge a conflict resolution, or stop
  two agents from writing in one worktree. Those are excluded by the rules
  above, not by any mechanism.

If the base branch is checked out nowhere, the no-checkout equivalent is
`git update-ref refs/heads/<base> <task-sha> <expected-base-sha>`, which fails
loudly on a mismatch. Never use it against a base that *is* checked out: it
moves the ref and leaves that working tree silently desynchronized.

After the last handoff is in, run the repository's required gate **once** on the
integrated base. Note the SHA you tested; before reporting success, confirm the
base is still that SHA, or the result is stale. If the repository names no such
gate, stop and report the missing decision — same as an ambiguous base.

## 6. Clean up

Only after the commit is integrated and verified, and only if the worktree is
clean:

```
git branch --merged <base>          # the task branch must be listed
git worktree remove <path>
git branch -d <branch>
```

That order is required: `git branch -d` refuses while a worktree still uses the
branch. Check `--merged` explicitly — `git branch -d` judges against the current
`HEAD`, which is not necessarily the base you integrated into.

If `git worktree remove` fails because a process holds a file, stop that process
and retry. If it still fails, leave the worktree and report it.

## Never

Never run a command that discards uncommitted work or rewrites another
worktree's `HEAD`. That includes, and is not limited to: `reset --hard`,
`checkout -- <path>`, `restore`, `switch --discard-changes`, `stash`, `clean`,
`rebase --skip`, `worktree remove --force`, `checkout --ignore-other-worktrees`,
`branch -D`, and any force push. If a state seems to require one, stop and
report instead.

Also never: fetch, pull, or push that the user did not ask for; remove another
agent's worktree; or commit to the base branch as a task agent.

## Rationalizations

| Thought | Reality |
|---|---|
| "They said be quick, a worktree is overhead" | `git worktree add` is one command and seconds. Untangling two agents' interleaved commits is not. |
| "The request named this checkout, so edit here" | Naming the repository is not choosing a checkout. Isolate anyway; the base checkout is the one everyone else is reading. |
| "It is one line in one file" | Collisions are about the shared working tree, index, and HEAD, not the size of your diff. |
| "The tree is clean right now" | Clean at inspection time says nothing about the next thirty seconds while another agent works. |
| "I did not touch their uncommitted files, so it is safe" | You still moved the branch they are based on. That is the change that hurts. |
| "Committing to main *is* the handoff" | A handoff is a commit on your own branch plus a report. Moving the base branch is integration. |
| "I am alone, so the ceremony is pointless" | Solo, you may integrate — after the handoff commit exists and you re-read the base SHA. Skip the branch and you lose the one artifact that makes the work revertible. |
| "The rebase was clean, so the check still holds" | A clean rebase is exactly how a renamed symbol meets its old caller. Rerun the check after every rebase. |
| "Only an import line conflicted, nothing is affected" | You cannot prove that from the diff. Rerun the check instead of self-assessing. |
| "The worktree is done, I will delete it now" | Not until its commit is integrated and verified. Removed early, unmerged work is gone. |

## Red flags

- you are about to edit a tracked file and have not run `git worktree list`
- `git status` in the base checkout shows changes you did not make, and you are
  about to commit
- you are typing `git merge`, `git push`, or `git branch -D` as a task agent
- `--ff-only` failed and you are reaching for a plain `git merge`
- a rebase happened and you are integrating without rerunning the check
- you ran the full gate, then merged something else, and are still reporting the
  old result

Any of these: stop, re-read the current state, and restart the step.

## Quick reference

| Situation | Action |
|---|---|
| About to change a tracked file | Create or confirm this task's linked worktree first |
| `--git-dir` != `--git-common-dir` | Already in a linked worktree — reuse only if it is yours |
| It is another task's worktree | Create your own; do not borrow |
| Base checkout is dirty | Branch from the committed base SHA; leave the changes alone |
| Base branch ambiguous, or `origin/HEAD` missing locally | Stop and ask; never query the remote |
| Work finished | Focused check, commit task-owned paths, report, stop |
| Working alone | You may integrate — after the handoff commit and a fresh base SHA read |
| Task is behind the base | `git -C <task-worktree> rebase <base>`, never from the base checkout |
| Any rebase happened | Rerun that task's focused check before and after the fast-forward |
| Integrating | `merge --ff-only`, one handoff at a time |
| `--ff-only` refused | Base moved, or the merge would overwrite dirty files — re-read state |
| Base checked out nowhere | `git update-ref refs/heads/<base> <new> <expected-old>` |
| Everything integrated | Run the required gate once, confirm the SHA did not move |
| Repository names no gate | Stop and report the missing decision |
| Integrated and verified | `git branch --merged`, `git worktree remove`, `git branch -d` |
| `worktree remove` blocked by a file lock | Stop that process and retry; never force |
