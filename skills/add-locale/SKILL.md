---
name: add-locale
description: Use to add a new language to an EXISTING site in a spec-driven-sites monorepo — register the locale in app/i18n/config.ts, add the full translation set plus per-locale tool slugs in dictionaries.ts, confirm generateStaticParams/sitemap/hreflang pick it up, and verify the static export builds with correct alternates.
---

# add-locale

Add one new language to an existing site in the **spec-driven-sites** factory. The
i18n architecture is **locale-driven**: most plumbing — `generateStaticParams`,
`sitemap.ts`, `hreflang` alternates — derives from the `locales` array and each tool's
`slugs` map, so it cascades automatically once you register the locale. The real work
is three files: the locale config, the full dictionary, and a slug per tool. Adding a
locale is one single-concern PR.

```
TASK ISSUE → locales[] + localeConfig + dictionary + per-tool slugs → static export verified → PR (Closes #<n>)
```

## When to use
- An existing site (`apps/<name>/`) should ship in a new language.
- Use `new-tool` to add a page, or `new-website` to scaffold a new site — this skill
  adds a *language* to a site that already exists.

## Before you start
Read `CONSTITUTION.md`, `CLAUDE.md` (the slug-mapping i18n rule + static-export
gotchas), and the task Issue (which BCP-47 locale, and which tools ship in it). Pick
the target app `apps/<name>/`. The canonical example to mirror is
`apps/example/app/i18n/`.

## Steps

1. **Branch off `main`.**
   ```sh
   git fetch origin && git switch main && git pull --ff-only
   git switch -c feat/locale-<code>     # e.g. feat/locale-es
   ```

2. **Register the locale** in `apps/<name>/app/i18n/config.ts`:
   - Add the new code to the `locales` tuple, e.g.
     `export const locales = ['en', 'pt-br', 'es'] as const;` (leave `defaultLocale`
     as-is — the default renders at `/`).
   - Add a `localeConfig` entry with the full `LocaleMeta`: `lang` (BCP-47, e.g.
     `es-ES`), `name` (e.g. `Español`), `shortName` (e.g. `ES`), `ogLocale` (e.g.
     `es_ES`), `currency` (ISO, e.g. `EUR`), and `siteName`. Because `localeConfig` is
     `Record<Locale, LocaleMeta>`, TypeScript will fail to compile until this entry
     exists — that's the guardrail working.

3. **Add a slug per tool for the new locale** in the same `config.ts`. For every
   `ToolDef` that should ship in this language, add the locale to its `locales` array
   **and** an SEO-facing localized slug to its `slugs` map:
   ```ts
   {
     id: 'word-counter',
     component: 'WordCounter',
     locales: ['en', 'pt-br', 'es'],
     slugs: {
       en: 'word-counter',
       'pt-br': 'contador-de-palavras',
       es: 'contador-de-palabras',
     },
   }
   ```
   Translate the slug for search intent in that language — do not reuse the English
   slug. A tool you deliberately don't translate yet simply omits the locale from both
   lists (it just won't render in that language).

4. **Add the full translation set** in `apps/<name>/app/i18n/dictionaries.ts`. The
   `dictionaries` map is `Record<Locale, Dictionary>`, so the new locale needs a
   **complete** `Dictionary` — nav, home, footer, cards, every tool's UI copy, each
   tool's `seo` block (`metaTitle`, `metaDescription`, `keywords`, `jsonLdName`) with
   **substantial editorial content** (heading, paragraphs, use-cases, FAQ — AdSense
   needs real content below the widget), and the policy pages. Mirror the shape of the
   existing locale object exactly and register it:
   ```ts
   const dictionaries: Record<Locale, Dictionary> = { en, 'pt-br': ptbr, es };
   ```
   Use correct accents/diacritics for the language; translate, never machine-paste.
   Missing keys are a type error — let the compiler enforce completeness.

5. **Confirm the derived plumbing picks the locale up** (no edits expected — verify
   it cascaded):
   - `getAllStaticParams()` now emits the new `locale × slug` pairs, so
     `[locale]/[slug]/page.tsx`'s `generateStaticParams` exports those pages.
   - `[locale]/layout.tsx` (and the `about` / `privacy` / `terms` routes) map over
     `locales`, so the new locale's homepage and policy pages are generated.
   - `app/sitemap.ts` iterates `locales` and `tools`, so the new URLs appear in the
     sitemap.
   - `getToolAlternatePaths(id)` / `getHomeAlternatePaths()` now include the locale, so
     `buildAlternates(...)` emits `hreflang` to and from it on every page.
   If any of these read from a hardcoded list instead of `locales`/`tools`, fix it to
   derive — that's a bug.

6. **Build the static export and verify.**
   ```sh
   pnpm --filter <name> build         # → apps/<name>/out  (must pass)
   pnpm lint && pnpm -r exec tsc --noEmit
   ```
   Then check the new locale actually renders:
   - `apps/<name>/out/<code>/` exists with the homepage, each tool's localized slug,
     and the policy pages.
   - On a tool page, `<html lang>` is the new BCP-47 tag, copy is translated, and
     numbers/currency/dates format via `Intl.*` for that locale.
   - The `<link rel="alternate" hreflang=...>` set includes the new locale on the other
     locales' pages **and** the others on the new locale's pages, plus `x-default`.
   - The canonical URL points at `/<code>/<slug>`, and the page appears in
     `out/sitemap.xml`.

7. **Open the PR** on `feat/locale-<code>` with `Closes #<n>`, summarizing the locale
   added, which tools ship in it, and how it was verified (build + rendered pages +
   alternates). Drive CI green; a human merges and deploys.

## Guardrails
- **One locale per PR.** If it grows ("add Spanish *and* a new tool"), split it.
- **Translate completely** — every `Dictionary` key, every tool slug, real editorial
  content. A partial dictionary is a type error; a half-translated page is a broken
  page.
- **Derive, don't duplicate** — the new locale must flow through `generateStaticParams`,
  `sitemap.ts`, and `hreflang` from `locales`/`tools`, not a parallel hardcoded list.
- **No manual ad slots** (Auto Ads only) and **no secrets** — `ca-pub-…`, DSN, and
  origin stay env-gated (`NEXT_PUBLIC_ADSENSE_CLIENT`, `NEXT_PUBLIC_SITE_URL`).
- **Never push to `main`. Never merge or deploy** without explicit human go-ahead.
