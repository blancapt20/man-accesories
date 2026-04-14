# Marcus — Men’s Accessories

An e‑commerce project for **Marcus**, a Spain‑based business selling **men’s accessories** (belts, rings, sunglasses, hats, necklaces, earrings, and more categories over time). The shop is aimed at customers **roughly 20–40 years old**, with a **Spanish and English** storefront, **EUR** prices, and **shipping within Spain** only.

This repository holds the **product specification**, **architecture decisions**, and **Cursor rules** for implementation. The Next.js application will be added here as development continues.

---

## What you get (product)

- **Customer experience:** landing → catalog → product pages → cart → checkout → **Stripe** payments → confirmation email (**Resend**) → order tracking (including **carrier** tracking links when available).
- **Limited drops:** support for high traffic on drop days via a **queue** (see [SPEC.md](./SPEC.md)) so inventory is not oversold under load.
- **Three roles:** **Customer**, **Admin** (Marcus’s team), and **Owner** (Marcus). Admins manage catalog, stock, KPIs, and a configurable **fixed shipping** fee. The Owner manages who has Admin access. Staff use the same storefront as customers plus protected **`/admin`** and **`/owner`** areas—no impersonation required for MVP.
- **Compliance & UX:** **GDPR‑aware** cookie consent, strong **mobile and desktop** layouts, and product images via **Supabase Storage** once assets are uploaded.

---

## Tech stack

| Area | Choice |
|------|--------|
| App & hosting | **Next.js** (App Router), **Vercel** |
| Data, auth, files | **Supabase** (Postgres, Auth, Storage, RLS) |
| Payments | **Stripe** |
| Email | **Resend** |
| Drops / concurrency | **Upstash** (Redis / QStash), per spec |
| UI | **Tailwind CSS**, **shadcn/ui** |

---

## Documentation

| Document | Purpose |
|----------|---------|
| **[SPEC.md](./SPEC.md)** | Master specification: flows, roles, data models, edge cases, MVP success criteria, and out‑of‑scope items. **Start here** for behaviour and scope. |
| **[development/SPRINTS.md](./development/SPRINTS.md)** | MVP sprint breakdown (goals, scope, traceability to SPEC); per‑sprint detail in [`development/sprints/`](./development/sprints/). |
| [`.cursor/rules/`](./.cursor/rules/) | Editor/agent rules: spec‑driven workflow, stack, TypeScript strictness, secrets, and project layout. |

---

## Repository status

There is **no `package.json` yet**—the runnable Next.js app is not in this tree. When the app is scaffolded, this README will be updated with **install**, **dev server**, and **environment variable** instructions (see `.env.example` when it exists).

---

## License

No license file is included yet. Add one when you are ready to open‑source or share the repo.
