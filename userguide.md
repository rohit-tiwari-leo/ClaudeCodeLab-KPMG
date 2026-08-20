# eShopOnWeb — User Guide

A step-by-step guide to setting up, running, and using the eShopOnWeb sample application (customer storefront + admin catalog management).

---

## 1. Prerequisites

- [.NET 8 SDK](https://dotnet.microsoft.com/download) (see `global.json` — 8.0.x)
- SQL Server LocalDB (installed with Visual Studio) **or** Docker Desktop
- Visual Studio 2022 (17.8+) *or* VS Code with the C# extension
- (Optional) [Azure Developer CLI (`azd`)](https://aka.ms/azure-dev/install) if deploying to Azure

---

## 2. Running the Application

Choose **one** of the three options below.

### Option A — Visual Studio (recommended for local development)

1. Open `eShopOnWeb.sln` in Visual Studio.
2. Set **Web** as the startup project (right-click `Web` → *Set as Startup Project*), or configure multiple startup projects (`Web` and `PublicApi`) via *Solution Properties → Startup Project* if you also want the API/Swagger running.
3. Press **F5** (or **Ctrl+F5** to run without debugging).
4. On first run, EF Core migrations apply automatically and the database is seeded with sample catalog data and demo users (see §3).
5. Your browser opens automatically to:
   - Web storefront: `https://localhost:5001`
   - Public API (if started): `https://localhost:5099/swagger`

### Option B — .NET CLI

```powershell
# From the repository root
cd src\Web
dotnet run
```

Then browse to `https://localhost:5001`.

To also run the API in a second terminal:

```powershell
cd src\PublicApi
dotnet run
```

Browse to `https://localhost:5099/swagger` for the API documentation.

### Option C — Docker Compose

```powershell
docker compose up --build
```

This starts three containers:

| Service | URL |
|---|---|
| Web storefront (`eshopwebmvc`) | http://localhost:5106 |
| Public API (`eshoppublicapi`) | http://localhost:5200 |
| SQL Server (Azure SQL Edge) | localhost:1433 |

---

## 3. Default Accounts

The database is seeded automatically with two accounts you can use immediately — no registration required:

| Role | Username | Password |
|---|---|---|
| Regular customer | `demouser@microsoft.com` | `Pass@word1` |
| Administrator | `admin@microsoft.com` | `Pass@word1` |

> You can also register a brand-new account from the site (see §5).

---

## 4. Storefront Walkthrough (Customer)

### 4.1 Browse the Catalog

1. Go to the home page (`https://localhost:5001`).
2. The catalog is displayed as a paged grid of products.
3. Use the **Brand** and **Type** dropdown filters at the top to narrow results.
4. Use the pagination controls at the bottom of the page to move between pages of results.

### 4.2 Add Items to the Basket

1. On any product card, click **Add to cart**.
2. A basket icon/badge in the top navigation updates with the item count.
3. You can add items to the basket **without logging in** — an anonymous basket is created and tied to your browser session via a cookie, then merged into your account basket once you log in.

### 4.3 View and Edit the Basket

1. Click the basket icon/link in the navigation bar to go to `/Basket`.
2. Here you can:
   - Update item quantities
   - Remove items
   - See the running subtotal
3. Click **Checkout** to proceed.

### 4.4 Log In (required to complete checkout)

1. If not already logged in, you'll be redirected to the login page.
2. Enter credentials (use the demo account from §3, or register — see §5).
3. On success, you're returned to the checkout flow with your basket intact.

### 4.5 Checkout

1. On the **Checkout** page, review your basket items.
2. Enter shipping/delivery details as prompted.
3. Enter payment details in the **credit card** section:
   - Card number (Luhn-valid test numbers work, e.g. `4111111111111111`)
   - Expiration month/year (must be a future date)
   - CVV (3–4 digits)
   - Cardholder name
4. Click **Place Order** / **Submit**.
5. The app simulates payment processing (`MockPaymentService`):
   - A valid card is **approved** and the order is placed.
   - The test card `4000000000000002` is always **declined** — use it to see the decline/error flow.
   - An invalid card number, expired date, or malformed CVV is rejected client-side/server-side with a validation message.
6. On approval, you're redirected to an order confirmation page.

### 4.6 View Order History

1. Click **My Orders** (or navigate to `/Order/MyOrders`) from the account menu.
2. See a list of all past orders with date and total.
3. Click into an individual order to see line items, quantities, prices, and shipping address (`/Order/Detail/{orderId}`).

### 4.7 Manage Your Account

From the account/profile menu you can:

1. **Change password** — Manage → Change Password.
2. **Set up Two-Factor Authentication (2FA)** — Manage → Two-Factor Authentication → follow the authenticator app enrollment (QR code) and save the generated recovery codes.
3. **Manage external logins** — Manage → External Logins (if any external providers are configured).
4. **Log out** — via the account menu.

---

## 5. Registering a New Account

1. From the login page, click **Register**.
2. Enter an email address and password (must meet ASP.NET Identity's default complexity rules).
3. Submit the form.
4. If email confirmation is enabled, confirm via the confirmation link (in local/dev mode this typically completes without a real email being sent).
5. You are now logged in as a standard customer and can shop immediately.

---

## 6. Admin Catalog Management (Blazor Admin)

The administration UI is a Blazor WebAssembly app hosted inside the `Web` project.

1. Log in with the **administrator** account (`admin@microsoft.com` / `Pass@word1`, or any account assigned the `Administrators` role).
2. Navigate to the **Admin** section (`/Admin` or the "Admin" link in the navigation, visible only to admins).
3. From the catalog item list you can:
   - **List** — browse all catalog items with paging.
   - **Create** — click *Add Item*, fill in name, description, price, brand, type, and upload a product image, then save.
   - **Edit** — click an item to modify its details.
   - **Details** — view full details of a single item.
   - **Delete** — remove an item from the catalog (with confirmation).
4. Changes made here call the **Public API** (`PublicApi` project) under the hood and are reflected immediately in the customer-facing catalog.

---

## 7. Using the Public API Directly

1. With `PublicApi` running, open `https://localhost:5099/swagger` (or `http://localhost:5200/swagger` under Docker).
2. **Authenticate** first:
   - Use the `Authenticate` endpoint under **AuthEndpoints**, passing a username/password (e.g. the admin demo account) to obtain a JWT.
   - Click **Authorize** in Swagger UI and paste the token as `Bearer {token}`.
3. Explore and call the available endpoints:
   - **CatalogItemEndpoints** — list (paged), get by ID, create, update, delete (create/update/delete require the Administrator role).
   - **CatalogBrandEndpoints** — list all brands.
   - **CatalogTypeEndpoints** — list all catalog types.
4. Responses are JSON; use the **Try it out** button in Swagger to execute requests interactively.

---

## 8. Running the Test Suites (optional, for developers)

```powershell
# Run all tests
dotnet test

# Run a specific suite
dotnet test tests\UnitTests\UnitTests.csproj
dotnet test tests\IntegrationTests\IntegrationTests.csproj
dotnet test tests\FunctionalTests\FunctionalTests.csproj
dotnet test tests\PublicApiIntegrationTests\PublicApiIntegrationTests.csproj
```

---

## 9. Troubleshooting

| Issue | Fix |
|---|---|
| Database connection fails on first run | Ensure SQL Server LocalDB is installed, or that Docker's `sqlserver` container is running and reachable on port 1433. |
| Login/registration errors | Check `ASPNETCORE_ENVIRONMENT=Development` is set so detailed errors are shown. |
| Checkout always fails | Confirm you're not accidentally using the decline test card `4000000000000002`; check the card isn't expired. |
| Admin section not visible | Confirm you're logged in with a user in the `Administrators` role (use the seeded `admin@microsoft.com` account). |
| API calls from Blazor Admin fail (CORS/401) | Confirm `PublicApi` is running and the `baseUrls.apiBase` setting in `src/Web/appsettings.json` matches its actual URL. |

---

## 10. Quick Reference — URLs & Ports

| Component | Local (VS/CLI) | Docker |
|---|---|---|
| Web storefront | https://localhost:5001 | http://localhost:5106 |
| Public API / Swagger | https://localhost:5099/swagger | http://localhost:5200/swagger |
| SQL Server | (localdb)\mssqllocaldb | localhost:1433 |
