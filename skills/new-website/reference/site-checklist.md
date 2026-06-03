# Per-site Definition of Done

A site in the spec-driven-sites factory is **Done** only when every box is checked,
for **every launch locale**. Half a site is not a small site — it's a broken one.

## i18n / slug-mapping
- [ ] `app/i18n/config.ts`: `locales` set to the launch locales; default locale renders at `/`.
- [ ] `localeConfig` complete per locale (`lang`, `name`, `shortName`, `ogLocale`, `currency`, `siteName`).
- [ ] Each tool is a `ToolDef` with a stable `id`, `component`, supported `locales`, and an SEO-facing `slugs` map (one localized URL per locale).
- [ ] `app/i18n/dictionaries.ts` has copy + SEO + editorial content keyed by locale, with correct accents/diacritics.
- [ ] One component per tool under `app/components/tools/`; each `case` added to the resolver switch in `app/[locale]/[slug]/page.tsx`.
- [ ] No language is hardcoded in components; numbers/currency/dates use `Intl.*`.

## SEO
- [ ] Every page: unique `title` + meta `description` + `keywords`; canonical URL.
- [ ] `hreflang` alternates to every other locale of the same tool (`getToolAlternatePaths`).
- [ ] `WebApplication` JSON-LD on each tool page.
- [ ] `app/sitemap.ts` covers every `locale × slug` plus homepages and policy pages; `app/robots.ts` points at the sitemap.
- [ ] Substantial, unique editorial content below each widget (use-cases, tips, FAQ).
- [ ] Brand assets replaced: `public/favicon.svg`, `public/logo.svg` (use `currentColor`), `public/og-image.svg` / OG PNG.

## UX states (every tool, every locale)
- [ ] **Empty / initial** — sensible default / hint / example, never a void.
- [ ] **Working** — progress shown, submit disabled, UI not frozen.
- [ ] **Error / invalid input** — plain-language message; never silently wrong.
- [ ] **Result** — formatted for the locale.

## Monetization & policy
- [ ] AdSense Auto Ads env-gated via `NEXT_PUBLIC_ADSENSE_CLIENT` (no-op when unset); **no manual ad slots**.
- [ ] No `ca-pub-…`, DSN, or token in source or CI — env only.
- [ ] Policy pages localized: **About**, **Privacy**, **Terms**, linked in the footer.

## Build, performance & accessibility
- [ ] `pnpm --filter <name> build` (static export) succeeds; `pnpm lint` and `pnpm -r exec tsc --noEmit` pass.
- [ ] Dark mode works with no flash of light theme (no-flash `<script>` in `<head>`).
- [ ] Lighthouse ≥ 90 (Performance / SEO / Best-Practices / Accessibility) on home + one tool page per locale.
- [ ] Core Web Vitals "good" on a mid-tier mobile profile (LCP / CLS / INP).
- [ ] Accessibility basics: one `<h1>` per page, semantic landmarks, every input has a `<label>`, keyboard-reachable with visible focus, WCAG AA contrast, `alt`/`aria-label` on images/SVGs.

## Deploy (human, after merge)
- [ ] `npx wrangler pages deploy apps/<name>/out --project-name <name> --branch main --commit-dirty=true`.
- [ ] CI vars/secrets set out-of-band (AdSense client + site URL as repo **variables**; Cloudflare token as a **secret**).
- [ ] Live domain added in the AdSense dashboard to enable Auto Ads.
