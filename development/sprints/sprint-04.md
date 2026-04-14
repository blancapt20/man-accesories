# Sprint 04 — Drop day infrastructure

**Parent plan:** [`../SPRINTS.md`](../SPRINTS.md)  
**Authority:** [`SPEC.md`](../../SPEC.md) (§5.2, §6 Upstash, §8.9–10)

**Depends on:** Sprint 03 (working checkout and stock rules).

**Timing:** Ship **before the first limited drop**; not required for initial soft launch without drop traffic.

---

## Sprint goal

Under **spike traffic** (~20k concurrent users per SPEC narrative), the system **queues** access to purchase, applies **rate limits**, returns **clean sold out** states, and uses **Vercel-friendly caching** so catalogue reads do not overwhelm the origin.

---

## In scope

### Upstash (or agreed queue)

- **Admission queue** for “Buy” / checkout session creation: controlled concurrency; users who never get capacity see **sold out** without entering a broken payment state (SPEC §5.2).
- Short-lived **reservations** if used: TTL + reconciliation with Stripe webhooks (align with SPEC §5.1).

### Rate limiting

- Per-IP or per-session limits on hot endpoints (checkout creation, queue join).

### Caching & edge

- **HTTP caching** or **ISR / fetch cache** for catalogue and PDP where safe (invalidate on admin publish from Sprint 05—or document manual revalidate until then).
- Optional **Vercel Edge** middleware for coarse rate limit or geo (Spain-only is product rule—do not over-engineer unless SPEC requires).

### Operations

- **`DROP_MODE`** (or similar) env + runbook: enable before drop, monitor queue depth, disable after (SPEC §8.10).

---

## Out of scope

- New merchandising, new payment methods, admin redesign.

---

## Key tasks (suggested)

| # | Task |
|---|------|
| 1 | Design queue state machine (waiting → admitted → expired / sold out). |
| 2 | Upstash Redis or QStash integration from Next.js Route Handlers / workers. |
| 3 | Wire “Buy” button to enqueue when `DROP_MODE=true`. |
| 4 | Load test or scripted burst against staging; capture metrics. |
| 5 | Cache headers / `revalidateTag` plan for catalogue. |
| 6 | Runbook markdown in repo (`development/drop-day-runbook.md` or similar). |

---

## Test layer

| Layer | What to cover |
|--------|----------------|
| **Unit** | Queue token TTL logic; admission state transitions (waiting → admitted → expired / sold out). |
| **Integration** | With `DROP_MODE=true`, hit “join queue” / checkout creation endpoint from **N parallel workers**; assert max **K** admissions per minute (or per variant) matches design. |
| **Load / stress** | **k6**, Artillery, or Locust against staging: catalogue PDP under cache vs no cache; checkout path under queue; capture p95 latency and error rate. |
| **E2E** | User A admitted completes Stripe test payment; User B sees **sold out** or queue expiry without entering paid state without stock. |
| **Caching** | Assert `Cache-Control` / tag revalidation: PDP updates after admin change when Sprint 05 hook exists (or document manual revalidate until then). |
| **Chaos / negative** | Upstash unavailable → graceful degradation message (no 500 stack trace to user). |

**CI gate (recommended):** unit + integration for queue state machine; **nightly** or **pre-drop** load script (not necessarily every PR if too slow).

---

## Success criteria

Sprint 04 is **done** when all of the following are true:

- [ ] **Fairness:** With `DROP_MODE` on, measured concurrent checkouts are **bounded** vs unbounded stampede (attach graph or numbers from load test).
- [ ] **UX:** Users who lose the race see **sold out** or queue-expired messaging in **ES** and **EN**; no stuck “pay now” without inventory.
- [ ] **Payments:** No **paid** Stripe state without matching stock allocation (document reconciliation + any reservation TTL).
- [ ] **Caching:** Catalogue/PDP strategy documented; hot paths validated under load.
- [ ] **Ops:** **Runbook** reviewed by Marcus or tech lead; includes enable/disable `DROP_MODE` and monitoring checklist (**SPEC §8.10**).
- [ ] **Tests:** Integration proof for queue + at least one load run documented in `development/`.

---

## Handoff to Sprint 05

Admin product updates should trigger **cache revalidation** if catalogue is cached (hook in Sprint 05 when admin CRUD ships).
