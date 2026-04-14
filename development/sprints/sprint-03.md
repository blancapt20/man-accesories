# Sprint 03 — Checkout & payments

**Parent plan:** [`../SPRINTS.md`](../SPRINTS.md)  
**Authority:** [`SPEC.md`](../../SPEC.md) (§3.1 steps 5–8, §4.5–4.8, §5.1, §5.3–5.4, §6, §8.1, §8.4)

**Depends on:** Sprint 02 (cart, catalogue, i18n).

---

## Sprint goal

**Guest checkout** (and logged-in if already built) completes via **Stripe**; orders persist; **stock** decreases **atomically** on confirmed payment; **GDPR cookie consent** is live; **Resend** sends **order confirmation** and **Marcus is emailed on each sale**.

---

## In scope

### Stripe

- Checkout Session (or agreed flow) in **EUR**; Spain shipping/billing fields.
- **Webhook** with signature verification and **idempotency** (no duplicate orders on retry).

### Orders & inventory

- `orders` + `order_items` (SPEC §4.5–4.6); link guest orders by email + optional `user_id`.
- **Stock decrement:** single transaction or `UPDATE … WHERE stock_quantity >= $qty RETURNING` pattern—**no race** that allows oversell at normal concurrency (SPEC §5.1). Document behaviour on failure.

### GDPR

- **Cookie consent** banner before non-essential scripts; store consent (**SPEC §4.8**); privacy/cookie copy linked from footer.

### Resend

- Transactional: **order confirmation** to customer.
- **Owner / Marcus:** email on **each completed sale** (SPEC §8.4).

### UX

- Confirmation / return page after Stripe; clear errors for payment decline and OOS at checkout (SPEC §5.3).

---

## Out of scope

- **Upstash drop queue** (Sprint 04)—this sprint may serialise checkout creation on server for modest traffic only; document limit.
- **Carrier API** (Sprint 07); tracking URL may be null.
- Full **admin** order UI (Sprint 05)—DB or minimal internal view OK for support.

---

## Key tasks (suggested)

| # | Task |
|---|------|
| 1 | Migrations for orders + RLS for customer read of own rows. |
| 2 | Create Checkout Session from cart; handle success/cancel URLs. |
| 3 | Webhook route: paid → insert order + items + **atomic stock** update. |
| 4 | Resend templates + env `RESEND_API_KEY` server-only. |
| 5 | Cookie consent component + `cookie_consents` persistence. |
| 6 | E2E manual test checklist in `development/` or README. |

---

## Test layer

| Layer | What to cover |
|--------|----------------|
| **Unit** | Webhook idempotency key logic; order total calculation from line items; cookie consent state machine if non-trivial. |
| **Integration** | **Stripe CLI** `stripe trigger` (or test mode) → webhook handler: creates **one** order; replay same event → still **one** order. |
| **Stock concurrency** | Two parallel requests (script or k6) attempting to buy **last** unit: assert DB ends with **one** paid order and stock **0** (or one failed checkout). |
| **E2E** | Full path: cart → Checkout Session redirect → Stripe test card → return URL → order visible; assert **Resend** sandbox/API received payloads (mock Resend in CI or use Resend test domain). |
| **GDPR** | E2E or manual: non-essential script does not run until consent; row in `cookie_consents` (or equivalent) after accept. |
| **Security** | Webhook **without** valid signature returns **400**; fuzz invalid body. |
| **Manual checklist** | Document in `development/`: guest vs logged-in, payment decline, OOS at checkout. |

**CI gate (recommended):** lint + typecheck + build; webhook integration test with Stripe test secrets in CI (fork/branch only) or nightly job.

---

## Success criteria

Sprint 03 is **done** when all of the following are true:

- [ ] **SPEC §8.1:** Cart → **Stripe** (test mode) pay → **order + line items** in DB → **customer** receives confirmation email.
- [ ] **SPEC §8.4:** **Marcus** receives a sale notification for each completed payment in staging.
- [ ] **Inventory:** **Atomic** stock decrement; **no oversell** under concurrent “last item” attempt (evidence from concurrency test or SQL audit).
- [ ] **GDPR:** Cookie consent recorded; non-essential scripts gated per **SPEC §5.4**.
- [ ] **Secrets:** No Stripe/Resend secrets in client bundle or `NEXT_PUBLIC_*`.
- [ ] **Tests:** Webhook idempotency proven; at least one automated E2E or signed integration test run attached to release notes.

---

## Handoff to Sprint 04

Feature flag or env placeholder for **drop mode**; checkout entry point identifiable so Sprint 04 can wrap it with a queue.
