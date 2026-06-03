---
name: fix-bug
description: Use to fix a reported bug the disciplined way — reproduce it, capture it in a FAILING test first, make the minimal change so that test passes, confirm no regressions, and open a small PR that Closes the bug Issue. Never pushes to main, never merges.
---

# fix-bug

Fix **one** reported bug through the Report → Analyze → Fix → Verify loop. The
defining rule: **the test comes before the fix.** You write a test that fails *because
of the bug*, then make the smallest change that turns it green — so the bug can never
silently come back. Pairs with `spec-driven-starter`-style apps.

```
BUG ISSUE → reproduce → FAILING test → minimal fix → suite green → PR (Closes #<n>) → human merges
```

## When to use
- A bug Issue exists (a defect report, a QA finding, a stack trace) and needs fixing.
- Use `implement-feature` instead when building *new* behavior from a task Issue.

## Before you start
Read `CONSTITUTION.md` (Prime directives, Testing pyramid, Definition of Done),
`CLAUDE.md` / `AGENTS.md` (real install / run / lint / typecheck / test commands), and
the bug Issue itself (`gh issue view <n>`). The reproduction steps in the Issue are
your starting contract.

## Steps

1. **Reproduce it first.** Do not touch code until you can make the bug happen on
   demand. Follow the Issue's steps; capture the exact wrong behavior (the error, the
   wrong value, the broken state). If you can't reproduce, say so on the Issue and ask
   for the missing detail — a fix you can't verify is a guess.

2. **Sync `main` and branch.** Never work on `main`:
   ```sh
   git fetch origin && git switch main && git pull --ff-only
   git switch -c fix/<slug>           # <slug> = short kebab summary of the bug
   ```

3. **Analyze — find the root cause, not the symptom.** Trace from the observed failure
   back to the line that's actually wrong. Resist patching the surface (swallowing the
   error, clamping the output); fix the cause. Note in one sentence *why* it broke — it
   guides where the test belongs and how small the fix can be.

4. **Write a FAILING test that captures the bug.** Pick the level that pins the cause
   per the testing pyramid (`CONSTITUTION.md`):
   - **unit** — the bug is in pure logic (off-by-one, wrong branch, bad parse).
   - **integration** — the bug only shows with a real collaborator (DB, queue, cache).
   - **e2e** — the bug is a broken user journey through a real build.
   Assert the *correct* behavior. Run it and confirm it **fails for the bug's reason**
   (not a typo, not a missing import). A test that passes before your fix proves
   nothing — go back until it genuinely captures the defect.

5. **Make the minimal fix.** Change the least code that turns the failing test green.
   One bug = one concern; if you discover a second, separate defect, file a new Issue
   rather than widening the diff. If the bug touched a UI surface, make sure the
   relevant state — **loading / not-connected / empty / error** — now behaves.

6. **Verify: new test passes, nothing else regressed.** Run the full local gates with
   this repo's real commands (from `CLAUDE.md`):
   ```sh
   <lint>        # e.g. npm run lint
   <typecheck>   # e.g. npm run typecheck
   <test>        # e.g. npm test   — the whole suite, not just your new test
   ```
   The new test is green and every pre-existing test still passes. Never commit a
   secret, token, or key.

7. **Commit and open the PR** (the branch, never `main`):
   ```sh
   git add -A
   git commit -m "fix: <one-line summary of the bug>"
   git push -u origin fix/<slug>
   gh pr create --base main --head fix/<slug> \
     --title "fix: <summary>" \
     --body "Closes #<n>

   ## Bug
   <what was broken; how to reproduce>

   ## Root cause
   <the actual cause, one or two sentences>

   ## Fix
   <the minimal change>

   ## Test
   <the failing test added; that it fails before and passes after; suite still green>"
   ```

8. **Drive CI green and respond to review,** then **stop** — a human merges.

## Guardrails
- **Failing test first.** No bug fix merges without a test that fails before the change
  and passes after. This is the rule that keeps the bug from regressing.
- **Fix the root cause, not the symptom** — and only that. One bug, one small PR.
- **Never push to `main`. Never merge** — open the PR; a human merges.
- **No secrets in the diff.** Env / secret-manager only.
- The PR must close its Issue (`Closes #<n>`).
