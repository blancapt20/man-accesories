# Sprint 07 — Order tracking

**Parent plan:** [`../SPRINTS.md`](../SPRINTS.md)  
**Authority:** [`SPEC.md`](../../SPEC.md) (§3.1 step 9, §5.7, §6 carrier, §8.1)

**Depends on:** Sprint 03 (orders, Resend); Sprint 05 (admin can set tracking if API fails).

---

## Sprint goal

Customers see **live tracking** where the carrier supports it: **carrier API** integration (or first provider), **tracking link** stored on the order, link included in **Resend** emails where appropriate, and a **customer order status** page (timeline + external tracking CTA).

---

## In scope

### Carrier

- Choose provider (SPEC Appendix B—Correos, SEUR, DHL Spain, etc.).
- **Adapter interface** in code (`lib/carriers/` or similar) so providers can be swapped.
- Fetch or compose **tracking URL**; handle **API failure** (SPEC §5.7): retry, fallback to manual URL set in admin from Sprint 05.

### Email

- Update Resend templates: order shipped / tracking available events if not already sent in Sprint 03.

### Customer UI

- **Order status page:** order number, line items summary, status steps, **Open tracking** button when URL exists.
- Logged-in: list orders; guest: magic link or order lookup by email+order number (decision in SPEC Appendix B—document).

---

## Out of scope

- Multi-carrier selection at checkout (single integration MVP OK).
- Push notifications.

---

## Key tasks (suggested)

| # | Task |
|---|------|
| 1 | Carrier client + env vars (`CARRIER_*`) in `.env.example`. |
| 2 | Webhook or admin action: on ship, call carrier, persist URL. |
| 3 | Resend template variables for `tracking_url`. |
| 4 | Customer routes `/orders/...` i18n. |
| 5 | Admin: manual override field tested when API down. |

---

## Test layer

| Layer | What to cover |
|--------|----------------|
| **Unit** | Carrier adapter: given fixture tracking number → correct URL; malformed API response → typed error. |
| **Integration** | Mock HTTP carrier API: success path persists `carrier_tracking_url`; 5xx/timeout → retry then fallback; admin **manual URL** overrides API. |
| **E2E** | Order marked shipped → customer opens order page → **Open tracking** opens carrier URL in new tab; Resend webhook or log assert template received `tracking_url`. |
| **i18n** | Order status page strings in **ES** and **EN**; guest lookup flow if implemented. |
| **Contract** | Snapshot or schema test for carrier response mapping (protect against API field renames). |

**CI gate (recommended):** unit + adapter integration with mocks on every PR; optional E2E against carrier **sandbox** if provider offers one.

---

## Success criteria

Sprint 07 is **done** when all of the following are true:

- [ ] **SPEC §8.1:** Customer can open **order status** page and reach **carrier tracking** when URL is populated.
- [ ] **Email:** Shipped / tracking-available communication includes a **working** tracking link (verified in Resend test inbox or provider logs).
- [ ] **Resilience:** Carrier API failure shows **graceful** customer messaging and **admin manual** path works (**SPEC §5.7**).
- [ ] **Guest / user:** Documented lookup path (logged-in list or email+order#) works in staging.
- [ ] **Tests:** Adapter unit tests + mocked integration green; E2E or scripted smoke for full ship→notify path signed off.

---

## MVP sign-off

Walk **SPEC §8** checklist end-to-end across Sprints **1–7**; close Appendix B items or defer with dates.
