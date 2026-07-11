# Full-stack app — Definition of Done

Walk this before opening the PR. An app is not done until every box holds.

## Spec & docs
- [ ] A written spec (`specs/` or PRD Issue) exists and its acceptance criteria are testable.
- [ ] `CLAUDE.md` (agent guide) covers: what it is · people/allowlist · stack + conventions ·
      data model table · business rules that must not break · auth flow · **full env matrix**.
- [ ] `README.md`: human overview · env table · local run commands · migration commands.
- [ ] `docs/design.md` (or a design spec) is the visual source of truth.

## Data & backend
- [ ] Migrations are versioned, timestamped, and none edit an already-applied file.
- [ ] **RLS enabled** on every table; public reads scoped by an explicit visibility predicate.
- [ ] The same visibility filter is repeated in the server query (not RLS-only).
- [ ] `lib/supabase.ts` `getDb()` is service-role, server-only, throws when unconfigured.
- [ ] **Money stored as integer cents**; formatted only at the display edge.
- [ ] Migrations applied to a real project: `supabase db push` succeeds.

## Auth & writes
- [ ] Login = email in the allowlist **and** password (or magic link to the allowlist).
- [ ] Session = stateless HMAC-SHA256 cookie signed with `AUTH_SECRET`.
- [ ] `middleware.ts` protects `/admin/**`.
- [ ] **Every write goes through `/api/*` and requires a valid session.**
- [ ] Uploads (if any) compress client-side before upload; deleting a record deletes its Storage files.

## UX (mobile-first, PT-BR)
- [ ] `frontend-design` was loaded before building UI; screens don't read as templates.
- [ ] Mobile layout designed first, scaled up with `sm:`/`md:`/`lg:`.
- [ ] Every data-touching surface handles **loading / empty / error / loaded / unauthorized**.
- [ ] `prefers-reduced-motion` respected wherever there's motion.
- [ ] Primary CTA (usually WhatsApp) works and is built from env, not hardcoded.

## Integrations & safety
- [ ] Every integration **no-ops when its env var is unset** and never breaks the core flow.
- [ ] **No secret literals** anywhere in source or CI.
- [ ] Price/amount decided on the **server**, never trusted from the client.
- [ ] Cron endpoints declared in `web/vercel.json` and guarded by `CRON_SECRET`.

## SEO & observability
- [ ] `app/sitemap.ts`, `app/robots.ts`, `app/manifest.ts` present and correct.
- [ ] Per-entity OpenGraph image where sharing matters.
- [ ] `@vercel/analytics` + `@vercel/speed-insights` mounted in the root layout.
- [ ] Durable error log (`app_logs` + `lib/logger.ts`) if there are payment/auth flows.

## Build & deploy
- [ ] `cd web && npm run build` is green (runs typecheck).
- [ ] `npm run lint` clean.
- [ ] `.github/workflows/deploy.yml` deploys `web/` to Vercel on push to `main`.
- [ ] Vercel env vars set; sensitive `NEXT_PUBLIC_*` set with `--no-sensitive`.
- [ ] GitHub secrets present: `VERCEL_ORG_ID`, `VERCEL_PROJECT_ID`, `VERCEL_TOKEN`.
- [ ] **Not deployed / merged without explicit human go-ahead** (main = production).
