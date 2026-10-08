# Sib's Trading – System Recommendations

Labels used throughout:

- **[Implemented]** exists in `sibs-trading.html` today
- **[Recommended]** should be done next; not built
- **[Future]** optional, later

Nothing marked Recommended or Future is claimed to exist.

---

## System Overview

A single-file web app (`sibs-trading.html`): plain HTML/CSS/JavaScript, no backend, no database server. Data is one JSON object in the browser's `localStorage`:

`settings, admin, users[], products[], orders[], stockLog[], notifications[], errorLog[], counter`

Roles are `admin` (one account) and customer. The role is determined at login from which account matched. There is no role field on customer records.

## Current Features **[Implemented]**

Unified login with lock; customer shop, basket, checkout, receipts, order history with progress, account page; admin dashboard, orders, inventory (manual stock, history, stock-in/out totals), customers, reports with CSV export, settings; in-app notifications with live update across tabs; error log; loading, empty and error states; responsive layout with card tables on phones; keyboard and screen-reader labelling.

## Strengths

- Zero setup, runs anywhere, easy to read and edit (one file, commented configuration block).
- Inventory rules are strict: no negative stock, every change logged, cancellations restore stock once, totals computed from product records.
- Orders never fail because of notifications.
- Existing data is migrated automatically between versions.

## Weaknesses

- **No server.** Anything stored or checked in the browser can be altered by a technical user on that device.
- **Single-device data.** Orders placed on a customer's phone are not visible to the admin on another device. This is the biggest practical limitation for a real business.
- No backup. Clearing browser data deletes everything.
- No email, SMS or push alerts: the owner is notified only while the admin page is open in the same browser.
- Passwords can't be reset by email.
- Storage limit (about 5 MB) and no concurrency control beyond re-reading before writes.
- No product images, taxes, discounts, returns/refunds, or delivery fees.

---

## Security Recommendations

**[Implemented]**
- Salted PBKDF2-SHA256 password hashing, with automatic upgrade of old hashes.
- Login lock (3 tries, 1 minute), the same response time for unknown usernames.
- Role taken from the stored account, not from the saved session; tampered sessions rejected (tested).
- All user text is HTML-escaped; CSV export blocks spreadsheet formula injection.
- Friendly errors; technical details go to the admin's System problems list, not to customers.
- Warning banner while the default admin password is in use.

**[Recommended]** (needs a server)
1. Move authentication and authorization to a backend. Check the user's role on **every** admin request on the server. Never trust role, price or total from the browser.
2. Use server-side sessions or short-lived signed tokens in `HttpOnly`, `Secure`, `SameSite` cookies.
3. Hash passwords on the server (Argon2id or bcrypt) and rate-limit login by IP and account on the server. The current lock can be reset by clearing browser data.
4. Remove the default admin password: force a password choice on first setup.
5. Add admin audit logging (who changed stock, prices, order status).
6. Store secrets (database credentials, API keys) only in environment variables on the server.

**[Future]** Two-factor authentication for admin; admin invitation flow.

## UI/UX Recommendations

**[Implemented]** One design system (type scale, spacing, buttons, inputs, cards, tables, modals, badges, toasts, icons paired with labels), light/dark theme tokens, subtle animations that respect reduced-motion, mobile card layout for tables, loading and empty states.

**[Recommended]** Add product photos; real-device testing on small Android phones; a printable daily orders sheet for the kitchen/cold room; a second-language option if your customers need it.

**[Future]** Installable app (PWA) with offline basket.

## Inventory Recommendations

**[Implemented]** Manual stocking (add, remove, set count), In Stock / Low Stock / Out of Stock, per-product low-stock alert, stock history with notes, stock-in/out totals, no negative stock, whole-number rule for pc/pack.

**[Recommended]**
- Track **batches or expiry/received dates**. This matters for raw chicken. Stock-in rows should carry a received date and use-by date, and stock-out should follow first-in, first-out.
- Record **wastage and spoilage** as a named reason (currently a free-text note).
- Separate **supplier** and **cost price** so profit can be reported (reports currently show revenue only, because cost isn't recorded).
- Reserve stock when an order is placed vs. when it is confirmed, if you start taking unpaid orders.

**[Future]** Barcode/QR scanning; weighing-scale integration; supplier purchase orders.

## Ordering Recommendations

**[Implemented]** Totals computed from product records, stock checked on every line, quantity limit, rollback on failure, cancellation restores stock, statuses Pending → Confirmed → Processing → Ready → Completed / Cancelled, receipts (print and download).

**[Recommended]** Real payment records (paid / unpaid / partial); delivery fee and minimum order amount; order cut-off times; **customers cannot cancel their own orders yet**, which you may want within a time window; for catch-weight meat, record the **actual weighed quantity** and adjust the final total.

**[Future]** Online payment integration; repeat-order button; scheduled/standing orders for regular buyers.

## Notification Recommendations

**[Implemented]** In-app admin notification with bell, unread count, list, read state, timestamp, click-through to order; live in-browser update; creation separated from the order and failures logged; settings flags `ENABLE_ADMIN_DASHBOARD_NOTIFICATIONS` and `SHOW_NEW_ORDER_TOAST`.

**Real-time choice, and why.** With no server there is nothing to push from. The browser "storage" event is the simplest reliable option here: it works instantly between tabs of the same browser and needs no extra code or services. It does not cross devices.

**[Recommended] Email to the owner.** Needs a server (an email provider's API key must never sit in browser code).
- Design: after the order is committed, write an *outbox* record, then a background worker sends the email and retries on failure. A failed email is logged and never affects the order.
- Subject: `New Order Received — Sib's Trading #ORD-10025`
- Body: order number, customer, date/time, items with quantities, total, status, and a link to the admin dashboard.
- Recipient from a server setting, for example `ADMIN_NOTIFICATION_EMAIL` in the server's environment. It is not hardcoded.
- Switches: `ENABLE_ADMIN_EMAIL_NOTIFICATIONS` (server config).

**[Future]** SMS or messaging-app alerts to the owner's phone; push notifications when the app is installed. For a business that is often away from the screen, **owner alerts on the phone are the most valuable missing piece**, and are the main reason to build the server version.

## GPT Recommendations (design only, **not implemented**)

GPT is **not** connected in this version and cannot be connected securely in a browser-only file: the OpenAI API key would be visible to anyone. When a backend exists, use this design.

**Architecture**

```
Browser  ->  Your backend (checks login + role)  ->  OpenAI API
                     |                                   |
                     +-- approved read-only tools  <-----+   (model asks to call a tool)
                     |
                     v  validated, parameterised queries
                  Database
```

- The key is read from the server environment (`OPENAI_API_KEY`), never committed, never sent to the browser.
- Use OpenAI's official SDK for whichever language the backend uses (the backend language is a choice you haven't made yet; nothing here assumes one).
- One endpoint, for example `POST /api/assistant`. Requires a logged-in user. Rate-limited, with request logging.

**Approved tools** (the model may only request these; the backend runs them):

| Tool | Who may use | Notes |
|---|---|---|
| `list_low_stock_items()` | admin | uses each product's low-stock level |
| `get_orders_summary(range)` | admin | `range` limited to today / 7 / 30 days |
| `list_pending_orders()` | admin | read-only |
| `top_products(range)` | admin | |
| `search_products(category, max_available, min_available)` | admin and customers | validated numbers and an allow-listed category |
| `get_my_orders()` | customers | only the logged-in customer's orders |

**Hard rules**
- No free-form SQL, ever. Each tool is a fixed function with validated parameters.
- The assistant is **read-only** at first. It cannot change stock, prices, roles, accounts, or orders. If you later add write actions, they must call the same business functions as the normal screens, require explicit user confirmation, and be admin-only and logged.
- The customer role can never see other customers' data or admin tools. Role checks happen on the server before any tool runs.
- Never send passwords, hashes, or API keys to the model.

**Useful features:** "Which products are low?", "Summarise today's orders and stock movement", "Show raw chicken under 50 kg", "Which products sell most?". Treat the model's text as a summary of the tool results, and show the underlying numbers next to it.

## Performance Recommendations

**[Implemented]** Stock log capped (5,000 rows), notifications capped (200), error log capped (100), search and filters update only the table (not the whole page).

**[Recommended]** On a server version, add database indexes on order date, status and product, and paginate orders and history instead of loading everything. The current whole-database-in-memory approach is fine for a small shop but slows down with many thousands of orders.

## Future Features (optional)

Customer self-cancellation window; loyalty/regular-customer pricing; multiple staff accounts with limited roles (for example "stock clerk" can add stock but not see revenue); delivery route list; daily closing report; accounting export.

---

## Suggested order of work

1. **Back up regularly:** export orders to CSV from Reports (available now).
2. **Build the server version** with real accounts, one shared database, server-side authorization (this unlocks everything below).
3. **Owner email/phone alerts** for new orders.
4. Batch and expiry tracking, cost price and profit.
5. The read-only GPT assistant, using the tool design above.
