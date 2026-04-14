# Sprint 02 — Customer catalog

**Parent plan:** [`../SPRINTS.md`](../SPRINTS.md)  
**Authority:** [`SPEC.md`](../../SPEC.md) (§1, §3.1 steps 1–4, §4, §5.8, Appendix A)

**Depends on:** Sprint 01 (schema, auth, middleware).

---

## Sprint goal

Customers see a **polished storefront**: **landing**, **catalog**, **product detail**, and **cart**—with **no checkout or payments** yet. This sprint owns **visual and UX quality** for the audience (**men ~20–40**, Spain), **Spanish + English**, and **strong mobile + desktop** responsiveness.

---

## In scope

### Pages & flows

- **Landing:** brand, navigation to catalogue, language switch (ES/EN), footer links to legal placeholders if present.
- **Catalog:** list published products; category filter and search per **SPEC**; EUR price; stock hint (in stock / out).
- **Product detail:** variants, descriptions (both locales), **image placeholders** or Storage public URLs if already available from seed.
- **Cart:** add/update/remove lines; validate quantities against `stock_quantity` on server; show line totals + indicative shipping note optional (exact shipping line item locked with checkout in Sprint 03).

### Design & i18n

- **Tailwind + shadcn/ui**; typography and spacing tuned for the target demographic (**SPEC Appendix A**).
- **i18n** wired for all customer-visible strings in **ES** and **EN** (routing or dictionary approach—stay consistent with Sprint 01).

### Technical

- Public routes only; no Stripe keys required in UI.
- SEO basics (metadata, Open Graph where useful).

---

## Out of scope

- Stripe, order persistence, webhooks, Resend.
- **GDPR cookie consent** (deferred to **Sprint 3** in this plan—do not load non-essential analytics scripts until then).
- Admin/owner panels, drop queue, carrier.

---

## Key tasks (suggested)

| # | Task |
|---|------|
| 1 | Layout shell: header, nav, footer, responsive grid. |
| 2 | Landing route + hero/featured products (from DB). |
| 3 | Catalog listing + filters + empty states. |
| 4 | PDP: variant selector, OOS disabled state, accessible labels. |
| 5 | Cart: persistence (cookie and/or DB for logged-in); server validation. |
| 6 | Locale switcher + translation files / CMS keys. |
| 7 | Visual QA on real devices (iOS + desktop). |

---

## Test layer

| Layer | What to cover |
|--------|----------------|
| **Unit** | Cart line math (totals, quantity clamping helpers); i18n key fallbacks if any. |
| **Integration / API** | Server Actions or Route Handlers: adding cart qty **> stock** returns structured error; PDP returns 404 for unpublished slug. |
| **E2E (Playwright or similar)** | Happy path: landing → catalog → PDP → add to cart → change qty → locale switch preserves cart (if spec’d). |
| **Visual / responsive** | Manual or **Percy/Chromatic** (optional): 375px / 768px / 1280px; no horizontal scroll; focus order on PDP variant controls. |
| **Accessibility** | Automated: **axe** in CI on key routes; fix critical/serious. |
| **i18n** | Snapshot or E2E: same route shows ES vs EN copy for hero + PDP title. |

**CI gate (recommended):** lint + typecheck + build + at least **one** E2E smoke (catalog + add to cart) on PR.

---

## Success criteria

Sprint 02 is **done** when all of the following are true:

- [ ] **Browse & buy path (pre-pay):** User can use **ES** and **EN**, view catalogue and PDP, and manage a **cart** with no errors when stock allows.
- [ ] **Stock honesty:** Server rejects quantity above `stock_quantity` with a **clear, translated** message.
- [ ] **UX bar:** No horizontal scroll on common mobile widths; tap targets and typography fit **SPEC Appendix A** (stakeholder sign-off or checklist).
- [ ] **Nav / roles:** Customer-facing chrome does **not** show admin/owner links (**SPEC §2.4**).
- [ ] **Tests:** E2E smoke green in CI; axe no critical violations on landing + PDP + cart.

---

## Handoff to Sprint 03

Cart representation stable enough to serialize into a **Stripe Checkout Session** (line items, currency EUR, metadata). Decide **guest checkout** and document for Sprint 03.
