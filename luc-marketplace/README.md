# LUC Marketplace
**LUC = Logozor Unique Computer**

A general-purpose Jumia/Temu-style e-commerce demo, rebuilt from the SwapZone
Market codebase. Same Supabase project and connection code — new tables so
nothing from SwapZone is touched or removed.

## What's new vs SwapZone Market

- General items instead of just phones — flexible categories, stock counts
- Cart (localStorage, works without login) → Checkout → Paystack **or** WhatsApp
- Customer Login / Sign Up (Supabase Auth) + My Orders page
- Admin panel: Products tab (unchanged workflow, now with Edit) + new **Orders** tab

## 1. Database setup

1. Open your Supabase project → **SQL Editor** → New query.
2. Open `setup.sql`, replace every `YOUR_ADMIN_EMAIL` with your real admin
   email address, then run it. This creates `luc_products` and `luc_orders`
   only — it does not touch the old `products` table.
3. Go to **Storage** → New bucket → name it exactly `luc-product-images` →
   turn **Public bucket** ON → Save. (The storage policies at the bottom of
   `setup.sql` already assume this bucket name.)
4. Go to **Authentication → Users** → Add user → create your admin account
   with the same email you put in `setup.sql`.

## 2. App configuration

Open `supabase-config.js` and set:

```js
window.LUC_ADMIN_EMAIL = "your-admin-email@example.com";       // must match setup.sql
window.LUC_WHATSAPP_NUMBER = "233241234567";                    // no + or spaces
window.LUC_PAYSTACK_PUBLIC_KEY = "pk_test_xxxxxxxxxxxxxxxxxxxx"; // Paystack PUBLIC key only
```

Get the Paystack public key from your Paystack Dashboard → Settings → API
Keys & Webhooks. Never put the **secret** key in this file — it's exposed
to every visitor's browser.

## 3. Pages

| File | Purpose |
|---|---|
| `index.html` | Storefront — search, category chips, product modal, add to cart |
| `cart.html` | Cart with quantity controls, order total |
| `checkout.html` | Requires login. Delivery form, Paystack or WhatsApp checkout |
| `login.html` | Customer login / sign up |
| `orders.html` | Customer's own order history |
| `admin.html` | Admin login → Products tab (CRUD) + Orders tab (status updates) |

## 4. How checkout works

- **Paystack**: opens the Paystack inline popup for the cart total (GHS →
  pesewas conversion is automatic). On success, an order is saved with
  `payment_status = 'paid'`.
- **WhatsApp**: opens `wa.me` with a prefilled order message (items, total,
  delivery details) to `LUC_WHATSAPP_NUMBER`. An order is saved with
  `payment_status = 'pending'` for you to confirm manually once payment is
  received.

Every order's `order_status` (processing → confirmed → shipped → delivered
→ cancelled) is managed from the admin Orders tab.

## 5. Deploying

Same as SwapZone — this is a static site (no build step). Drop the folder
into Netlify (drag-and-drop or connect the repo) and it works as-is. If you
use a custom domain, just point DNS the same way you did for the other EVOS
products.

## 6. Security notes (demo-level)

- Admin access is enforced by **Row Level Security** matching your admin
  email — the client-side check in `admin.js` is just a UX nicety, not the
  real gate.
- If you add more admins later, you'll need to update the RLS policies in
  `setup.sql` (e.g. check against a list, or a `luc_admins` table) instead
  of a single hardcoded email.
- The Paystack integration here is **client-only** for demo purposes: it
  trusts the browser's "payment successful" callback. For production, add
  a server-side webhook (Paystack → your backend) that verifies the
  transaction with your **secret** key before treating an order as paid.
