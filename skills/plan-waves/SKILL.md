---
name: plan-waves
description: Use after a PRD exists to decompose it into small, single-concern tasks with an explicit dependency graph grouped into parallel "waves," filing one GitHub Issue per task linked back to the PRD.
---

# plan-waves

Take an agreed PRD Issue and produce an executable plan: small, single-concern,
one-PR-sized tasks, organized by their dependency graph into **waves**. Everything
in a wave is independent and runs in parallel; waves run in order. Each task becomes
its own Issue, so implementation can fan out across parallel agents/worktrees safely.

```
PRD (Issue) → TASK BREAKDOWN (waves) → one Issue per task → ready for implement-feature
```

## When to use
- A PRD Issue exists (from `specify`) and is ready to break down.
- You need a parallelizable plan before anyone writes code.

## Before you start
Read `CONSTITUTION.md`, `CLAUDE.md` / `AGENTS.md`, and the PRD Issue itself. Skim
`docs/process/spec-driven-workflow.md` for how this repo groups waves, and
`docs/templates/task-template.md` for the per-task Issue shape.

## Steps

1. **Read the PRD.** Pull it with `gh issue view <prd-number>`. The acceptance
   criteria are the surface you must cover with tasks — every criterion should be
   satisfied by at least one task.

2. **Decompose into single-concern tasks.** Split the work so each task is one PR
   you could describe in a sentence without "and". If you can't, split it further.
   Each task should ship its own test(s).

3. **Draw the dependency graph.** For each task, record what must merge first.
   Then group into waves:
   - **Wave 1** = tasks with no unmet dependencies.
   - **Wave N** = tasks whose dependencies are all in earlier waves.

   Sketch it plainly, e.g.:
   ```
   wave 1   [ data model ]  [ scaffolding ]          ← parallel, no deps
   wave 2   [ feature A ]   [ feature B ]            ← parallel, after wave 1
   wave 3   [ wiring + e2e ]                          ← after A and B
   ```

4. **Post the wave plan** as a comment on the PRD Issue so the breakdown lives
   beside the spec:
   ```sh
   gh issue comment <prd-number> --body-file <plan.md>
   ```

5. **File one Issue per task.** Use `docs/templates/task-template.md` for each.
   Every task Issue states its **acceptance criteria**, its **test level(s)**
   (unit / integration / e2e), `Depends on: #…` for each dependency, and
   `Part of #<prd>` to link back. Label it with its wave if the repo uses wave
   labels.
   ```sh
   gh issue create --title "<task title>" --label "task,wave-1" \
     --body-file <task-body.md>
   ```

6. **Report.** Print the task Issues grouped by wave, with dependencies shown.
   State that **wave 1 is ready to implement now** — point at `implement-feature`
   for a single task, or run the wave in parallel via isolated worktrees.

## Guardrails
- One concern per task/PR — reviewable on a phone.
- Never let a task depend on something in its own wave; if it does, it belongs in a
  later wave.
- Every PRD acceptance criterion must be covered by at least one task.
- Do not write implementation code here — that's `implement-feature`.
