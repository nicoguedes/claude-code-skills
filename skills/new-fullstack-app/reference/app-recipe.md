# Full-stack app recipe (viracancao / catalogo-dora)

The shared stack, layout, and env matrix both sibling apps follow. Deviate only when
the spec demands it.

## The deployment stack (the backbone — do not substitute)

```
        ┌─────────────────────────── Vercel ───────────────────────────┐
Browser │  Next.js app (web/)                                          │      ┌── Supabase ──┐
  ─────▶ │   • UI: pages + Server Components              (serverless)  │ ───▶ │  Postgres    │
        │   • BACKEND: app/api/* route handlers + Server Actions       │      │  Storage     │
        │   • Cron: web/vercel.json (Vercel Cron)                      │      │  RLS + Auth  │
        └──────────────────────────────────────────────────────────────┘      └──────────────┘
                    ▲  push to main = deploy to production (GitHub Action)
```

- **Supabase = database only.** Postgres + Storage + RLS. The single source of truth;
  schema is versioned SQL migrations in `supabase/migrations/`. No other datastore.
- **Vercel = webapp + backend, one deploy.** The Next.js app in `web/` contains *both*
  the frontend (pages/UI) and the backend (`app/api/*` route handlers, Server
  Components, Server Actions) — Vercel runs the server code as **serverless functions**.
  There is **no separate backend service, no Docker, no long-running server.**
- **CD is continuous:** every push to `main` builds on Vercel infra and publishes to
  production. Migrations are applied separately with the Supabase CLI.
- One Vercel project ⇄ one Supabase project. That's the entire runtime topology.

## Stack

| Layer | Choice |
|-------|--------|
| Framework | **Next.js 14**, App Router |
| Language | **TypeScript `strict`** |
| Styling | **Tailwind CSS**; design tokens in `tailwind.config.ts` (no loose hex) |
| Path alias | `@/*` → `./*` (in `web/tsconfig.json`) |
| Backend | **Supabase** — Postgres + Storage + **RLS** |
| DB migrations | Versioned SQL in `supabase/migrations/` (timestamped; never edit an applied one) |
| Hosting | **Vercel** (build runs on Vercel infra) |
| CD | GitHub Action on push to `main` → `vercel deploy --prod --yes` |
| Analytics | `@vercel/analytics` + `@vercel/speed-insights` in root layout |
| Cron | Declared in `web/vercel.json`, guarded by `CRON_SECRET` |
| Language of code/UI | **PT-BR** (variables, functions, comments, commits, copy) |

Add `framer-motion` + `lenis` **only** when the UX needs scroll/motion (viracancao
has them; catalogo-dora does not). Add `openai` / payment SDKs / `resend` only when an
integration in the spec needs them.

## Directory layout

```
<name>/
  web/
    app/            # App Router: page.tsx, admin/, api/, sitemap.ts, robots.ts, manifest.ts, layout.tsx, globals.css
    components/     # UI (Server Components for reads; client only where needed)
    lib/            # supabase.ts (getDb), config.ts, format.ts, auth.ts, guard.ts, <entity>.ts, logger.ts…
    middleware.ts   # protects /admin/**
    public/         # icons, og assets
    next.config.mjs # reactStrictMode; images.remotePatterns → <ref>.supabase.co storage
    tailwind.config.ts / postcss.config.mjs / tsconfig.json / vercel.json
    package.json    # scripts: dev / build / start / lint
  supabase/
    config.toml
    migrations/     # <timestamp>_<name>.sql
  specs/ or docs/   # spec(s) + docs/design.md (design source of truth)
  brand/            # logo/favicon (optional)
  .github/workflows/deploy.yml
  CLAUDE.md  README.md  .gitignore
```

## Core conventions (do not break)

- **Money in integer cents** in DB + API (`preco_centavos`); format only at the edge
  (`lib/format.ts`, strip `,00` for round values).
- **Service role only on the server** (`lib/supabase.ts` → `getDb()`); never import
  into a client component; it ignores RLS.
- **RLS on every table** — public reads scoped by an explicit visibility predicate;
  all writes service-role. Repeat the same filter in the server query too.
- **All writes through `/api/*` with a valid session.** Auth = email allowlist +
  password (or magic link), session = HMAC-SHA256 cookie / `AUTH_SECRET`,
  `middleware.ts` guards `/admin/**`.
- **Env-gated integrations** — each no-ops when its var is unset; never break the core
  flow. **No secret literals** in source or CI.
- **Server decides price/amount** — never trust the client (anti-fraud).
- **Mobile-first** — most traffic is phones (ads); `frontend-design` before any UI;
  respect `prefers-reduced-motion`.

## Environment matrix (Vercel project + `web/.env.local`)

| Var | For | Notes |
|-----|-----|-------|
| `NEXT_PUBLIC_SITE_URL` | canonical origin | no trailing slash |
| `NEXT_PUBLIC_SUPABASE_URL` | Supabase project URL | public |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | anon key | public |
| `SUPABASE_SERVICE_ROLE_KEY` | service role | **secret, server-only** |
| `AUTH_SECRET` / `CONTA_SECRET` | sign session cookie | `openssl rand -hex 32` |
| `ADMIN_EMAILS` / `OPERATOR_EMAILS` | allowlist | comma-separated |
| `ADMIN_PASSWORD` | shared admin password | catalogo-dora style |
| `CRON_SECRET` | guard `/api/cron/*` | Vercel sends `Authorization: Bearer` |
| `NEXT_PUBLIC_WHATSAPP_NUMBER` | contact/order CTA | with country code |
| *(optional per spec)* | payments, Telegram/Resend/Zatten notify, OpenAI, Meta pixel/CAPI | all env-gated, no-op when unset |

**GitHub secrets for CD:** `VERCEL_ORG_ID`, `VERCEL_PROJECT_ID`, `VERCEL_TOKEN`
(+ `SUPABASE_ACCESS_TOKEN` for migration automation).

⚠️ **`NEXT_PUBLIC_*` sensitive trap:** the CD's `vercel pull` masks "sensitive" vars to
empty, so a sensitive `NEXT_PUBLIC_*` inlines **blank** into the bundle. Set those with
`--no-sensitive`:
`printf '%s' '<value>' | vercel env add <NAME> production --no-sensitive --force`.

## Deploy workflow (`.github/workflows/deploy.yml`)

Push to `main` (+ `workflow_dispatch`), concurrency-guarded, `working-directory: web`,
`npm i -g vercel@latest` then `vercel deploy --prod --yes --token="$VERCEL_TOKEN"`.
The build runs on Vercel infra (where env vars exist) — send code, don't `--prebuilt`.

Migrations to prod:
```bash
supabase link --project-ref <ref>
supabase db push
# or, no DB password (automation): Management API query endpoint with SUPABASE_ACCESS_TOKEN
```
