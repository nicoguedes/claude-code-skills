---
name: specify
description: Use when turning a rough product idea into a written, agreed spec — authors a PRD as a GitHub Issue (problem, users, scope, non-goals, UX states, acceptance criteria) before any code is written.
---

# specify

Turn a rough idea into a **PRD authored as a GitHub Issue** — the agreed *what* and
*why*, never the *how*. Nothing gets implemented until this spec exists and its
acceptance criteria are testable. Pairs with `spec-driven-starter`-style apps.

```
IDEA → PRD (GitHub Issue) → ready for plan-waves
```

## When to use
- The user describes a feature or product that is more than a one-line fix.
- You want the contract written down and agreed *before* code is generated.

## Before you start
Read the repo's rules so the PRD respects its gates:
- `CONSTITUTION.md` — the non-negotiable rules.
- `CLAUDE.md` / `AGENTS.md` — this repo's stack, commands, and house style.
- `docs/templates/prd-template.md` — the exact section list you must fill.

## Steps

1. **Restate the intent.** In one or two sentences, state the problem, the target
   user, and the obvious non-goals. Ask only the questions you genuinely need
   answered to write a testable spec — do not interrogate.

2. **Draft the PRD from the template.** Copy the structure of
   `docs/templates/prd-template.md` and fill every section:
   - **Problem** — what's broken/missing and why now; quantify the pain if you can.
   - **Users** — who it's for, their context, what success looks like *for them*.
   - **Scope** — the concrete capabilities this will deliver.
   - **Non-goals** — what it explicitly will **not** do (prevents scope creep; as
     important as Scope).
   - **UX states** — for every surface that loads or mutates data, define
     **loading / not-connected (unauthorized) / empty / error / loaded** distinctly.
     A surface that only handles the happy path is not specified.
   - **Data model** and **API / interfaces** — entities, fields, contracts, shapes.
   - **Risks & open questions** — what's uncertain or needs a decision.
   - **Acceptance criteria** — numbered, testable `Given / When / Then` statements.
     Every one must be observable and checkable; never "works well".

3. **File it as an Issue** with the `prd` label:
   ```sh
   gh issue create \
     --title "PRD: <title>" \
     --label prd \
     --body-file <(cat <<'EOF'
   <the filled-in PRD body>
   EOF
   )
   ```
   If a PRD issue form exists (`.github/ISSUE_TEMPLATE/prd.md`), follow its fields.

4. **Report.** Print the PRD Issue URL. State that the next step is `plan-waves`
   to decompose it into a wave-grouped task plan with one Issue per task.

## Guardrails
- The PRD defines *what* and *why* only — no implementation, no code in this skill.
- Specs are living **Issues**, not stale repo files — keep them in GitHub.
- Every acceptance criterion must be observable and checkable.
- All four+ UX states must be specified for any data-touching surface
  (`CONSTITUTION.md` → "UX states").
