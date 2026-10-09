# Sib's Trading – Ordering & Inventory System

An ordering and inventory system for **Sib's Trading**, a raw chicken and chicken by-products business. Customers browse stock and place orders. The admin manages stock, orders and customers.

The whole app is **one file**: `sibs-trading.html` (plain HTML, CSS and JavaScript, no framework, no build step). It runs in one of two modes:

| Mode | When | Where data lives |
|---|---|---|
| **Cloud** (normal) | the page is hosted on the web and can reach Firebase | Your Firebase Realtime Database, shared by every device |
| **Local** (fallback) | Firebase cannot be reached, or `USE_FIREBASE` is `false` | The browser on one device only. The app shows a notice when this happens |

**Before cloud mode works you must follow `FIREBASE_SETUP.md`** (publish the rules in `database.rules.json`, enable sign-in, create the admin account). Never leave the database in test mode.

---

## Features (what is actually implemented)

### Login and roles
- **One login page** for customers and admin. After logging in, the system decides the role: admin goes to the Admin dashboard, customers go to the Shop.
- **Cloud mode:** sign-in uses Firebase Authentication (email and password). The role comes from the database: only user IDs listed under `/admins` (which only you can edit, in the Firebase console) are admins. A customer profile cannot make someone an admin.
- **Forgot password** (cloud): sends a real reset email. Admin can also send a customer a reset email.
- **One login per browser:** every tab of the same browser shares the same login. Opening the site in a second tab opens the account that is already logged in; a login page left open in another tab switches to it as soon as someone logs in; logging out in one tab logs out all of them. To switch accounts, log out first.
- **Automatic log out:** after 12 hours with no click or key press (in any tab) the person is logged out and asked to log in again. Because the login is now kept when the browser is closed, **use Log out on shared computers.**
- **Login lock:** 3 wrong tries in a row pauses login for 1 minute (countdown, survives a refresh).
- **Local mode:** usernames instead of emails, passwords stored as salted PBKDF2 hashes in the browser, admin resets customer passwords.

### Customer features
- Browse by category (Whole & cuts, By-products) with search, description, price per unit and live availability.
- Basket, checkout (pickup or delivery, payment method, note), order confirmation with a printable and downloadable receipt.
- **Contact number checks:** phone fields (checkout, Account, and the shop number in Settings) accept **digits only**: letters and symbols are removed as you type or paste, and a `+` is allowed only at the start. A customer's number must be a valid mobile number (11 digits starting with 09, or +63 9...), and obviously fake numbers (all the same digit, 1234567..., 0000...) are refused with a clear message under the field. Numbers are stored in one standard form (09171234567) and shown neatly (0917 123 4567). The check is repeated when the order is saved. It checks the **format**; it cannot know whether the number is in service (that needs an SMS verification code, which needs a server).
- **My orders** with a progress bar (Pending, Confirmed, Processing, Ready, Completed) and receipt.
- **Account** page: edit name, phone, delivery address; change password.
- **Notifications at every step of the order:** a bell with an unread count and a list, so customers can follow their order. Each step gives a notification, a pop-up and a short chime (the chime can be switched off in the bell panel):
  1. **Order received**: right after the customer places the order.
  2. **Order confirmed**: when the admin confirms it.
  3. **Order being prepared**: when it moves to Processing.
  4. **Order ready for pickup / ready for delivery**: when it is Ready (the wording follows the order type).
  5. **Order completed**: when it is Completed.
  6. **Order cancelled**: if it is cancelled.
  The bell keeps the whole history of each order (newest first), it is still there after a reload, and clicking an entry opens that order with its progress bar. Updates arrive **live while the app is open**.

### Admin features
- **Layout:** a left sidebar (Dashboard, Orders, Inventory, Customers, Reports, Settings, with your name and Log out at the bottom) and a slim top bar with the page name and the notification bell. On phones the sidebar becomes a slide-in menu opened with the menu button. Customers keep a top navigation bar.
- **Customer view (preview):** a **Customer view** page in the sidebar shows the shop exactly as customers see it, inside a browser-style frame, with a **Desktop / Phone** switch, search and category filters. It is read-only (no ordering), shows the same products, prices, descriptions and availability, and updates live when you change stock or prices or a customer orders. The dashboard also has a small **Customer view** card with the first four products and a button to open the full preview.
- **Dashboard:** revenue, sales today, total/pending/completed orders, customers, products in stock, low/out-of-stock, restock list, latest orders.
- **Orders:** search, status filters with counts, change status (customer is notified for the statuses you choose), details window, receipts. Cancelling returns the items to stock once.
- **Inventory:** products with search, filter and sort; In Stock / Low Stock / Out of Stock; stock-in and stock-out totals; add/edit/delete products; **manual stock** (Add, Remove, Set count) with a note; stock history with product filter.
- **Customers:** list; reset email (cloud) or password (local); deactivate/remove.
- **Reports:** orders, revenue, average order, top products, orders by status; CSV export.
- **Notifications:** bell with unread count for each new order; click to open the order.
- **Settings:** receipt address, phone, message, currency; change password; system problems list.

### Notification settings (safe to edit)
At the top of the script, in the **NOTIFICATION SETTINGS** block:

| Setting | Meaning |
|---|---|
| `ENABLE_ADMIN_DASHBOARD_NOTIFICATIONS` | Admin bell for new orders |
| `ENABLE_CUSTOMER_NOTIFICATIONS` | Customer bell |
| `NOTIFY_CUSTOMER_ORDER_RECEIVED` | "Order received" right after ordering |
| `NOTIFY_CUSTOMER_ON_STATUSES` | Order steps that notify the customer, after "received". Default: all of them (`'confirmed'`, `'processing'`, `'ready'`, `'completed'`, `'cancelled'`). Delete any you do not want announced |
| `SHOW_NOTIFICATION_TOASTS` | Pop-up when a notification arrives |
| `TOAST_MS`, `MAX_TOASTS` | How long a pop-up message stays (default 3500 ms) and the most shown at once (default 3). The same message is never shown twice at once, and clicking a message closes it |
| `NOTIFICATION_SOUND` | Short "ding-dong" chime when a notification arrives (admin and customers). Each person can switch it off with **Sound: on/off** in the bell panel; that choice is remembered on their device |

The wording of customer messages is in the function `customerNotice`.

**Not included:** email, SMS and phone push notifications (they need a server-side piece; see `SYSTEM_RECOMMENDATIONS.md`). Notifications show only while the app is open.

### Inventory and order rules
- Stock changes only through logged actions (manual add/remove/set, orders, cancellations), each written to the stock history.
- Stock can never go below zero.
- Order lines and totals are built from the product records, never from the basket. In cloud mode the totals shown are always recalculated from the order lines.
- **Number boxes take numbers only:** every quantity and price box (shop, basket, admin stock, product price, low-stock level, starting stock) accepts only digits and one decimal point. Letters such as "e", and "+" or "-", are removed as they are typed or pasted; a comma counts as a decimal point; at most 2 decimals; items sold per `pc` or `pack` take whole numbers only (the decimal point is removed). Phones show a number keypad.
- `pc` and `pack` items must be whole numbers; limit of 500 per product per order (configurable).
- Cloud mode: stock is taken with atomic "subtract if enough is left" steps, so two customers cannot both take the last unit.
- Notifications are created after the order is saved and can never cancel an order. Problems are listed in **Settings > System problems**.

### Not included
Email/SMS/phone push, GPT/OpenAI, product images, returns/refunds, server-side order processing.

---

## Technology (actual)

| Part | What is used |
|---|---|
| App | One HTML file: plain JavaScript, CSS |
| Database | Firebase Realtime Database (cloud mode) or browser `localStorage` (local mode) |
| Sign-in | Firebase Authentication, email and password (cloud) |
| Firebase SDK | Firebase JavaScript SDK v10.14.1, loaded from Google's CDN when the page opens |
| Hosting | Not included: use Firebase Hosting or any static host |
| Email / SMS / OpenAI | None |

## Installation and running

1. Follow **`FIREBASE_SETUP.md`** (rules, sign-in, admin account, hosting).
2. Open the hosted address and log in as the admin.
3. Click **Load sample products** (or add your own), set real prices, then add your first stock.

**Quick local try-out (no Firebase):** set `USE_FIREBASE=false` near the top of the script and open the file in a browser. Log in with `admin` / `admin123` and change the password in Settings.

### Environment variables / database setup
There is **no `.env` file**: this is a browser app, and anything in it is visible to visitors. The Firebase web configuration in `FIREBASE_CONFIG` identifies your project but is not a secret; the rules protect the data. `OPENAI_API_KEY` and `ADMIN_NOTIFICATION_EMAIL` only make sense once a server exists (see recommendations); they are not used here.

---

## What I can safely change manually

All in `sibs-trading.html`. Open it in a text editor and search for the text shown.

| What | Setting | Search for |
|---|---|---|
| Business name | `APP_NAME` (and the `<title>` tag at the top of the file) | `MANUAL CONFIGURATION` |
| Tagline | `APP_TAGLINE` | same block |
| Use cloud or local mode | `USE_FIREBASE` | `DATABASE (FIREBASE)` |
| Firebase project | `FIREBASE_CONFIG` | same block |
| Firebase SDK version | `FIREBASE_SDK_VERSION` | same block |
| Default currency (first run) | `DEFAULT_CURRENCY`. Afterwards: Admin > Settings | `MANUAL CONFIGURATION` |
| Receipt number format | `ORDER_PREFIX`, `ORDER_NUMBER_OFFSET` | same block |
| Low-stock default for new products | `LOW_STOCK_DEFAULT` | same block |
| Max quantity per product per order | `MAX_QTY_PER_ITEM` | same block |
| Automatic log out after inactivity | `SESSION_IDLE_MINUTES` (default 720 = 12 hours; `0` = never) | same block |
| How long the loading animation shows when changing tabs | `PAGE_TRANSITION_MS` (default 600; `0` switches it off; skipped automatically on devices set to "reduce motion") | same block |
| Login tries and lock time | `MAX_TRIES`, `LOCK_MS` | same block |
| Minimum password length | `MIN_PASSWORD_LENGTH` | same block |
| Which phone numbers are accepted | `PHONE_REGION`: `'PH'` (Philippine mobile numbers, default) or `'ANY'` (any international number, 8 to 15 digits with an optional +). Also update the phone rules in `database.rules.json` if you change it | same block |
| Notification switches and which statuses notify | see the notification table above | `NOTIFICATION SETTINGS` |
| Customer notification wording | `customerNotice` | `The wording a customer sees` |
| Sample products (first run, or "Load sample products") | `SAMPLE_PRODUCTS` | `Sample products` |
| Local-mode default admin login | `DEFAULT_ADMIN_USERNAME`, `DEFAULT_ADMIN_PASSWORD` | `DEFAULT ADMIN ACCOUNT` |
| Theme colors and fonts | the `:root{ ... }` variables at the top of the `<style>` | `--pine`, `--saffron` |
| Receipt look | `RECEIPT_CSS` | `RECEIPT_CSS` |
| Security rules | `database.rules.json` (then publish in the Firebase console) | |
| Admin notification email, OpenAI settings | **not in this app** (need a server) | |

**Do not change** (marked `IMPORTANT` in the code): storage keys, the password section (local mode), the inventory logic section, and the `STORES` section (order placement, stock and cancellation). They keep stock, history and orders consistent.

---

## Troubleshooting

| Problem | Likely cause and fix |
|---|---|
| Yellow notice "Cloud database not reachable" | The page can't load Firebase (offline, blocked network, or opened inside a preview that blocks it). The app is in local mode and data stays on this device. Host it and open the real address |
| "Could not read data from the database" | Rules are missing or wrong. Publish `database.rules.json` |
| Admin logs in but sees the Shop | The admin's User UID is not under `/admins` with the value `true`. See setup step 3 |
| "This account is not active" | The customer was deactivated (Customers page) or has no profile. Reactivate in Customers |
| Customer can't register | Email already used, password under 6 characters, or Email/Password sign-in not enabled |
| "I am still logged in after closing the browser" | By design: the login is shared by all tabs and kept until you log out or the inactivity limit passes. Use **Log out**, especially on shared computers |
| I want to log in as a different person | Log out first (it logs out every tab), then log in with the other account |
| "You were logged out because of inactivity" | No click or key press for `SESSION_IDLE_MINUTES`. An admin who leaves the screen open only to hear alerts will be logged out after that time; raise it, or set it to `0`, if you prefer |
| "Enter a valid mobile number" / "does not look real" | The number must be 11 digits starting with 09 (or +63 9...) and not an obvious pattern. Ask the customer for their real mobile number. For overseas customers, set `PHONE_REGION` to `'ANY'` (and update the rules file) |
| Print does nothing | Some embedded views block printing. Use **Download** and print the file |
| "Log in is paused" | 3 wrong tries. Wait 1 minute (browser-based, per device) |
| No notification on the customer's phone while the app is closed | Not supported: notifications appear only while the app is open |
| No sound when a notification arrives | Browsers allow sound only after you have clicked or tapped the page once (logging in counts). After a page reload, click anywhere once. Also check **Sound: on** in the bell panel, that the device is not muted, and that `NOTIFICATION_SOUND` is `true`. The chime is generated by the browser; there is no sound file to replace |
| Something went wrong message | A problem was caught and, if you are the admin, listed in Settings > System problems |

## Security notes (honest summary)
- Cloud mode: passwords are handled by Firebase Authentication. Access control is enforced by Firebase using `database.rules.json`, **not** by this page. Publish the rules.
- The rules let a customer **lower** a product's stock when ordering. Without a server they cannot verify that a decrease matches a real order, so a skilled, signed-in customer could misuse this. Moving order placement into a server function is the fix (see `SYSTEM_RECOMMENDATIONS.md`).
- Local mode has no server at all: someone with browser developer tools can alter data on their own device.
- The login lock is stored in the browser; Firebase adds its own limit on repeated failed sign-ins.

## Tested

**Local mode** (headless Chromium, final file): 271 automated checks pass. They cover login and roles, lockout, tampered sessions, hash upgrade, manual stock, ordering and totals, live updates between two tabs, admin and customer notifications (received, ready for pickup, ready for delivery, click-through, mark all read), cancellation, the last-unit race, a notification failure not breaking an order, receipts, CSV export, search, the admin sidebar (desktop and the phone slide-in menu), the logo going home, the tab-change loading animation (timing, fast clicks, log out during loading, reduced motion), number boxes (no letters, e, + or -; one decimal point; 2 decimals; whole numbers for pieces; pasted text cleaned; basket and admin boxes; wrong values still refused), pop-up messages (repeated clicks show one message, at most three at a time, click to dismiss, red quantity box), customer notifications at every order step (received, confirmed, preparing, ready for pickup or delivery, completed, cancelled: bell, pop-up and chime each time, history kept after reload, repeat status not duplicated, only-Ready setting still works), contact-number checks (letters and symbols removed, invalid and fake numbers refused, standard storage form, account and shop number fields, the international setting), the Customer view preview (same products, prices and availability as the real shop, read-only, search, categories, desktop/phone size, live updates, no sideways scroll on a phone), the shared login between tabs (second tab opens the logged-in account, log out and log in follow across tabs, tampered sessions rejected, inactivity log out), and the notification sound (it plays for admin and customer when it should, not for your own order, and stays silent when switched off).

**Cloud mode** (55 checks pass), **against a stand-in for Firebase that I built for testing, not against real Firebase.** They check my code's logic: sign-in through the same page, admin-from-database role, registration, ordering with atomic stock, live notifications both ways, tampered totals ignored, customer isolation, deactivation, password reset, fallback to local mode.

The sound was tested by checking that the browser is asked to play the two chime notes; I cannot hear audio, so please listen to it once yourself.

**Not tested:** the real Firebase service, your project's configuration, the security rules (they were never run on Firebase's rules engine; use the Rules Playground), the browser print dialog, Safari and Firefox, phones in real use.
