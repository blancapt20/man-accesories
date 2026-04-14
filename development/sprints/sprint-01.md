# Sprint 01 — Foundation

**Parent plan:** [`../SPRINTS.md`](../SPRINTS.md)  
**Authority:** [`SPEC.md`](../../SPEC.md) (§2, §4, §6, §2.4)

---

## Sprint goal

**Supabase + Next.js spine** is production-ready: schema, **RLS**, **auth**, **Owner seed**, **admin invite flow**, **role middleware**, and **environment / project structure**. **No customer-facing UI** (no landing, catalog, or cart pages—those are Sprint 02).

---

## In scope

### Supabase

- Project wired to Next.js (server client, anon vs service role boundaries per `.cursor/rules/secrets-environment.mdc`).
- Migrations for core tables aligned with **SPEC §4**: `profiles` (with `role`: `user` | `admin` | `owner`), `categories`, `products`, `product_variants`, `shipping_config` (seed row), and any tables needed for **invites** (e.g. `admin_invites`: email, token hash, expires_at, invited_by) if not using Supabase-only invite patterns.
- **RLS** policies: baseline read for published catalogue (if tables exist); users read/update own profile where appropriate; **writes** for catalogue restricted until admin policies land (Sprint 05)—document interim (e.g. service role in CI only).

### Auth & roles

- **Owner seed:** one-time secure procedure (SQL + doc, or Supabase dashboard) to create Owner `profile` + auth user; never commit secrets.
- **Admin invite flow:** e.g. Owner generates invite link or email; invitee signs up or accepts and receives `role = admin`. Document the exact mechanism in `SPEC.md` Appendix B or a short `development/` note when locked.

### Next.js

- App Router project skeleton: `src/app`, `lib/supabase`, middleware for **session refresh** and **route protection** for `/admin/**` and `/owner/**` (return 403 or redirect to sign-in as agreed).
- **TypeScript strict**, `.env.example` listing **all** keys (Supabase URL/anon, service role server-only, placeholders for Stripe/Resend/Upstash used later).

### Deploy

- Vercel project + staging; environment variables configured; optional root route returns **minimal** response (e.g. “API OK” or 404) until Sprint 02.

---

## Out of scope

- Landing, catalog, PDP, cart, Stripe, Resend, Upstash, GDPR UI, carrier, Excel UI.

---

## Key tasks (suggested)

| # | Task |
|---|------|
| 1 | Scaffold Next.js per `.cursor/rules/project-structure.mdc`. |
| 2 | Supabase migrations + RLS v1 + generated types. |
| 3 | Auth helpers (server vs browser client); sign-in/sign-up routes **only if needed** for invite acceptance—can be minimal pages without product chrome. |
| 4 | Owner bootstrap runbook (`development/` or README section). |
| 5 | Admin invite: schema + API route or Server Action + email stub (Resend optional in Sprint 01—logging only OK). |
| 6 | Middleware: roles for `/admin`, `/owner`. |
| 7 | `.env.example` + Vercel env checklist. |

---

## Test layer

| Layer | What to cover |
|--------|----------------|
| **Automated** | `pnpm lint` + `pnpm exec tsc --noEmit` + **production build** in CI; optional **unit tests** for env parsing / role helpers if extracted. |
| **Integration** | Scripted checks against **staging Supabase**: migrations apply cleanly from empty DB; **RLS** negative tests (anon cannot `UPDATE products`; user cannot set `role` via client). Use Supabase SQL editor, `psql`, or `@supabase/supabase-js` in a Node test with **service role** only in CI secrets. |
| **Middleware / routes** | Minimal test: HTTP `GET /admin` and `/owner` **without** session → 302/403 as designed; with **user** JWT → blocked; with **admin** → `/admin` OK, `/owner` blocked; with **owner** → both OK. Can be **Playwright**, **Vitest + Next experimental**, or documented **manual** matrix. |
| **Security smoke** | Grep / bundle analysis: no `service_role` string in `app/**` client boundaries; no keys in `NEXT_PUBLIC_*`. |
| **Manual** | Execute **Owner seed** + **one full admin invite** on staging; capture screenshots or checklist sign-off. |

**CI gate (recommended):** lint + typecheck + `next build` on every PR; migrations dry-run in CI against ephemeral Postgres if feasible.

---

## Success criteria

Sprint 01 is **done** when all of the following are true:

- [ ] **Deploy:** Staging on Vercel boots; no catalogue UI required for sign-off.
- [ ] **Identity:** Owner exists with `role = owner`; at least one **admin** is created only via the **documented invite flow** (not ad-hoc SQL in runbooks as the sole path).
- [ ] **Authorization:** Sessions without staff role cannot access `/admin` or `/owner`; `admin` cannot access `/owner`.
- [ ] **Secrets:** No credentials in git; `service_role` never bundled into client code; `.env.example` is complete for all current and stubbed keys.
- [ ] **Tests:** CI runs lint + typecheck + build green; middleware/route matrix above executed (automated or signed-off manual doc in `development/`).

---

## Handoff to Sprint 02

Catalogue data readable by server components or Route Handlers for upcoming pages; seed data optional SQL for Sprint 02 demos.
