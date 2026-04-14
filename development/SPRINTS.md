# Sprint plan — Marcus shop (MVP)

This file is the **high-level breakdown** of how we ship the MVP described in [`SPEC.md`](../SPEC.md). Detailed backlog, tasks, **test layer**, and **success criteria** for each sprint live in [`sprints/`](./sprints/) (`sprint-01.md` … `sprint-07.md`).

**Rules of engagement**

- Scope and behaviour still **defer to `SPEC.md`**; if this document disagrees with SPEC, **update SPEC first** (or record an explicit stakeholder decision there).
- Each sprint should end with **something deployable or demoable** on Vercel (staging at minimum), except Sprint 1 may ship as **APIs + middleware + health route** with **no customer UI** if that is the agreed bar—still deployable.
- “Done” for the overall MVP aligns with **SPEC §8 — Success criteria**, spread across these sprints.

---

## Overview (7 sprints)

| Sprint | Theme | Primary outcome |
|--------|--------|------------------|
| **1** | Foundation | Supabase (schema, RLS), auth with **Owner seed** + **admin invite flow**, Next.js role **middleware**, env vars, repo **structure**—**no product UI** yet. |
| **2** | Customer catalog | **Landing**, **catalog**, **PDP**, **cart**—no checkout. Strong **UX** (men **20–40**, **ES/EN**, **mobile + desktop**). |
| **3** | Checkout & payments | **Stripe**, **guest checkout**, orders, **atomic stock** decrement, **GDPR cookie consent**, **Resend** confirmation (+ owner sale email per SPEC). |
| **4** | Drop day infrastructure | **Upstash** queue, rate limits, **sold out** UX, **Vercel / edge caching** strategy—ship **before first drop**, not required for soft launch. |
| **5** | Admin panel | Admin **login**, **KPI dashboard**, **stock**, **catalog CRUD**, **categories**, **shipping** config, **Excel seed** of Marcus’s data. |
| **6** | Owner panel + roles | Owner **login**, **role management**, **admin invite flow** completion, **view-as-user** / **view-as-admin** switching (reconcile with **SPEC §2.4** if this means true impersonation—see sprint-06 note). |
| **7** | Order tracking | **Carrier API**, tracking in **confirmation email**, **customer order status** page with live link. |

---

## Sprint 1 — Foundation

**Goal:** Backend and app skeleton ready; identities and roles work end-to-end without storefront UI.

**In scope:** Supabase setup, migrations, schema, RLS baseline; **Owner account seeded** (secure process); **invite flow** for admins (magic link or invite table—document); Next.js project with **middleware** for `/admin`, `/owner`; `.env.example`; no landing/catalog/cart UI.

**Out of scope:** Any customer-facing pages beyond a dev health or blank root if required for deploy.

**Detail:** [`sprints/sprint-01.md`](./sprints/sprint-01.md)

---

## Sprint 2 — Customer catalog

**Goal:** Shoppable **front of house** without paying yet—design quality matters here.

**In scope:** Landing, catalog (filters/search per SPEC), PDP with variants, **cart** (persist + validate stock client/server); **Tailwind + shadcn**; **i18n ES/EN**; responsive layouts; image placeholders until Storage in admin sprint.

**Out of scope:** Stripe, webhooks, emails, GDPR banner (unless you want early placement—SPEC ties consent to non-essential scripts; **canonical consent in Sprint 3** per this plan).

**Detail:** [`sprints/sprint-02.md`](./sprints/sprint-02.md)

---

## Sprint 3 — Checkout & payments

**Goal:** Money path is real; inventory and law-of-the-land cookies for launch.

**In scope:** Guest checkout; Stripe Checkout/Payment flow; order + line items; **atomic SQL** (or equivalent) stock decrement on paid webhook; **GDPR cookie consent** + logging; **Resend** order confirmation + **Marcus notification on sale** (SPEC §8.4).

**Out of scope:** Drop queue (Sprint 4), carrier live API (Sprint 7), full admin UI (Sprint 5).

**Detail:** [`sprints/sprint-03.md`](./sprints/sprint-03.md)

---

## Sprint 4 — Drop day infrastructure

**Goal:** First **limited drop** will not melt Postgres or Stripe.

**In scope:** Upstash queue (or Redis) for admission to checkout; rate limiting; **sold out** without zombie payments; caching headers / **edge** strategy on Vercel for catalogue hot paths; feature flag; **runbook** for drop day.

**Out of scope:** New merchandising features—this sprint is **capacity and fairness** only.

**Timing:** Complete **before first drop event**; not required for day-one soft launch without drops.

**Detail:** [`sprints/sprint-04.md`](./sprints/sprint-04.md)

---

## Sprint 5 — Admin panel

**Goal:** Team operates catalog and stock without SQL.

**In scope:** Admin auth path, dashboard KPIs, stock page, product add/remove/edit, category management, **fixed shipping** config, **Supabase Storage** for images, **import/seed from Marcus’s Excel** (one-off tool or documented script).

**Out of scope:** Owner-only role UI (Sprint 6) except any invites started in Sprint 1.

**Detail:** [`sprints/sprint-05.md`](./sprints/sprint-05.md)

---

## Sprint 6 — Owner panel + role management

**Goal:** Marcus controls who is admin; staff can preview other “modes” if product still requires it.

**In scope:** `/owner` routes, grant/revoke admin, invite flow polish, **view-as-user** and **view-as-admin** switching **as specified in stakeholder sprint list**—align implementation with SPEC (extended nav vs true impersonation; update SPEC if impersonation is in).

**Out of scope:** Carrier integration (Sprint 7).

**Detail:** [`sprints/sprint-06.md`](./sprints/sprint-06.md)

---

## Sprint 7 — Order tracking

**Goal:** Post-purchase transparency.

**In scope:** Carrier API adapter; tracking URL on order; include link in **Resend** templates where appropriate; **customer order status** page with timeline + tracking.

**Out of scope:** New payment methods, international carriers.

**Detail:** [`sprints/sprint-07.md`](./sprints/sprint-07.md)

---

## Traceability to SPEC (approximate)

| SPEC area | Sprint(s) |
|-----------|-----------|
| §2 Roles, middleware, RLS | 1, 5–6 |
| §3.1 steps 1–4 (landing → PDP; cart without pay in 2) | 2 |
| §3.1 steps 5–8 (checkout, Stripe, email) | 3 |
| §5.2 Drops, §8.9–10 | 4 |
| §3.2 Admin flows, §4.11 Excel | 5 |
| §3.3 Owner, §2.4 (reconcile view-as) | 6 |
| §3.1 step 9 tracking, §5.7 | 7 |
| §5.4 GDPR cookie consent | **3** (this sprint plan) |
| §8 Success criteria | Verified across **3–7** after features land |

---

## After Sprint 7

Hardening, accessibility, SEO, monitoring, and backlog from **SPEC §7** (out of scope) and **Appendix B** (open decisions).
