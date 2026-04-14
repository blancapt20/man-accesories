# Sprint 05 — Admin panel

**Parent plan:** [`../SPRINTS.md`](../SPRINTS.md)  
**Authority:** [`SPEC.md`](../../SPEC.md) (§2.2, §3.2, §4, §4.11, §8.2–3, §8.5–6)

**Depends on:** Sprint 01 (roles, middleware); Sprint 03 (orders exist for KPIs). Can start UI in parallel late Sprint 03 if staffed.

---

## Sprint goal

**Admins** run day-to-day ops: **dashboard KPIs**, **stock**, **catalog CRUD**, **categories**, **fixed shipping** price, **product images** (Supabase Storage), and **initial catalogue + stock from Marcus’s Excel** (import script or admin tool + documentation).

---

## In scope

### `/admin` (protected)

- **Login** path for staff (same Supabase Auth; `role = admin` | `owner`).
- **Dashboard:** orders count, revenue, top SKUs, low-stock alerts (metrics from real tables).
- **Stock management:** per-variant quantities; safe updates (optimistic concurrency or `updated_at` check).
- **Products:** create / update / archive; variants; publish flags; **no staff-only fields** on public PDP queries (SPEC §2.4).
- **Categories:** full CRUD; new categories without deploy (SPEC §4.2).
- **Shipping config:** edit fixed EUR shipping (SPEC §4.7).
- **Storage:** upload images; attach to variants; public read for storefront.

### Excel

- **One-time import** or seed pipeline from Marcus’s spreadsheet → `categories`, `products`, `product_variants`, stock (SPEC §4.11, §8.5); go-live checklist in `development/`.

---

## Out of scope

- Owner-only **role management** UI (Sprint 06)—unless invite UI was split; finish Owner flows there.
- Carrier live API (Sprint 07); optional manual tracking URL field if order schema supports it.

---

## Key tasks (suggested)

| # | Task |
|---|------|
| 1 | Admin layout + navigation (Store + Admin). |
| 2 | RLS / server actions: staff write policies audited. |
| 3 | Dashboard widgets wired to SQL or views. |
| 4 | Stock + catalog + category forms (shadcn). |
| 5 | Storage upload + image URL on variants. |
| 6 | Excel parser script + dry-run mode + import log. |

---

## Test layer

| Layer | What to cover |
|--------|----------------|
| **Unit** | Excel row mapper; slugify; money/stock parsing edge cases (empty row, duplicate SKU). |
| **Integration** | Server Actions with **admin JWT**: CRUD succeeds; with **user JWT**: **403** or RLS no-op. Dashboard SQL returns expected aggregates on seeded orders. |
| **Storage** | Upload file → public URL loads in browser; **non-admin** cannot upload (forbidden). |
| **E2E (Playwright)** | Admin login → create category → create product + variant → adjust stock → change shipping → see change on storefront cart/shipping hint if applicable. |
| **Regression** | Public PDP network response: **no** internal keys (cost, draft flags) in JSON for admin browsing as customer (**SPEC §2.4**). |
| **Excel import** | Dry-run produces diff report; real import on **staging** snapshot before/after row counts. |

**CI gate (recommended):** lint + typecheck + build + integration tests for RLS-sensitive mutations (service role only where intended).

---

## Success criteria

Sprint 05 is **done** when all of the following are true:

- [ ] **SPEC §8.3:** Admin manages **stock**, **catalogue**, **categories**, and **fixed shipping** entirely in-app (no raw SQL for daily ops).
- [ ] **SPEC §8.6:** New uploads appear on **public PDP** within expected cache/revalidate window.
- [ ] **SPEC §8.5:** Excel import **documented** and **executed successfully** on staging at least once with log/output archived.
- [ ] **Data isolation:** Storefront responses for an admin browsing as a customer do **not** leak staff-only fields (**SPEC §2.4**).
- [ ] **Tests:** CI green; critical admin E2E path passes; import dry-run is part of release checklist before production import.

---

## Handoff to Sprint 06

List of users with emails for **Owner** to promote; audit log table if not already present for role changes in Sprint 06.
