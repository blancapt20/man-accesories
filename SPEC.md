# Marcus — Men’s Accessories E‑commerce — Master Specification

This document is the single source of truth for scope, behaviour, data, and technical decisions for the MVP and near‑term roadmap. Implementation should trace back to sections here.

---

## 1. What you’re building and why

Marcus runs a men’s accessories business in Spain and needs a professional online shop to sell belts, rings, sunglasses, hats, necklaces, earrings, and future categories (e.g. socks). Today he manages stock in Excel and has no product photos in the system yet; go‑live requires a one‑time migration from that spreadsheet into a proper database, with ongoing stock and catalog management handled in the app by his team. The site must serve **Spanish and English**, **EUR** pricing, **Spain‑only** sales and shipping, **GDPR‑compliant** consent for cookies and marketing where applicable, and **strong mobile and desktop** UX for a primary audience of **men aged roughly 20–40** (clean, confident typography, fast paths to product and checkout, minimal friction). The business expects **~5 orders per day** on average but **limited‑edition drops** that can drive **spikes on the order of ~20k concurrent visitors**; the architecture must protect inventory and checkout under load via **queued purchase processing** rather than letting every “Buy” hit the database at once. Marcus (Owner) needs control over who is Admin; Admins need dashboards, KPIs, and stock/catalog tools; customers need a standard e‑commerce journey through Stripe, email (Resend), and order tracking linked to a **carrier API**. Deployment targets **Vercel** with **Next.js**, **Supabase** (database + auth + product image storage), **Stripe** (payments), **Resend** (transactional email), and **Upstash** (Redis/queue) for drop‑day fairness and overselling prevention.

---

## 2. Roles and permissions

### 2.1 User (customer)

- **Sees:** Public marketing/landing, catalog, product detail, cart, checkout (when authenticated or guest per product rules), legal pages (privacy, cookies, terms), language toggle (ES/EN), GDPR cookie banner/consent UI.
- **Can do:** Browse without account; optional account creation/login; add to cart; complete checkout (Stripe); view own order history and tracking when logged in; receive confirmation and transactional emails.
- **Cannot do:** Access `/admin`, `/owner`, or any staff-only APIs; manage catalog, stock, roles, or other users’ data beyond their own orders/account.

### 2.2 Admin (Marcus’s team)

- **Sees:** The same **public storefront** as customers (catalog, product pages, cart, checkout) **plus** an **extended navigation** (e.g. Store + Admin) and all pages under a **protected `/admin` area** (dashboard / KPIs, stock management, catalog CRUD, shipping configuration as defined).
- **Can do:** Create/update/archive products and variants; adjust stock; manage categories (including creating new categories for future lines); configure fixed shipping price; trigger or view operational reports as specified in MVP; handle order fulfilment states that the spec defines (e.g. packed/shipped) if in scope for MVP; browse the storefront while logged in as themselves (same session as day‑to‑day work—not a separate “customer identity”).
- **Cannot do:** Grant or revoke Admin/Owner roles; change Owner-only settings unless explicitly delegated in a future phase (not MVP unless stated below).

### 2.3 Owner (Marcus only)

- **Sees:** Same as **Admin** for the storefront and `/admin`, **plus** a **protected `/owner` (or equivalent) role-management** UI (promote users to Admin, revoke Admin). Navigation reflects all areas they are allowed to use (Store + Admin + Owner).
- **Can do:** Assign/remove Admin role on selected accounts; all Admin capabilities (Owner is a superset of Admin for MVP).
- **Cannot do:** Nothing above Owner in this product (single tenant, single business).

### 2.4 Implementation note (auth)

**MVP (required):**

- Persist a **role** (`user` | `admin` | `owner`) on the user profile in Supabase (or equivalent claims synced from profile).
- **Navigation:** customers see only storefront/account links; **Admin** and **Owner** see **additional links** to `/admin` (and Owner to `/owner`). Extra menu items are UX only.
- **Route protection:** `/admin/**` (and `/owner/**`) must be gated in Next.js (middleware and/or layout) so only `admin` or `owner` (and only `owner` where applicable) can load those routes.
- **Server-side authorization:** every Route Handler and Server Action that reads or writes sensitive data must **verify the session role**; never rely on hiding UI alone.
- **Supabase RLS:** policies must match roles (e.g. customers read published catalog and their own orders; staff policies for writes and order reads as required). Public product pages must **never** expose staff-only fields (cost, drafts, internal notes) even when the viewer is an admin—keep that data on admin routes or admin-only queries.

---

## 3. User flows

### 3.1 Customer (User)

1. **Landing** — Brand, featured products or drop teaser, navigation to catalog, language switch, cookie consent.
2. **Login (optional)** — Account for order history and faster checkout; guest checkout allowed if MVP policy allows (recommend: guest + optional account).
3. **Catalog** — Filter by category (belts, rings, etc.), sort, search; show price in EUR, stock state (in stock / low / out / drop-only messaging as applicable).
4. **Product detail** — Images from Supabase Storage (placeholders until assets exist), description, variant selector if applicable, add to cart; on **drop days**, “Buy” may enqueue the user.
5. **Cart** — Line items, quantities constrained by available stock or queue outcome, shipping fee (fixed, from config), estimated total.
6. **Checkout** — Shipping/billing for Spain, review, Stripe payment.
7. **Stripe** — Payment succeeds → order confirmed.
8. **Confirmation email** (Resend) — Order summary, link to order detail/tracking.
9. **Order tracking** — Status timeline + **carrier tracking URL** from integrated carrier API when shipment exists.

### 3.2 Admin

1. **Login** — Supabase Auth (email magic link or password per project choice).
2. **Dashboard** — Product KPIs (e.g. revenue, orders, top SKUs, low stock alerts) scoped to MVP metrics you implement from available data.
3. **Stock management** — List SKUs/variants with quantities; adjust stock; sync semantics: single source of truth is DB post go‑live (Excel retired operationally after cutover).
4. **Catalog management** — Add/remove/edit products; assign categories; upload or attach image paths in Supabase Storage; create **new categories** without code deploy.
5. **Storefront** — Use the same public routes as customers (optional link in nav, e.g. “Tienda / Store”) to verify merchandising; staff session stays the same; no impersonation requirement.

### 3.3 Owner

1. **Login** — Same auth provider; role = Owner.
2. **Role management** — Search users; grant Admin; revoke Admin; audit who changed what and when.
3. **Admin + storefront** — Same as Admin: open `/admin` from nav and the public storefront from Store links; all gated by role + RLS as in §2.4.

---

## 4. Data models

Conceptual entities (exact SQL names can follow Supabase conventions). All monetary fields in **integer minor units (cents)** or **numeric with explicit EUR** — pick one and stick to it in implementation.

### 4.1 `profiles` / `users` (extends Supabase `auth.users`)

- `id` (FK to auth)
- `email`, `full_name` (optional)
- `role`: `user` | `admin` | `owner`
- `locale_preference`: `es` | `en` (optional)
- `created_at`, `updated_at`

### 4.2 `categories`

- `id`, `slug`, `name_es`, `name_en`, `sort_order`, `is_active`, `created_at`, `updated_at`
- Supports future categories (socks, etc.) created by Admin in UI

### 4.3 `products`

- `id`, `slug`
- `category_id` (FK)
- `title_es`, `title_en`, `description_es`, `description_en` (rich text or markdown per choice)
- `is_published`, `is_drop_limited` (boolean or enum for drop behaviour)
- `metadata` (JSON): optional tags, materials, size notes
- `created_at`, `updated_at`

### 4.4 `product_variants` (SKUs)

Marcus has **per-SKU price and stock**; even “simple” products should have at least one variant row.

- `id`, `product_id` (FK)
- `sku` (unique), `title_es`, `title_en` (e.g. “Black / 105 cm”)
- `price_eur` (minor units recommended)
- `stock_quantity` (non-negative; oversell protection at application + queue layer)
- `stripe_price_id` (optional cache for Stripe Price object)
- `image_paths` (array or join table; stored in Supabase Storage bucket)
- `is_active`, `created_at`, `updated_at`

### 4.5 `orders`

- `id`, `order_number` (human-readable)
- `user_id` (nullable for guest if supported)
- `email` (always required for guest + receipts)
- `currency` (always `EUR` for MVP)
- `subtotal`, `shipping_amount`, `tax` (if applicable; Spain VAT rules to be confirmed with accountant — model fields anyway)
- `total`
- `status`: `pending_payment` | `paid` | `processing` | `shipped` | `cancelled` | `refunded` (adjust to legal/ops needs)
- `stripe_checkout_session_id` / `stripe_payment_intent_id`
- `shipping_address` (JSON), `billing_address` (JSON)
- `carrier_code`, `carrier_tracking_number`, `carrier_tracking_url` (from carrier API)
- `created_at`, `updated_at`

### 4.6 `order_items`

- `id`, `order_id` (FK), `variant_id` (FK), `quantity`, `unit_price`, `line_total`

### 4.7 `shipping_config` (singleton or versioned)

- `fixed_shipping_price_eur` (editable by Admin)
- `effective_from` (optional versioning so price changes are auditable)
- `updated_by`, `updated_at`

### 4.8 `cookie_consents` (GDPR)

- `id`, `user_id` (nullable), `anonymous_id` (if pre-login)
- `preferences` (JSON: necessary/analytics/marketing flags as implemented)
- `ip_hash` or region (minimize PII), `policy_version`, `created_at`

### 4.9 `audit_logs` (recommended for Owner actions)

- `actor_id`, `action`, `target_user_id`, `metadata`, `created_at`

### 4.10 Queue / drop infrastructure (logical)

- Not necessarily relational tables: **Upstash Redis** keys or **Upstash QStash** / queue patterns for “claim slot → reserve/decrement stock → create Stripe session or PaymentIntent” in **controlled concurrency**.
- Persist enough in DB to reconcile: `reservation_id`, `variant_id`, `expires_at`, `status` if you use soft reservations.

### 4.11 Excel migration (one-time)

- Import script or admin-only tool: maps spreadsheet columns → `categories`, `products`, `product_variants`, initial `stock_quantity`, and placeholder image paths.
- **Cutover:** run import on go‑live day; freeze Excel as source of truth at handover timestamp.

---

## 5. Edge cases

### 5.1 Overselling

- **Risk:** Two customers pay for last unit.
- **Mitigations:** Server-side transactional decrement or **optimistic locking** on `stock_quantity`; combine with **drop queue** so only N checkouts proceed at a time; Stripe webhooks confirm payment before finalising allocation; on payment failure, release reservation.

### 5.2 Drop day queue (~20k traffic)

- **Risk:** DB and Stripe rate limits; thundering herd on “Buy”.
- **Behaviour:** “Buy” enqueues; user sees **position / wait** or **simple spinner** with honest messaging; worker processes queue in **single-flight or small batches**; users who never get stock see **clear “Sold out”** without broken payments.
- **Tech direction:** **Upstash Redis** for counters, locks, or token bucket; optional **Upstash QStash** for job fan-out; integrate cleanly with **Next.js** on Vercel.

### 5.3 Out of stock UX

- Catalog: disable add-to-cart or grey out with explanation.
- Cart: if stock drops while browsing, prompt to update quantities before checkout.
- Checkout: revalidate stock server-side; if invalid, block payment and show actionable message.

### 5.4 GDPR (Spain / EU)

- **Cookie consent** before non-essential scripts (analytics, marketing pixels): granular choices, versioned policy text, log consent.
- **Privacy policy** and **legal basis** for order processing (contract), marketing (consent if used), data retention for orders and logs.
- **Data subject requests:** out of MVP unless required — at minimum document contact process in privacy policy.

### 5.5 Guest vs logged-in orders

- If guest checkout: link order to email only; risk of typos — confirmation email is critical; optional “create account from this order” link.

### 5.6 Owner demotes last Admin

- UI guardrails + confirm dialogs; Owner can always assign new Admin.

### 5.7 Carrier API failures

- If tracking URL cannot be fetched, show manual status updates from Admin and retry later.

### 5.8 Image absence pre-launch

- Supabase Storage buckets and paths configured; UI uses **consistent placeholders** until real assets are uploaded.

### 5.9 Language and content

- Missing translation falls back to other language or shows “translation pending” per product policy (decide once in implementation).

---

## 6. Tech stack and why

| Tool | Responsibility |
|------|----------------|
| **Next.js (App Router)** | Full-stack React: SSR/SSG for SEO and performance, Route Handlers for webhooks (Stripe), Server Actions for mutations, i18n routing or middleware for **ES/EN**. |
| **Vercel** | Hosting, previews, edge-friendly deployment, scales for marketing traffic; pair with queue workers for drop logic. |
| **Supabase** | **Postgres** for relational data; **Auth** for users/roles; **Storage** for product images; **RLS** for data isolation; optional **Edge Functions** if needed alongside Next.js. |
| **Stripe** | Checkout or Payment Intents, EUR, webhooks for authoritative payment state, refunds later if needed. |
| **Resend** | Transactional email: order confirmation, shipping updates, Owner notification **on each sale** (in addition to customer emails if required). |
| **Upstash (Redis / QStash)** | **Drop-day queue**, rate limiting, distributed locks, short-lived reservations; generous free tier; first-class Next.js patterns. |
| **Carrier API** (TBD: Correos, SEUR, DHL Spain, etc.) | Generate **tracking URLs** and surface them in order tracking; abstract behind an interface so the provider can change. |

---

## 7. Out of scope (explicit)

- **International shipping** and multi-currency beyond **EUR** / **Spain-only** fulfilment for MVP.
- **Loyalty points**, referrals, subscriptions, B2B wholesale portal.
- **Socks** (and other future categories) as **merchandising content** is in scope via Admin-created categories; **dedicated sock supply chain features** are not unless Marcus asks later.
- **Mobile native apps** (only responsive web).
- **Advanced ERP** integration (NetSuite, etc.) — Excel is replaced by the app, not by an external ERP in MVP.
- **In-app chat support** — email/contact link is enough unless added later.
- **User impersonation** (“log in as customer” for support) — optional future phase if support needs it; MVP uses extended staff nav + protected admin routes only.
- **Automated VAT invoicing** beyond storing what Stripe/checkout captures — confirm with accountant before building legal invoicing.

---

## 8. Success criteria — MVP “done”

1. **Customer path works end-to-end:** landing → catalog → product → cart → checkout → **Stripe (EUR)** → **Resend** confirmation → order visible with **tracking link** when carrier data exists.
2. **Roles enforced:** User / Admin / Owner permissions as specified; **`/admin` and `/owner` (or equivalent) protected** by middleware/layout + server checks + RLS; **Owner** can grant/revoke Admin; **Admin** cannot manage roles; staff browse the storefront via normal nav (extended menu), not impersonation.
3. **Admin operations:** dashboard with meaningful **product/order KPIs**; **stock** and **catalog** CRUD; **fixed shipping price** editable; **new categories** creatable without deploy.
4. **Owner notification:** Marcus receives an email (Resend) on **each completed sale** (configurable template, includes order summary).
5. **Data cutover:** Excel stock imported into Supabase; **go‑live day** refresh procedure documented and executed once.
6. **Supabase Storage:** buckets, ACLs, and env vars ready; products render with placeholders until images exist.
7. **i18n:** **Spanish and English** across customer-facing surfaces; admin can be ES/EN per preference (minimum: customer-facing full i18n).
8. **GDPR:** cookie consent for non-essential cookies; privacy/cookie pages; consent logging.
9. **Responsive UX:** strong layouts on **mobile and desktop**; performance acceptable under normal load; **drop mode** demonstrably prevents checkout stampede (load test or documented queue behaviour).
10. **Deployed** to Vercel with staging + production, secrets managed, Stripe webhooks verified, and a short **runbook** for drop days (enable drop mode, monitor queue, disable after event).

---

## Appendix A — Brand & UX direction (non-binding for engineering, binding for design)

- Audience: **men 20–40**, Spain, accessories (belts, jewellery, hats, eyewear).
- Tone: confident, minimal, high-contrast readability, large tap targets on mobile, fast checkout, photography-led product pages once assets exist.

---

## Appendix B — Open decisions to lock before build

- Guest checkout: **yes/no** (recommended: yes with email).
- Exact **carrier** and API fields for tracking URL.
- Tax display rules (VAT included in shown prices vs added at checkout) for Spanish B2C.
- Whether **Marcus sale email** is duplicate of admin notification or separate “owner digest”.
- Drop queue UX: show **queue position** vs **generic wait**.

---

*Document version: 1.1 — aligned with stakeholder brief (Marcus, men’s accessories, Spain, EUR, ES/EN, Next.js + Supabase + Stripe + Resend + Vercel + Upstash). §2.4: MVP auth model is extended staff nav + protected `/admin`/`/owner` + server checks + RLS; impersonation optional later only.*
