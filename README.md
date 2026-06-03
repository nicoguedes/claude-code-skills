# claude-code-skills

A curated, forkable collection of reusable **Claude Code skills** for spec-driven,
agent-first development. Each skill is a self-contained procedure a coding agent can
follow end-to-end — from a rough idea, to a written spec, to a wave-planned task
breakdown, to a tested PR that a human merges.

These skills encode one opinion: **agree on the *what* and *why* in writing before
the *how*, ship small single-concern PRs with tests, and keep a human on the merge
button.** They are the canonical home of the skills bundled into the sister repos
below — copy the ones you need into your own repo's `.claude/skills/`.

## What a skill is

A skill is a directory under `skills/` containing a `SKILL.md` with YAML frontmatter:

```
---
name: <kebab-case-name>
description: <one sentence, third-person, describing WHEN to use it>
---
# <Title>
<clear, numbered instructions the agent follows; may reference bundled files>
```

The `description` is what triggers the skill, so it names the situation. A skill may
bundle supporting files (reference docs, templates, scripts) alongside its `SKILL.md`.

## The skills

| Skill | What it does | Pairs with |
|-------|--------------|------------|
| [`specify`](skills/specify/SKILL.md) | Turns a rough product idea into a PRD authored as a GitHub Issue — problem, users, scope, non-goals, UX states, testable acceptance criteria. | [spec-driven-starter](https://github.com/nicoguedes/spec-driven-starter) |
| [`plan-waves`](skills/plan-waves/SKILL.md) | Decomposes a PRD into small, single-concern tasks with an explicit dependency graph grouped into parallel "waves"; files one Issue per task linked to the PRD. | [spec-driven-starter](https://github.com/nicoguedes/spec-driven-starter) |
| [`implement-feature`](skills/implement-feature/SKILL.md) | Picks up a single task Issue, branches `feat/<slug>`, implements it with the right test levels, and opens a PR that `Closes #<n>` — never pushes to main, never merges. | [spec-driven-starter](https://github.com/nicoguedes/spec-driven-starter) |
| [`review-pr`](skills/review-pr/SKILL.md) | Reviews an open PR against the Constitution and the linked Issue's acceptance criteria (scope, test levels, UX states, no secrets); approves or requests changes, never merges unprompted. | [spec-driven-starter](https://github.com/nicoguedes/spec-driven-starter) |
| [`fix-bug`](skills/fix-bug/SKILL.md) | Fixes a reported bug the disciplined way: reproduce it, capture it in a **failing test first**, make the minimal root-cause fix so the test passes, confirm no regressions, and open a small PR that `Closes #<n>` — never pushes to main, never merges. | [spec-driven-starter](https://github.com/nicoguedes/spec-driven-starter) |
| [`write-tests`](skills/write-tests/SKILL.md) | Backfills the testing pyramid for under-tested code: fast unit tests for pure logic, integration tests against real collaborators (e.g. ephemeral containers), a thin e2e for the critical path — named by the stack's convention, changing no behavior. | [spec-driven-starter](https://github.com/nicoguedes/spec-driven-starter) |
| [`new-website`](skills/new-website/SKILL.md) | Scaffolds a new static site in a sites monorepo: `scripts/new-site.mjs`, i18n slug-mapping, SEO (metadata, sitemap, robots, hreflang, OG), env-gated AdSense Auto Ads, dark mode, and Cloudflare Pages deploy wiring — driven from a Site PRD. Bundles a per-site Definition-of-Done checklist. | [spec-driven-sites](https://github.com/nicoguedes/spec-driven-sites) |
| [`new-tool`](skills/new-tool/SKILL.md) | Adds a new tool/page to an existing site: one component under `app/components/tools/`, registered with per-locale slugs in the i18n config, plus localized copy + SEO, wired into the static params. | [spec-driven-sites](https://github.com/nicoguedes/spec-driven-sites) |
| [`add-locale`](skills/add-locale/SKILL.md) | Adds a new language to an existing site: registers the locale in `app/i18n/config.ts`, adds the full translation set + per-locale tool slugs in `dictionaries.ts`, and verifies `generateStaticParams`/sitemap/hreflang and the static export all pick it up. | [spec-driven-sites](https://github.com/nicoguedes/spec-driven-sites) |
| [`worktree-fanout`](skills/worktree-fanout/SKILL.md) | Orchestrates a whole wave of independent task Issues as parallel agents: computes which tasks have all deps merged, creates one `git worktree` + Developer agent per task, gates the next wave on review+merge of the current one, and cleans up worktrees after merge. | [claude-code-kit](https://github.com/nicoguedes/claude-code-kit) |

The first six are the spec-driven **lifecycle** — `specify` → `plan-waves` →
`implement-feature` → `review-pr`, with `fix-bug` and `write-tests` as its
defect-fixing and test-backfilling loops. The next three (`new-website`, `new-tool`,
`add-locale`) are the sites-factory authoring loop. `worktree-fanout` is the
orchestration layer that fans a wave of task Issues out across parallel worktrees.

## Installing a skill

Skills are just folders. Copy the one(s) you want into your repo's
`.claude/skills/` directory:

```sh
# from your project root
mkdir -p .claude/skills
cp -R /path/to/claude-code-skills/skills/implement-feature .claude/skills/
```

Claude Code discovers the skill from its `SKILL.md` frontmatter and offers it when
the situation in the `description` arises. Bundled reference files (e.g.
`new-website/reference/site-checklist.md`) travel with the folder — copy the whole
directory, not just `SKILL.md`.

These skills reference the real file paths and commands of their paired repo
(`CONSTITUTION.md`, `docs/templates/…`, `scripts/new-site.mjs`,
`app/i18n/config.ts`, …), so drop them into that repo and they work as written.

## Sister repos

- [**spec-driven-starter**](https://github.com/nicoguedes/spec-driven-starter) — the spec-driven, agent-first template for a generic application.
- [**spec-driven-sites**](https://github.com/nicoguedes/spec-driven-sites) — the same method as a static-site factory: one codebase, N SEO-driven multilingual tool sites.
- [**claude-code-kit**](https://github.com/nicoguedes/claude-code-kit) — the reusable `.claude/` harness (agents, hooks, commands, settings) that runs the loop.

## License

[MIT](LICENSE) © 2026 Vinícius Guedes
