---
name: new-fullstack-app
description: Use to scaffold a NEW full-stack web app the way viracancao and catalogo-dora were built — a Next.js 14 App Router + TypeScript + Tailwind app in web/, backed by Supabase (Postgres + Storage + RLS) with versioned migrations, email-allowlist auth, prices-in-cents money, mobile-first PT-BR UI, env-gated integrations, and Vercel deploy via a push-to-main GitHub Action. Driven from a product spec first.
---

# new-fullstack-app

Scaffold a complete new **full-stack web app** and take it to a deployable first
version, following the exact recipe proven by **viracancao** (personalized-music
storefront) and **catalogo-dora** (single-piece thrift catalog).

**The deployment stack is the whole point — get it right first:**
- **Supabase = the database.** Postgres + Storage + RLS. Schema lives as versioned
  SQL migrations in `supabase/`. It is the *only* stateful backend.
- **Vercel = the webapp AND the backend.** One Next.js codebase in `web/`: the
  pages/UI **and** the server logic ship together. There is **no separate backend
  service** — the backend is Next.js **API routes / route handlers** (`app/api/*`)
  plus Server Components and Server Actions, which Vercel runs as **serverless
  functions**. Cron is Vercel Cron (`web/vercel.json`). Deploy is **continuous: every
  push to `main` publishes to production** via a GitHub Action.

So the shape is always: **one Vercel app talking to one Supabase project.** No Docker,
no separate API server, no separate DB host. Drive the work from a written spec first.

```
SPEC → web/ (Next 14: UI + app/api backend) ── Vercel (serverless) ──▶ Supabase (Postgres+Storage+RLS)
                                     └── push to main = CD to production ──┘
```

## When to use
- The user wants a brand-new product web app "like viracancao / catalogo-dora":
  a public site + an admin/operator area, backed by a database, deployed to Vercel.
- Use `new-website` instead for a static SEO tool-site in the sites monorepo, and
  `new-tool` / `add-locale` for changes inside an existing static site.

## Before you start
- Read the two reference apps' `CLAUDE.md` (viracancao, catalogo-dora) — they are the
  house style for stack, money, auth, deploy, and env conventions.
- Read this skill's [`reference/app-recipe.md`](reference/app-recipe.md) (the shared
  stack + directory layout + env matrix) and
  [`reference/app-checklist.md`](reference/app-checklist.md) (the Definition of Done).
- Confirm a **spec** exists (a `specs/` folder like viracancao, or a PRD Issue). If
  not, write the product spec first — what it does, the data model, the money rules,
  who logs into the admin, and the acceptance criteria. No code before the spec.
- **Write code, comments, and names in PT-BR** (variables, functions, commits, UI),
  as in the sibling projects — unless the spec says otherwise.

## Steps

1. **Read the spec.** It defines the niche, the public surface, the admin/operator
   surface, the data model (tables + fields), the money rules, the auth allowlist,
   and the acceptance criteria. Restate the intent in a sentence before scaffolding.

2. **Create the repo skeleton** at `<name>/` (sibling to the other apps):
   ```
   <name>/
     web/              # the Next.js 14 app (App Router)
     supabase/         # config.toml + migrations/ (versioned)
     specs/ or docs/   # the spec(s) + docs/design.md (design source of truth)
     brand/            # logo/favicon assets (optional)
     .github/workflows/deploy.yml
     CLAUDE.md         # agent/dev guide — stack, money, auth, deploy, envs
     README.md         # human overview + env table + run commands
     .gitignore        # node_modules, .next, .env*.local, .vercel, supabase/.temp
   ```
   Copy `.gitignore`, the deploy workflow, and `tsconfig`/`next.config` shapes from
   catalogo-dora — it is the smaller, cleaner template of the two.

3. **Scaffold the Next.js app** in `web/` — App Router, **TypeScript `strict`**,
   **Tailwind**, path alias `@/*` → `./*`. Baseline dependencies mirror the sibling
   apps: `next react react-dom @supabase/supabase-js @vercel/analytics
   @vercel/speed-insights` (+ `framer-motion lenis` only if the UX needs motion).
   Scripts: `dev / build / start / lint`. Verify it boots: `cd web && npm run dev`.

4. **Set up Supabase** in `supabase/` (versioned migrations — never edit an applied
   one; always add a new timestamped file):
   - `lib/supabase.ts` → a memoized `getDb()` service-role client, **server-only**,
     that throws if `NEXT_PUBLIC_SUPABASE_URL` / `SUPABASE_SERVICE_ROLE_KEY` are
     missing. Never import it into a client component.
   - First migration: the core table(s) with **RLS enabled** — public reads scoped by
     an explicit visibility predicate, all writes service-role only. Repeat the same
     visibility filter in the server query (`lib/<entity>.ts`), don't rely on RLS
     alone. **Store money as integer cents**; format only at the edge (`lib/format.ts`).

5. **Build the public surface** (Server Components for reads): the listing/home page,
   the detail page, and the primary call-to-action (usually a **WhatsApp** contact/
   order link built from `NEXT_PUBLIC_WHATSAPP_NUMBER`). Mobile-first — most traffic
   is phones. **Load the `frontend-design` skill BEFORE building any UI** so it reads
   as intentional design, not a template. Respect `prefers-reduced-motion`.

6. **Build the admin/operator surface** behind email-allowlist auth:
   - Login = an email in `ADMIN_EMAILS` (or `OPERATOR_EMAILS`) **and** the shared
     `ADMIN_PASSWORD` (catalogo-dora), or a magic-link restricted to the allowlist
     (viracancao). Session = a stateless cookie signed with HMAC-SHA256 / `AUTH_SECRET`.
   - `middleware.ts` protects `/admin/**`. **Every write goes through `/api/*` and
     requires a valid session** (`lib/auth.ts` → verify session; `lib/guard.ts`).
   - Uploads (if any) go through `/api/upload`; compress images client-side before
     upload (canvas, ~1600px, JPEG ~0.82) so it works on mobile data.

7. **Wire integrations env-gated.** Every integration **no-ops when its env var is
   unset** and never breaks the core flow (payments, notifications via Telegram/
   WhatsApp/Resend, analytics, cron). Read integration state from env only — **no
   secret literals in source or CI**. Decide price/amount on the **server**, never
   trust the client. Cron endpoints (`/api/cron/*`) are declared in `web/vercel.json`
   and guarded by `CRON_SECRET`.

8. **Wire SEO + observability basics.** `app/sitemap.ts`, `app/robots.ts`,
   `app/manifest.ts`, per-entity OpenGraph image where it matters, and
   `@vercel/analytics` + `@vercel/speed-insights` in the root layout. Add a durable
   error log (a Supabase `app_logs` table + a fire-and-forget `lib/logger.ts`) if the
   app has real payment/auth flows, as viracancao does.

9. **Write `CLAUDE.md` and `README.md`.** `CLAUDE.md` is the agent/dev guide: what it
   is, the people/allowlist, the stack + conventions, the data model table, the
   business rules that must not break, the auth flow, and the **full env matrix with
   what each var is for**. `README.md` is the human overview + env table + run
   commands. Keep both in the repo root, PT-BR.

10. **Set up continuous deploy to Vercel.** `.github/workflows/deploy.yml`: on push to
    `main` (and `workflow_dispatch`), `working-directory: web`, run
    `vercel deploy --prod --yes --token="$VERCEL_TOKEN"` (build runs on Vercel infra,
    where the env vars live). Store `VERCEL_ORG_ID` / `VERCEL_PROJECT_ID` /
    `VERCEL_TOKEN` as **GitHub secrets**; set the app env vars in the Vercel project.
    ⚠️ Any `NEXT_PUBLIC_*` that is "sensitive" gets masked to empty by `vercel pull` —
    set those with `--no-sensitive` (see viracancao's CLAUDE.md for the trap).

11. **Verify against the checklist.** Walk [`reference/app-checklist.md`](reference/app-checklist.md):
    `npm run build` green (runs typecheck), `npm run lint`, all UX states
    (loading / empty / error / loaded / unauthorized) on every data surface, RLS +
    server filter agree, money in cents, secrets absent from source, mobile layout
    first. Apply migrations to a real Supabase project: `supabase link --project-ref
    <ref> && supabase db push`.

12. **Open the PR** on `feat/new-app-<name>` (linking the spec), summarizing what
    shipped and how it was verified. **Never push to `main` or deploy without explicit
    human go-ahead** — the push-to-main CD means a merge ships to production.

## Guardrails
- **Spec before code.** No product code until the spec/PRD and its acceptance criteria exist.
- **Never push to `main`. Never deploy** without explicit human go-ahead — main is production.
- **No secrets in source or CI** — env reads only, every integration no-ops when unset.
- **Money is integer cents** end to end; format only at the display edge.
- **Service role stays server-side**; every write is authenticated through `/api/*`.
- **RLS is not optional** — enable it and mirror the visibility filter in the server query.
- **Mobile-first, PT-BR, `frontend-design` before UI** — a desktop-only or templated screen is not done.
