---
name: new-website
description: Use to scaffold a NEW static site inside a spec-driven-sites monorepo — runs scripts/new-site.mjs, sets up i18n slug-mapping, SEO (metadata, sitemap, robots, hreflang, OG), env-gated AdSense Auto Ads, dark mode, and the Cloudflare Pages deploy wiring, driven from a "new site" PRD/Issue first.
---

# new-website

Scaffold a complete new static tool-site inside a **spec-driven-sites** monorepo and
take it to "Definition of Done" for its launch locales. The site factory is one
Next.js codebase (`output: 'export'`) with **one component per tool** and
**slug-mapping i18n** — no per-language page duplication. Always drive the work from
a Site PRD Issue first.

```
SITE PRD (Issue) → pnpm new-site <name> → i18n + SEO + ads + deploy → PR (Closes #<n>)
```

## When to use
- The user wants a brand-new site in the sites monorepo (not a tool on an existing site).
- Use `new-tool` instead when adding a page to a site that already exists.

## Before you start
Read `CONSTITUTION.md`, `CLAUDE.md` (the i18n slug-mapping rule and static-export
gotchas live there), `docs/templates/site-prd-template.md`, and this skill's
[`reference/site-checklist.md`](reference/site-checklist.md) (the per-site Definition
of Done). Confirm a Site PRD Issue exists (label `site-prd`); if not, write one first.

## Steps

1. **Read the Site PRD.** It defines the niche, target locales (chosen by CPM/volume),
   the tool set, the slug map, SEO/keyword intent, monetization, non-goals, and
   acceptance criteria. `gh issue view <site-prd>`.

2. **Branch off `main`.**
   ```sh
   git fetch origin && git switch main && git pull --ff-only
   git switch -c feat/new-site-<name>
   ```

3. **Scaffold the app** from the example template:
   ```sh
   pnpm new-site <name>     # copies apps/example → apps/<name>, names it @sites/<name>
   pnpm install             # link the new workspace package
   pnpm --filter <name> dev # http://localhost:3000 — sanity-check it boots
   ```

4. **Set up i18n + the slug map** in `apps/<name>/app/i18n/config.ts`:
   - Set `locales` to the PRD's launch locales (default locale renders at `/`).
   - Fill `localeConfig` per locale: `lang` (BCP-47), `name`, `shortName`,
     `ogLocale`, `currency`, `siteName`.
   - Declare each tool as a `ToolDef`: stable `id`, `component` name, the `locales`
     it ships in, and a `slugs` map — the **SEO-facing localized URL per locale**.
   `getAllStaticParams()` expands every `locale × slug` for static export;
   `getToolAlternatePaths(id)` feeds `hreflang`.

5. **Add copy + SEO + editorial content** in `apps/<name>/app/i18n/dictionaries.ts`,
   keyed by locale: per-tool `metaTitle`, `metaDescription`, `keywords`, `jsonLdName`,
   and substantial editorial content (heading, paragraphs, use-cases, FAQ) — AdSense
   needs real content below the widget. Use correct accents/diacritics per locale.

6. **Implement each tool component** under `apps/<name>/app/components/tools/<Tool>.tsx`
   (one per tool) and add its `case` to the resolver switch in
   `apps/<name>/app/[locale]/[slug]/page.tsx`. Each component receives `locale` + `t`
   and must not hardcode language; format numbers/currency/dates with `Intl.*`.
   Handle all UX states: **empty/initial · working · error/invalid · result**.

7. **Wire SEO end to end.** `generateMetadata` in `[locale]/[slug]/page.tsx` already
   emits title/description/keywords, canonical via `absoluteUrl`, `hreflang`
   alternates via `buildAlternates(getToolAlternatePaths(id))`, OpenGraph, and
   `WebApplication` JSON-LD — confirm they resolve for your tools. Confirm
   `app/sitemap.ts` covers every `locale × slug` plus homepages and policy pages,
   and `app/robots.ts` points at the sitemap. Replace `public/og-image.svg`,
   `public/logo.svg`, `public/favicon.svg` for the brand (logo/favicon use
   `currentColor` for dark-mode).

8. **Localize the policy pages** (`[locale]/about`, `[locale]/privacy`,
   `[locale]/terms`) — AdSense approval requires them, linked in the footer.

9. **Confirm AdSense Auto Ads + dark mode + env wiring** (do **not** add manual ad
   slots — Auto Ads place ads themselves; manual `<ins>` causes CLS):
   - AdSense is env-gated via `NEXT_PUBLIC_ADSENSE_CLIENT` (format
     `ca-pub-XXXXXXXXXXXXXXXX`) and **no-ops when unset** — never hardcode an id.
   - `NEXT_PUBLIC_SITE_URL` is the canonical origin for absolute URLs (no trailing
     slash). `NEXT_PUBLIC_ERROR_DSN` is optional and also env-only.
   - The dark-mode no-flash `<script>` is already injected in the root layout; verify
     no FOUC.

10. **Build the static export and verify:**
    ```sh
    pnpm --filter <name> build         # → apps/<name>/out
    pnpm lint && pnpm -r exec tsc --noEmit
    ```
    Walk [`reference/site-checklist.md`](reference/site-checklist.md): all locales,
    SEO, all UX states, Lighthouse ≥ 90 on home + one tool page per locale.

11. **Prepare the deploy wiring (Cloudflare Pages).** The reference target is
    Cloudflare Pages free tier. Document/confirm:
    ```sh
    npx wrangler pages deploy apps/<name>/out --project-name <name> --branch main --commit-dirty=true
    ```
    In CI, store `NEXT_PUBLIC_ADSENSE_CLIENT` and `NEXT_PUBLIC_SITE_URL` as repository
    **variables** and the Cloudflare token as a **secret** — never in YAML. Do not
    deploy from this skill; a human ships after merge.

12. **Open the PR** on `feat/new-site-<name>` with `Closes #<site-prd>`, summarizing
    what shipped and how it was verified (build + checklist + Lighthouse). Drive CI
    green; a human merges and deploys.

## Guardrails
- **Never push to `main`. Never merge or deploy** without explicit human go-ahead.
- **No secrets** — no real `ca-pub-…`, DSN, or token in source or CI; env reads only,
  no-op when unset.
- **No manual ad slots** — Auto Ads only.
- A site is not done until the full kit ships for every launch locale (see the
  checklist) — half a site is a broken site.
