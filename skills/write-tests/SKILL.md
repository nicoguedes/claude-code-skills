---
name: write-tests
description: Use to backfill the testing pyramid for under-tested code — identify the unit/integration/e2e layers, add fast unit tests for pure logic, integration tests with real collaborators, and a thin e2e for the critical path, naming them by the stack's convention, without changing behavior.
---

# write-tests

Add the missing tests to code that works but isn't covered — build out the **testing
pyramid** so future changes fail loudly instead of silently. This is a
characterization job: you **lock in the current behavior**, you do not change it. Pairs
with `spec-driven-starter`-style apps.

```
UNDER-TESTED CODE → identify layers → unit (broad) + integration (real deps) + thin e2e → PR
```

## When to use
- A module, feature, or path lacks tests and you want a safety net before refactoring,
  fixing, or extending it.
- Use `fix-bug` instead when there's a defect to capture; use `implement-feature` when
  building new behavior. This skill **changes no behavior** — it only adds tests.

## Before you start
Read `CONSTITUTION.md` (the **Testing pyramid** section — many fast unit tests, fewer
integration, a thin e2e), `CLAUDE.md` / `AGENTS.md` (the test runner, where tests live,
and the **naming convention** — match it exactly), and the code under test. Run the
existing suite first so you know the current green baseline.

## Steps

1. **Sync `main` and branch.** Tests are a single concern; never work on `main`:
   ```sh
   git fetch origin && git switch main && git pull --ff-only
   git switch -c test/<slug>          # <slug> = the area you're covering
   ```

2. **Map the code to the three layers.** Read what's there and sort it:
   - **pure logic** (calculations, parsing, formatting, branching) → **unit**.
   - **code that talks to a real collaborator** (DB, queue, cache, HTTP service) →
     **integration**.
   - **the one or two user journeys that must never break** → **e2e**.
   Most of your new tests land at the unit layer; that's the pyramid's wide base.

3. **Add unit tests for the pure logic — the widest layer.** Cover the real branches:
   the happy path, the edges (zero, empty, max, boundary), and the invalid inputs. No
   I/O, no network — milliseconds each. Test behavior through the public surface, not
   private internals, so the tests survive refactors.

4. **Add integration tests for the real collaborators.** Exercise the actual dependency
   rather than a mock where a mock would lie — spin ephemeral dependencies in
   containers (e.g. **testcontainers** or your stack's equivalent) so a real database /
   broker / cache is in the loop and the test is reproducible. Mock only what you can't
   cheaply run for real.

5. **Add a thin e2e for the critical path.** One (maybe two) high-value journeys driven
   the way a user drives them, through a real build (e.g. a browser driver such as
   **Playwright**, or your stack's equivalent). Keep it small and deterministic — cover
   the path that must not break, not every permutation.

6. **Name and place them by this repo's convention.** Put each test where `CLAUDE.md`
   says tests live and match the existing file/case naming exactly (e.g.
   `*.test.ts` / `*_test.py` / `Test*` — whatever this repo uses). Consistency is what
   makes the suite navigable.

7. **Confirm the tests are real, then run the gates.** Each new test must be able to
   **fail** — temporarily break the code (or assert the wrong value) and watch it go
   red, then restore. A test that can't fail is worse than no test. Then run the full
   local gates with this repo's real commands:
   ```sh
   <lint>        # e.g. npm run lint
   <typecheck>   # e.g. npm run typecheck
   <test>        # e.g. npm test   — the whole suite, all green
   ```
   You changed **no production code**; if you found a bug while writing tests, **stop
   and file a bug Issue** (then `fix-bug`) — don't fix it here.

8. **Commit and open the PR** (the branch, never `main`):
   ```sh
   git add -A
   git commit -m "test: cover <area> with unit/integration/e2e"
   git push -u origin test/<slug>
   gh pr create --base main --head test/<slug> \
     --title "test: cover <area>" \
     --body "## What
   Backfills the testing pyramid for <area>; no behavior change.

   ## Layers added
   - unit: <what>
   - integration: <what real collaborator>
   - e2e: <which critical path>

   ## How verified
   Each test confirmed able to fail; full suite green; no production code touched."
   ```

9. **Drive CI green and respond to review,** then **stop** — a human merges.

## Guardrails
- **Change no behavior.** This is characterization; production code stays as-is. A bug
  found here becomes a new Issue, not a sneaky edit.
- **Respect the pyramid's shape** — most tests unit, fewer integration, a thin e2e. A
  suite that's all e2e is slow and flaky.
- **Don't mock what you can cheaply run for real;** prefer ephemeral containers for
  integration tests.
- Every new test must be **able to fail** — verify it before you trust it.
- **Never push to `main`. Never merge** — open the PR; a human merges. No secrets in
  test fixtures.
