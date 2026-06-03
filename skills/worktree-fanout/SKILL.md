---
name: worktree-fanout
description: Use to orchestrate a whole wave of independent task Issues as parallel agents — compute which tasks have all dependencies merged, create one git worktree per task, run one Developer agent per worktree concurrently, gate the next wave on review+merge of the current one, and clean up worktrees after merge.
---

# worktree-fanout

Drive a **wave** of ready task Issues to merged PRs in parallel without the agents
colliding, then advance wave by wave. Each task gets its **own git worktree** on its
**own branch** with **one Developer agent**. The rule that keeps parallel work correct:
**parallel within a wave, sequential across waves.** Pairs with the `claude-code-kit`
harness (the `orchestrator` + `developer` subagents and the `block-push-to-main` hook).

```
WAVE PLAN → ready tasks (deps merged) → one worktree+agent each → review+merge ALL → next wave
```

## When to use
- `plan-waves` produced a dependency-ordered task plan and two or more tasks are ready.
- You're orchestrating many Issues at once, not implementing a single one
  (`implement-feature` is the single-task flow run *inside* each worktree).

## Before you start
Read `CONSTITUTION.md` (Prime directives, "parallel within a wave, sequential across
waves"), the PRD and its task Issues, and — if present — `docs/worktree-orchestration.md`
for the full rationale. You are the orchestrator: you compute waves and gate merges;
you do **not** write feature code or merge.

## Steps

1. **Compute the ready wave from the dependency graph.** List the open task Issues and
   their `Depends on: #…` links. A task is **ready** iff *every* dependency Issue has
   **merged** into `main`. Wave N = all currently-ready tasks. If a task's deps are
   still open, it is **not** in this wave — no matter how independent it looks.
   ```sh
   gh issue list --label task --state open
   gh issue view <n>     # read each task's "Depends on:" lines
   ```

2. **Sync the base** so every worktree branches from the latest merged state:
   ```sh
   git fetch origin && git switch main && git pull --ff-only
   ```

3. **Create one worktree + branch per ready task.** Keep worktrees as **siblings** of
   the repo (not nested inside it) so build tools and ignores don't collide. Name them
   predictably — `feat/<slug>` for the branch, `<repo>-<slug>` for the path — so a
   glance at the list tells you which agent owns what:
   ```sh
   # for each ready task: issue #N, short kebab <slug>
   git worktree add ../<repo>-auth      -b feat/auth      main
   git worktree add ../<repo>-profile   -b feat/profile   main
   git worktree add ../<repo>-settings  -b feat/settings  main

   git worktree list     # your fleet dashboard
   # /path/to/<repo>            abc1234 [main]
   # /path/to/<repo>-auth       def5678 [feat/auth]
   # /path/to/<repo>-profile    9abcd01 [feat/profile]
   # /path/to/<repo>-settings   2345ef6 [feat/settings]
   ```

4. **Run one Developer agent per worktree, in parallel.** Point one Developer
   agent / `/implement` session at each worktree, scoped to **exactly one** Issue. They
   run concurrently — that is the entire point of the wave. Each follows the
   single-task loop (failing test → minimal change → green gates) and ends at an open
   PR with `Closes #N`. The `block-push-to-main` hook is the backstop that none of them
   can push to `main`.
   ```sh
   # in a separate session per worktree, e.g.:
   cd ../<repo>-auth && claude   # then: /implement #57
   ```

5. **Gate the wave (the sequential part).** The wave is **done** only when **every** PR
   in it is reviewed and **merged** and CI on `main` is green. The merge is the single
   human-only step. Do **not** start the next wave until this one has fully merged —
   even if a later task "looks ready". The graph is the source of truth.
   ```sh
   gh pr list --state open --label wave-1
   gh pr checks <pr>     # confirm gates green before a human merges
   ```

6. **Clean up each merged task's worktree.** Worktrees are disposable; the durable
   artifact is the merged PR. After a task's PR merges:
   ```sh
   git worktree remove ../<repo>-auth   # add --force only for untracked junk you don't want
   git branch -d feat/auth              # local branch now redundant
   git worktree prune                   # clear stale administrative files
   ```

7. **Advance to the next wave.** Resync and recompute:
   ```sh
   git switch main && git pull --ff-only
   ```
   Recompute step 1 — the tasks whose deps are *now* all merged are the next wave —
   and repeat. When every wave has merged, run QA against the PRD's acceptance criteria.

## Guardrails
- **Parallel within a wave, sequential across waves.** Never let a child task begin
  before its parent has merged, even if both look ready. Honoring the graph is what
  keeps concurrent work correct.
- **One concern per worktree/PR** — reviewable on a phone. Split anything bigger before
  creating its tree.
- **Worktrees are siblings, never nested;** one branch, one owner — don't check out the
  same branch in two trees.
- **The orchestrator never writes feature code and never merges.** Agents open PRs; a
  human merges; you do the wave bookkeeping.
- **Keep `main` pristine** — everything lands by PR. Don't hoard stale worktrees.
