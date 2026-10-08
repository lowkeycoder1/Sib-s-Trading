# Sib's Trading – Ordering & Inventory System

A browser-based ordering and inventory system for **Sib's Trading**, a raw chicken and chicken by-products business. Customers browse stock and place orders. The admin manages stock, orders and customers.

> **Read this first: what this system is and is not**
>
> The whole system is **one file**, `sibs-trading.html`. It is plain HTML, CSS and JavaScript with **no framework, no backend server, and no database server**. All data (products, orders, accounts) is stored in the browser's `localStorage` on the device that opens the file.
>
> That means some things in a "professional" system are **not possible in this version**: server-side authorization, email alerts, and GPT/OpenAI integration (these need a server so that secrets and permission checks stay off the customer's device). They are documented as recommendations in `SYSTEM_RECOMMENDATIONS.md`, not claimed as features.

---

## Features (what is actually implemented)

### Authentication
- **One unified login page** for customers and admin. After a correct login the system decides the role from the account: admin goes to the Admin dashboard, customers go to the Shop.
- Customers create their own accounts ("Create customer account"). There is no way to create an admin account from the login screen.
- **Login lock:** 3 wrong tries in a row pauses login for 1 minute (live countdown; survives a page refresh). Applies to every login attempt.
- Passwords are stored as salted PBKDF2-SHA256 hashes. Hashes from the first version of the app are upgraded automatically the next time that person logs in.
- "Forgot password?" explains how to get a reset (admin resets customers; there is no email reset).
- A saved session is only restored if the account still exists, and the role always comes from the stored account, never from the saved session.

### Customer features
- Browse products by category (Whole & cuts, By-products) with search, description, price per unit, and live availability (Available / Only X left / Sold out).
- Basket with quantity editing, subtotal and total.
- Checkout for pickup or delivery, payment method, note.
- Order confirmation with a printable and downloadable receipt.
- **My orders** with status progress (Pending, Confirmed, Processing, Ready, Completed) and receipt.
- **Account** page: edit name, phone, delivery address; change password.

### Admin features
- **Dashboard:** revenue, sales today, total/pending/completed orders, customers, products in stock, low/out-of-stock count, restock list, latest orders.
- **Orders:** search, status filters with counts, change status, order details window, receipts. Cancelling returns the items to stock (once only).
- **Inventory:** products table with search, stock-status filter and sorting; status labels (In Stock / Low Stock / Out of Stock); stock-in and stock-out totals; add/edit/delete products; **manual stock update** (Add, Remove, Set count) with a note; full stock history with product filter.
- **Customers:** list, reset password, remove.
- **Reports:** orders, revenue, average order, top products, orders by status for Today / 7 days / 30 days / All time; **CSV export** of orders.
- **Settings:** receipt address, phone, message and currency symbol; change admin password; recent system problems list; reset all data.

### Notifications (implemented)
- **In-app admin notifications:** when a customer places an order, a notification is saved. The admin sees a bell with an unread count, a list (order number, customer, item count, total, "time ago"), read/unread state, "Mark all as read", and clicking a notification opens that order. The browser tab title also shows the unread count.
- **Live updates:** if the admin and customer are using the **same browser**, the admin screen updates instantly (browser storage events), including a pop-up "New order received".
- **Failsafe:** the notification is created *after* the order is saved, in its own error handler. If it fails, the order still stands and the failure is written to **Settings > System problems**.
- **Not implemented:** email notifications, SMS, push notifications. See `SYSTEM_RECOMMENDATIONS.md`.

### Inventory and order rules (implemented)
- Stock changes only through logged actions (manual add/remove/set, orders, cancellations). Every change is written to the stock history.
- Stock can never go below zero (manual removal and ordering both check).
- The order total is calculated from the product record at the time of the order, never from what is typed in the basket.
- Items sold per `pc` or `pack` must be whole numbers; kilograms allow decimals.
- A single product is limited to 500 units per order (configurable).
- Before an order or a stock change, the latest data is re-read from storage, so two tabs cannot sell the same last kilogram.
- If saving an order fails half-way, everything is rolled back.

### Not included
Product images, email/SMS, GPT/OpenAI, a server, a database server, multi-device sync, returns/refunds (only cancellation).

---

## Technology stack (actual)

| Part | What is used |
|---|---|
| Frontend | Plain HTML, CSS and JavaScript in one file |
| Fonts | Google Fonts (Bricolage Grotesque, DM Sans), with system fallbacks |
| Backend | None |
| Database | Browser `localStorage` (one JSON object) |
| Authentication | Client-side, PBKDF2-SHA256 via the browser's Web Crypto |
| Email / SMS / OpenAI | None |

---

## Installation and running

No installation is needed.

1. Open `sibs-trading.html` in a modern browser (Chrome, Edge, Firefox, Safari), **or** publish it to any static web host.
2. **First login (admin):** username `admin`, password `admin123`.
3. **Change the admin password immediately** in Settings. A banner reminds you until you do.
4. Go to Inventory and add your first delivery. Stock starts at 0, so customers see "Sold out" until you do.
5. Replace the sample prices with your own.

Important: data lives in the browser on the device where the file is opened. The admin and customers must use the same browser on the same device to see each other's orders. For several devices, a server version is needed (see recommendations).

### Environment variables / database setup
**None.** This version has no `.env` file and no database to set up. Variables such as `OPENAI_API_KEY` or `ADMIN_NOTIFICATION_EMAIL` only make sense once a backend exists, and are described in `SYSTEM_RECOMMENDATIONS.md`.

---

## What I can safely change manually

All settings are in `sibs-trading.html`. Open the file in a text editor and search for the text in the right-hand column.

| What | Where | Search for |
|---|---|---|
| Business name | `APP_NAME` | `MANUAL CONFIGURATION` (top of the `<script>`). Also edit the `<title>` tag near the top of the file |
| Tagline on the login page | `APP_TAGLINE` | same block |
| Default currency symbol (first run only) | `DEFAULT_CURRENCY` | same block. Afterwards change it in Admin > Settings |
| Receipt number format | `ORDER_PREFIX`, `ORDER_NUMBER_OFFSET` | same block |
| Low-stock default for new products | `LOW_STOCK_DEFAULT` | same block. Each product's own alert level is set when adding/editing the product |
| Maximum quantity per product per order | `MAX_QTY_PER_ITEM` | same block |
| Login tries and lock time | `MAX_TRIES`, `LOCK_MS` | same block |
| Minimum password length | `MIN_PASSWORD_LENGTH` | same block |
| Dashboard notifications on/off | `ENABLE_ADMIN_DASHBOARD_NOTIFICATIONS` | `NOTIFICATION SETTINGS` |
| "New order received" pop-up on/off | `SHOW_NEW_ORDER_TOAST` | `NOTIFICATION SETTINGS` |
| How many notifications are kept | `MAX_NOTIFICATIONS_KEPT` | `NOTIFICATION SETTINGS` |
| Default admin username / password (first run only) | `DEFAULT_ADMIN_USERNAME`, `DEFAULT_ADMIN_PASSWORD` | `DEFAULT ADMIN ACCOUNT` |
| Sample products and prices (first run only) | the `products:[ ... ]` list inside `seed()` | `Sample products` |
| Theme colors and fonts | the `:root{ ... }` variables at the top of the `<style>` | `--pine`, `--saffron`, `--f-display` |
| Receipt look | `RECEIPT_CSS` | `RECEIPT_CSS` |
| Receipt address, phone, message, currency | in the app | Admin > Settings |

Settings that need a backend (**not in this file**): admin notification email, OpenAI settings, API base URL. See `SYSTEM_RECOMMENDATIONS.md`.

**Do not change** (marked `IMPORTANT` in the code): the storage keys (changing them starts an empty database), the password hashing section, `logStock()` and the inventory logic section, and `forms.checkout`. These keep stock, history and orders consistent. Changing the first-run seed data only affects a fresh browser; it does not alter existing saved data.

Existing data is kept when you upgrade from the previous file. It is migrated automatically.

---

## Forgot the admin password

There is no email reset. If the admin password is lost, the only option is to clear this page's site data in your browser, which **deletes all products, orders and accounts**. Export your orders first from Reports > Export orders (CSV) and change the password regularly to a value you keep safe.

## Troubleshooting

| Problem | Likely cause and fix |
|---|---|
| Customers can't see the admin's stock changes | Different browser or device. Data is stored per browser. Use one browser, or move to a server version |
| A banner says saving is turned off | The browser blocks storage (private mode or blocked cookies/site data). Allow site data |
| "Log in is paused" | 3 wrong tries. Wait for the countdown (default 1 minute) |
| Print does nothing | Some embedded or restricted views block printing. Use **Download** and print the downloaded file |
| Prices show the wrong currency | Admin > Settings > Currency symbol |
| Everything disappeared | Site data was cleared. There is no automatic backup. Export CSV regularly |
| Something went wrong message | A problem was caught and recorded under Admin > Settings > System problems |

## Security notes (honest summary)
- Because there is no server, **all checks run in the browser**. Someone who knows how to use browser developer tools can alter locally stored data on their own device. This is acceptable for a single shop-counter device but not for an internet-facing system holding real customer data.
- Admin-only actions are guarded in code, and customers cannot open other customers' orders through the interface. These are guards in the browser, not server enforcement.

## Tested

Tested in headless Chromium against the final file (46 automated checks, all passing): unified login and role routing, registration, lockout and its expiry and refresh persistence, tampered-session rejection, hash upgrade, manual stock in/out/validation, ordering, totals, live stock and live notifications between two tabs, notification click-through, status updates, cancellation and single stock restore, two-buyer last-unit race, notification-failure resilience, receipt download, CSV export, search/filter, and mobile layout (no horizontal scroll).

**Not tested:** the browser print dialog, Safari/Firefox, real multi-device use, and the host download permission path (the standard browser download path was tested).
