---
name: new-tool
description: Use to add a new tool/page to an EXISTING site in a spec-driven-sites monorepo — create one component under app/components/tools/, register it with per-locale slugs in the i18n config, add localized copy + SEO content, and wire it into the static params.
---

# new-tool

Add one new tool to an existing site in the **spec-driven-sites** factory. The
architecture is **one component per tool** and **slug-mapping i18n** — there is no
per-language page duplication, so the whole change is: one component, one `ToolDef`,
one dictionary block, one resolver `case`. One tool = one single-concern PR.

```
TASK ISSUE → ToolDef + component + dictionary + resolver case → PR (Closes #<n>)
```

## When to use
- Adding a tool/page to a site that already exists (`apps/<name>/`).
- Use `new-website` instead to scaffold a brand-new site.

## Before you start
Read `CONSTITUTION.md`, `CLAUDE.md` (the i18n slug-mapping rule + static-export
gotchas), and the task Issue. Pick the target app `apps/<name>/`.

## Steps

1. **Branch off `main`.**
   ```sh
   git fetch origin && git switch main && git pull --ff-only
   git switch -c feat/<tool-slug>
   ```

2. **Register the tool** in `apps/<name>/app/i18n/config.ts` — add a `ToolDef` to the
   `tools` array:
   - a stable `id` (keys the dictionary; survives slug changes),
   - the `component` name (PascalCase),
   - the `locales` it ships in,
   - a `slugs` map: the **SEO-facing localized URL per locale** (e.g.
     `{ en: 'word-counter', 'pt-br': 'contador-de-palavras' }`).
   This automatically flows into `getAllStaticParams()` (static params),
   `getToolAlternatePaths(id)` (`hreflang`), and `sitemap.ts`.

3. **Add localized copy + SEO** in `apps/<name>/app/i18n/dictionaries.ts`, keyed by
   locale: the tool's UI strings plus its `seo` block (`metaTitle`,
   `metaDescription`, `keywords`, `jsonLdName`) and **substantial editorial content**
   (heading, paragraphs, use-cases, FAQ) — AdSense needs real content below the
   widget. Use correct accents/diacritics per locale. Add the type interface for the
   tool's copy if needed.

4. **Implement the component** at `apps/<name>/app/components/tools/<Tool>.tsx`. It
   receives `locale` + `t` (its translations) as props and must **not hardcode
   language**. Format numbers/currency/dates with `Intl.*` and the locale's config.
   Handle every UX state distinctly: **empty/initial · working · error/invalid ·
   result** — never a void or a silently-wrong number.

5. **Wire it into the resolver** in `apps/<name>/app/[locale]/[slug]/page.tsx`: import
   the component and add its `case` to the `switch (tool.component)` so the slug
   resolves to the rendered widget.

6. **Verify SEO is complete for the new page:** unique title + description, canonical,
   `hreflang` to every other locale of this tool, `WebApplication` JSON-LD, and a
   `sitemap.ts` entry per `locale × slug` (this is automatic once the `ToolDef` is
   added — confirm it).

7. **Build and check:**
   ```sh
   pnpm --filter <name> build         # static export must pass
   pnpm lint && pnpm -r exec tsc --noEmit
   ```
   Spot-check each new `/<locale>/<slug>` renders, formats per locale, and handles all
   UX states. Run Lighthouse on the new page per locale (target ≥ 90).

8. **Open the PR** on `feat/<tool-slug>` with `Closes #<n>`, summarizing the tool and
   how it was verified (build + Lighthouse). Drive CI green; a human merges.

## Guardrails
- **One tool per PR.** If it grows ("add the tool *and* a locale"), split it.
- **Never push to `main`. Never merge** without explicit human approval.
- **No manual ad slots** — Auto Ads only. **No secrets** — env reads only.
- The page isn't done until all locales, full SEO, and all UX states are handled.
