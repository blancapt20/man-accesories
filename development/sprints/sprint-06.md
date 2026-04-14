# Sprint 06 — Owner panel + role management

**Parent plan:** [`../SPRINTS.md`](../SPRINTS.md)  
**Authority:** [`SPEC.md`](../../SPEC.md) (§2.3, §3.3, §4.9, §7 impersonation)

**Depends on:** Sprint 01 (invites started), Sprint 05 (admin app patterns).

---

## Sprint goal

**Marcus (Owner)** has a dedicated **`/owner`** experience: **role management** (grant/revoke Admin), **admin invite flow** completion (if not finished in Sprint 01), and **view-as-user** / **view-as-admin** **mode switching** as requested in the sprint plan.

---

## SPEC alignment (important)

**SPEC §2.4 (v1.1)** describes MVP staff access as **extended navigation** + protected routes, **not** user impersonation; **SPEC §7** lists true impersonation as out of scope for MVP.

This sprint’s **“view-as-user” / “view-as-admin”** must be defined with stakeholders:

| Interpretation | Implementation sketch |
|----------------|-------------------------|
| **A — Preview (recommended for MVP)** | Same session; toggle only **which nav chrome** or **which route group** is emphasised; **no** JWT as another user; audit optional. Matches SPEC spirit. |
| **B — True impersonation** | Session assumes target user’s claims for support; requires **banner, audit log, time limit, exit**—update **SPEC.md** (remove from §7 for this feature) before building. |

Document the chosen interpretation in **SPEC.md** before marking sprint done.

---

## In scope

- **`/owner`** routes: only `role = owner`.
- **Role management:** search users; set role to `admin` or `user`; guardrails when revoking last admin (SPEC §5.6).
- **Audit log** for role changes (SPEC §4.9).
- **Invite flow:** polish from Sprint 01—Owner sends invite; invitee becomes Admin.
- **View-as-user / view-as-admin:** implement per stakeholder choice **A or B** above.

---

## Out of scope

- Carrier integration (Sprint 07).
- New checkout logic.

---

## Key tasks (suggested)

| # | Task |
|---|------|
| 1 | Owner layout + `/owner` middleware guard. |
| 2 | Role management UI + confirmations. |
| 3 | Wire invites to Resend if not already. |
| 4 | Implement preview or impersonation per decision + SPEC update. |
| 5 | Negative tests: admin cannot hit `/owner`. |

---

## Test layer

| Layer | What to cover |
|--------|----------------|
| **Unit** | Role transition rules (e.g. cannot remove last admin without override path); invite token expiry. |
| **Integration** | API/Server Actions: **owner** can `PATCH` role; **admin** receives **403**; audit row inserted with `actor_id`, `target_user_id`. |
| **E2E** | Owner: grant admin → new admin can open `/admin`; revoke → user blocked from `/admin`. |
| **View-as (A or B)** | **A:** toggle only changes nav chrome—assert same `sub` in JWT before/after. **B:** assert banner visible, session ends on exit, audit contains impersonation start/stop. |
| **Security regression** | Direct `GET /owner` as admin → blocked; CSRF/session rules unchanged on mutations. |

**CI gate (recommended):** integration tests for role mutations + E2E for Owner happy path on staging before release.

---

## Success criteria

Sprint 06 is **done** when all of the following are true:

- [ ] **SPEC §8.2 (Owner):** Owner can **grant** and **revoke** Admin; Admin **cannot** access `/owner`.
- [ ] **Audit:** Every production role change has a corresponding **audit log** entry (**SPEC §4.9**).
- [ ] **SPEC sync:** **SPEC.md** documents the chosen **view-as** behaviour (**A** preview vs **B** impersonation); if **B**, §7/out-of-scope updated accordingly.
- [ ] **View-as:** If **B**, banner + time limit + exit + audit are **verified** in staging; if **A**, stakeholder accepts preview behaviour.
- [ ] **Tests:** CI integration + E2E Owner path green; negative admin→owner tests green.

---

## Handoff to Sprint 07

Order records include fields needed for **carrier_code**, **tracking URL**, and customer-visible status text.
