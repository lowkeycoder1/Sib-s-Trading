# Sib's Trading: Ordering and Inventory Management System

An online ordering and inventory system for **Sib's Trading**, a business that sells raw chicken and chicken by-products.

- **Live website:** <https://sibstrading.vercel.app>
- More detail: [TECHNICAL_NOTES.md](TECHNICAL_NOTES.md) (settings and behaviour), [FIREBASE_SETUP.md](FIREBASE_SETUP.md) (Firebase setup guide), [SYSTEM_RECOMMENDATIONS.md](SYSTEM_RECOMMENDATIONS.md) (strengths, weaknesses and next steps)

---

## Business Name and Description

**Sib's Trading** sells raw chicken and chicken by-products by weight. Products include whole chicken and cuts (whole chicken, breast, thigh, drumstick, wings) and by-products (feet, liver, gizzard, heart, neck, skin). The owner can add, change or remove products at any time.

**Who uses the system**

- **Customers** create an account, browse the shop, order for pickup or delivery, get a receipt and receive a notification at each step of their order.
- **The admin (shop owner)** manages products and stock by hand, processes orders, sees reports, manages customer accounts and is notified of every new order.

## Problem Being Solved

A raw chicken business sells perishable goods by weight, so it must always know exactly how much of each product is on hand, and customers need to know what is available and what they will pay before they order. When this is handled by phone calls, messages and paper notes, it commonly leads to:

- orders that are lost, forgotten or misunderstood;
- stock counts that are out of date, so more can be sold than the shop really has;
- prices and totals worked out by hand, which can be wrong;
- no record of past orders and no receipts;
- customers who cannot tell whether their order has been received or is ready;
- an owner who is not told straight away that a new order has arrived.

Sib's Trading replaces this with one system: every order is recorded with a receipt number, prices and totals are calculated by the system, stock goes down automatically with each order and can never go below zero, every stock change is logged, and both the owner and the customer are notified as the order moves forward.

## Feature List

**Landing page**

- The public first page shows the business name and tagline, a live price board of the products, and a clear call to action: a **Log in** button and a **Create customer account** link.

**Users and accounts**

- Register (full name, email or username, password), log in and log out.
- Forgot password: a reset link is sent by email (online mode).
- Account page: edit name, phone number and delivery address; change password.
- One login is shared by all tabs of the same browser; automatic log out after 12 hours without activity.
- Repeated wrong passwords pause log in for a short time.

**Roles and ownership**

- Two roles: **Admin** and **Customer**. The role is decided by the database (only accounts listed under `admins` are admins), not by anything the browser sends.
- Customers can only see their own orders and notifications. Admin pages are not available to customers.

**Complete CRUD on Products (main entity)**

| Action | What the system does |
|---|---|
| Create | Admin > Inventory > Add product: name, description, category, unit (kg, pc, pack), price, low-stock level, starting stock |
| Read | Customers browse the shop with search and categories. Admin sees an inventory table with search, stock-status filter (In Stock, Low Stock, Out of Stock), sorting, stock in / out totals and a full stock history |
| Update | Edit a product. Manual stock changes: add stock, remove stock or set an exact count, each with a note |
| Delete | Delete a product after a confirmation window. Past orders and receipts are kept |

**Orders**

- Customer: basket, checkout (pickup or delivery, payment method, note), order confirmation, and a receipt that can be printed or downloaded. My orders shows a progress bar (Pending, Confirmed, Processing, Ready, Completed).
- Admin: orders list with search and status filters, order details, status updates, cancelling an order (the stock is returned once).
- Totals are calculated by the system from the product prices, never typed in by the customer.

**Inventory**

- Stock is changed manually by the admin (delivery, spoiled stock, corrections). It can never go below zero and every change is logged.
- Low-stock level per product, with a restock list on the dashboard.

**Notifications**

- Admin: a bell with an unread count and a short sound for every new order.
- Customer: a notification and sound at every step: received, confirmed, being prepared, ready for pickup or delivery, completed, cancelled.

**Admin panel**

- Dashboard (revenue, orders, customers, stock), reports (today, 7 days, 30 days, all time, top products, CSV export), customers page (send password reset, deactivate or reactivate) and a **Customer view** preview of the shop.

**Validation**

- Contact number: digits only, exactly 11 digits starting with 09; obvious placeholders such as 09111111111 are refused.
- Quantities and prices: digits and one decimal point only; whole numbers for items sold per piece or pack; limits per order; a quantity above the stock is refused.
- Email format, minimum password length, required fields, delivery address required for delivery orders.
- Clear messages; a wrong value turns the box red.

**Design**: responsive for desktop, tablet and phone; light and dark themes; labelled, keyboard-friendly forms.

## Tech Stack

| Part | Technology |
|---|---|
| Frontend | HTML5, CSS3 and JavaScript in a single file (no framework, no build step) |
| Authentication | Firebase Authentication (email and password); passwords are hashed by Firebase |
| Database | Firebase Realtime Database with security rules (`database.rules.json`) |
| Server logic (optional) | Firebase Cloud Functions, Node.js 20 (`firebase-admin`, `firebase-functions`) |
| Local fallback | Browser `localStorage`; Web Crypto PBKDF2-SHA256 password hashing |
| Hosting | Vercel (live site: sibstrading.vercel.app); GitHub Pages or Firebase Hosting also work |
| Fonts | Google Fonts: Bricolage Grotesque, DM Sans |
| Testing (development only) | Playwright with headless Chromium; Node.js for the server order logic |

## Setup and Run Instructions

### What is in the repository

| File or folder | Purpose |
|---|---|
| `index.html` | The whole application (the file is called `sibs-trading.html`; rename it to `index.html` for hosting on Vercel or GitHub Pages) |
| `README.md` | This file |
| `FIREBASE_SETUP.md` | Detailed Firebase setup guide |
| `database.rules.json` | Database security rules (use these first) |
| `database.rules.secure.json` | Stricter rules, only after the Cloud Function is deployed |
| `functions/` | Optional Cloud Function for server-side ordering |
| `firebase.json` | Firebase deployment settings |
| `.gitignore` | Keeps `node_modules` and private files out of the repository |

### A. Quickest way to run it (no accounts needed)

1. Download or clone the repository.
2. Open `index.html` in a text editor, find `const USE_FIREBASE=true;` and change `true` to `false`. Save.
3. Double-click `index.html` to open it in Chrome, Edge or Firefox. Data is kept in that browser only.
4. Log in as the admin (see Test Accounts), change the password in Settings and add stock in Inventory. Then click **Log out**, choose **Create customer account**, register, place an order, log out, and log in as the admin again to see the order and its notification. Local mode keeps its data in one browser, so use the same browser for all of this.

### B. Online with Firebase (shared database, used by the live site)

1. Create a Firebase project at <https://console.firebase.google.com>. Add a **Realtime Database** and enable **Authentication > Sign-in method > Email/Password**.
2. Copy your project's web configuration into `FIREBASE_CONFIG` near the top of `index.html` (the repository already contains the project's configuration; it identifies the project and is not a password).
3. **Realtime Database > Rules:** paste the contents of `database.rules.json` and click **Publish**. Do not leave the database in test mode.
4. **Authentication > Users > Add user:** create the admin (email and password). Copy the new user's **User UID**.
5. **Realtime Database > Data:** add `admins`, then a child named with that UID and the value `true`.
6. Upload the files to a GitHub repository and rename `sibs-trading.html` to `index.html`. Host it on **Vercel** (Add New > Project > import the repository, framework preset "Other", leave the build command empty) or on GitHub Pages (Settings > Pages > Deploy from branch, `main`, `/ (root)`).
7. **Authentication > Settings > Authorized domains:** add the address of your site without `https://`, for example `sibstrading.vercel.app`. Without this, logging in fails on the live site.
8. Open the site, log in as the admin, click **Load sample products** on the dashboard, set the prices in Inventory and add the first stock.

### C. Optional: server-side ordering (Cloud Function)

Needs the Firebase Blaze (pay as you go) plan. In the folder that contains `firebase.json`: `cd functions`, `npm install`, `cd ..`, then `firebase deploy --only functions`. Afterwards publish `database.rules.secure.json` and set `SERVER_ORDERS` to `'required'` in `index.html`. Full steps: `FIREBASE_SETUP.md` (Step 8). Until then the app orders directly from the browser, which still works.

### Forgot the admin password

Online: reset it in the Firebase console (Authentication > Users). Local mode: clear the site data for the page in the browser (this deletes the local data) and the default admin returns.

**Verified from a clean machine:** **[REPLACE: date, computer and browser on which you followed steps A and B from scratch]**

## AI Tools Used

| Tool | What it was used for |
|---|---|
| **Claude** (Anthropic), chat assistant | Wrote and refactored the application code (login and registration, roles, product and order CRUD, inventory rules, receipts, notifications and sounds, validation, admin sidebar, customer-view preview, reports); wrote the Firebase integration, the database security rules and the optional Cloud Function; wrote automated tests; helped find and fix bugs reported by the student; wrote the documentation. |

**The student's part:** **[REPLACE: describe in your own words what you did.]** The student chose the business and described every feature wanted, created the Firebase project and the GitHub repository, deployed the site, tested the live website, reported problems with screenshots, and decided what to change.

**Limits of the AI help:** the AI could not open the student's Firebase project or GitHub account, and its own tests of the online features used a simulated Firebase, so the real service must be checked by the student. The deployed application does not call any AI service; AI was used only to build it.

## Test Accounts

| Role | Mode | Username or email / password | Notes |
|---|---|---|---|
| Admin | Local mode (built in) | `admin` / `admin123` | Created automatically on first run. Change it in Settings. Demo password for local testing only. |
| Regular user | Local mode | `testcustomer` / `test1234` | Not preset: create it with **Create customer account** on the login page. |
| Admin | Online (live site) | `admin@gmail.com` / `Admin123` | The admin account created in Firebase (setup B, steps 4 and 5). |
| Regular user | Online (live site) | **[REPLACE: customer email]** / **[REPLACE: password]** | Create it with **Create customer account** on the live site. |

Passwords need at least 6 characters. For a test order use a contact number in the accepted format, for example `09175550123` (only the format is checked).

## Security and Validation Summary

- **Passwords hashed:** Firebase Authentication stores salted, hashed passwords; the application never sees or stores them. Local mode uses PBKDF2-SHA256 with a random salt.
- **Roles:** the role comes from the database (`admins` list); customers only read their own orders (`database.rules.json`).
- **Validation:** checked in the page and again when an order is saved.
- **Delete confirmation:** deleting a product (and cancelling an order, removing or deactivating a customer, resetting data) asks for confirmation first.
- **No secrets committed:** no private keys, no `.env` file and no API secrets. The Firebase web configuration is a public project identifier; access is controlled by Firebase Authentication and the database rules. The demo admin password for local mode is documented above. `.gitignore` excludes `node_modules` and private files.

## Testing and Known Limitations

During development the system was checked with 424 automated tests (370 browser tests with Playwright and 54 tests of the server order rules with Node.js). The online behaviour was tested against a simulated Firebase, not the live service. The automated tests are not part of this repository.

- In local mode, data is kept in one browser on one device.
- The contact-number check confirms the format only, not that the number is in service.
- Notifications and sounds work while the app is open; phone push, SMS and email to the owner are not included.
- Until the optional Cloud Function is deployed, ordering runs from the customer's browser (see `FIREBASE_SETUP.md`).
