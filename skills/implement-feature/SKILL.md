---
name: implement-feature
description: Use to pick up a single task Issue and implement it end-to-end — branch feat/<slug>, write the change with the right test levels, keep it one concern, and open a PR that Closes the Issue. Never pushes to main, never merges.
---

# implement-feature

Implement **exactly one** task Issue: branch, code, test, and open a PR that closes
the Issue. The Issue *is* the spec — build what it specifies, nothing more. Pairs
with `spec-driven-starter`-style apps.

```
TASK ISSUE → feat/<slug> branch → change + tests → PR (Closes #<n>) → human merges
```

## When to use
- A single task Issue is ready (its `Depends on:` Issues have all merged into `main`).
- You're implementing one concern, not a wave.

## Before you start
Read `CONSTITUTION.md`, `CLAUDE.md` / `AGENTS.md`, and the task Issue
(`gh issue view <n>`). Note the project's real install / run / lint / typecheck /
test commands from `CLAUDE.md` — they must match the gates in
`.github/workflows/ci.yml`.

## Steps

1. **Confirm readiness.** Verify every `Depends on:` Issue has merged. If a
   dependency is still open, stop — this task isn't ready.

2. **Sync `main` and branch.** Never work on `main`:
   ```sh
   git fetch origin && git switch main && git pull --ff-only
   git switch -c feat/<slug>          # <slug> = short kebab summary of the task
   ```
   For parallel work, do this inside a dedicated worktree
   (`git worktree add ../wt-<slug> -b feat/<slug> main`).

3. **Write a failing test first.** Pick the level(s) the change actually needs,
   per the testing pyramid (`CONSTITUTION.md`):
   - **unit** — pure logic in isolation (the widest layer).
   - **integration** — real collaborators (DB, queue, cache) where mocks would lie.
   - **e2e** — a thin layer of real user journeys through a real build.
   The test must fail before your change and pass after. A behavior change without
   a test does not merge.

4. **Implement the one concern.** Make the smallest change that satisfies the
   Issue's acceptance criteria. If scope grows beyond one sentence, **stop and file
   a new Issue** rather than expanding the diff. On any UI surface, handle all four
   states — **loading / not-connected / empty / error** — not just the happy path.

5. **Make the local gates green** using this repo's real commands (from `CLAUDE.md`),
   e.g.:
   ```sh
   <lint>        # e.g. npm run lint
   <typecheck>   # e.g. npm run typecheck
   <test>        # e.g. npm test
   ```
   Never commit a secret, token, or key — env/secret-manager only.

6. **Commit and push the branch** (the branch, never `main`):
   ```sh
   git add -A
   git commit -m "feat: <one-line summary>"
   git push -u origin feat/<slug>
   ```

7. **Open the PR.** Single concern, clear title, body that links the Issue and says
   what changed and how it was tested:
   ```sh
   gh pr create --base main --head feat/<slug> \
     --title "feat: <summary>" \
     --body "Closes #<n>

   ## What
   <one paragraph>

   ## How it's tested
   <unit/integration/e2e added; how to verify>"
   ```

8. **Drive CI green and respond to review.** Push fixups to the same branch until
   CI passes and the reviewer is satisfied. Then **stop** — a human performs the merge.

## Guardrails
- **Never push to `main`. Never merge.** Open the PR; a human merges. This is the
  one irreducible human-in-the-loop step.
- **One concern per PR.** Split anything you can't describe without "and".
- **Every behavior change ships a test** that fails before and passes after.
- **No secrets in the diff** — not in code, config, or CI.
- The PR must close its Issue (`Closes #<n>`).
