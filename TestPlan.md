# eShopOnWeb — High-Level Test Plan

## 1. Purpose & Scope

This document describes the testing strategy for eShopOnWeb across all modules: `ApplicationCore` (domain), `Infrastructure` (data), `Web` (storefront), `PublicApi` (REST API), and `BlazorAdmin`/`BlazorShared` (admin UI). It covers what is tested today, how, at what level, and where the gaps are — as a baseline for ongoing development and for any future modernization work.

See [NewFeaturePlan.md](NewFeaturePlan.md) for feature-specific test plans (reviews, wishlist, order status) and the dependency graph behind module boundaries.

## 2. Test Levels & Existing Tooling

| Level | Project | Framework | What it exercises |
|---|---|---|---|
| Unit | [tests/UnitTests](tests/UnitTests) | xUnit + NSubstitute | `ApplicationCore` entities, services, specifications; a few `Web` extensions and MediatR handlers |
| Integration | [tests/IntegrationTests](tests/IntegrationTests) | xUnit + EF Core InMemory | `EfRepository<T>` + `CatalogContext` round-trips (real EF query translation, not mocked) |
| Functional / E2E-in-process | [tests/FunctionalTests](tests/FunctionalTests) | xUnit + `WebApplicationFactory` | `Web` Razor Pages/controllers and `PublicApi` auth flow, hosted in-memory |
| API contract | [tests/PublicApiIntegrationTests](tests/PublicApiIntegrationTests) | MSTest + `WebApplicationFactory` | `PublicApi` endpoints, role/JWT-gated authorization (200/401/403) |

No project currently does: browser-driven E2E (Selenium/Playwright), load/performance testing, or automated security scanning. These are gaps (Section 5).

### Shared test infrastructure to reuse

- **Builders** (`tests/UnitTests/Builders/`: `BasketBuilder.cs`, `OrderBuilder.cs`, `AddressBuilder.cs`) — fluent entity construction, reused by `IntegrationTests` via project reference. Any new aggregate should get its own builder here rather than constructing entities ad hoc in test bodies.
- **`WebTestFixture.cs`** (`tests/FunctionalTests/Web/`) — `TestApplication : WebApplicationFactory<IBasketViewModelService>`, swaps both `CatalogContext` and `AppIdentityDbContext` to InMemory.
- **`ApiTestFixture.cs`** / **`ApiTokenHelper.cs`** (`tests/FunctionalTests/PublicApi/`) — mints admin/normal-user JWTs for authorization tests; the same pattern is duplicated in `tests/PublicApiIntegrationTests/ApiTokenHelper.cs` (MSTest variant).

## 3. Coverage by Module

### 3.1 ApplicationCore (domain) — strongest coverage today

| Area | Tested via | Notes |
|---|---|---|
| `Basket` entity behavior | `UnitTests/ApplicationCore/Entities/BasketTests/*` | Add item, remove empty items, total items |
| `Order` entity behavior | `UnitTests/ApplicationCore/Entities/OrderTests/OrderTotal.cs` | Total calculation only — no status/lifecycle tests yet (see gap) |
| `BasketService` | `UnitTests/ApplicationCore/Services/BasketServiceTests/*` | Add item, delete basket, **anonymous→signed-in transfer** (`TransferBasket.cs`) |
| Specifications | `UnitTests/ApplicationCore/Specifications/*` | Evaluated against in-memory `List<T>`, not a real provider — validates the `Where`/`Include` predicate logic only |
| `OrderService` | *(none found)* | **Gap** — `CreateOrderAsync` has no unit test |
| Guard/extension helpers | `UnitTests/ApplicationCore/Extensions/*` | `JsonExtensions` |

**Strategy:** every entity behavior method (the `AddItem`/`RemoveItem`/`Update*` style methods with `Guard.Against` calls) should have a corresponding `[Fact]`/`[Theory]` pair — one happy path, one boundary/guard-clause path. This is the cheapest, fastest-running test tier and should carry the majority of the assertion count.

### 3.2 Infrastructure (data layer)

| Area | Tested via | Notes |
|---|---|---|
| `EfRepository<Basket>` | `IntegrationTests/Repositories/BasketRepositoryTests/SetQuantities.cs` | Real EF query translation against InMemory provider |
| `EfRepository<Order>` | `IntegrationTests/Repositories/OrderRepositoryTests/GetById.cs`, `GetByIdWithItemsAsync.cs` | |
| `CatalogContextSeed` / migrations | *(none)* | **Gap** — no test verifies a real SQL Server migration applies cleanly; only ever exercised via InMemory, which skips `Database.Migrate()` entirely |
| Query services (`BasketQueryService`) | *(none)* | **Gap** — no test for the DB-side `SumAsync` aggregation |

**Caveat to carry forward:** EF Core InMemory does not validate real SQL translation, constraint enforcement (unique indexes, FK cascade behavior), or provider-specific behavior (HiLo sequences, `decimal` precision). Anything relying on those needs either a LocalDB/SQL Server-backed integration test or manual verification against a real migration.

### 3.3 Web (storefront: Razor Pages + MVC)

| Area | Tested via | Notes |
|---|---|---|
| Home/catalog page | `FunctionalTests/Web/Pages/HomePageOnGet.cs` | |
| Basket page | `FunctionalTests/Web/Pages/Basket/IndexTest.cs`, `CheckoutTest.cs`, `BasketPageCheckout.cs` | |
| Sign-in | `FunctionalTests/Web/Controllers/AccountControllerSignIn.cs` | |
| Catalog controller | `FunctionalTests/Web/Controllers/CatalogControllerIndex.cs` | |
| Order controller | `FunctionalTests/Web/Controllers/OrderControllerIndex.cs` | Currently only checks the anonymous-redirect path — no signed-in assertion of order content |
| MediatR handlers (`GetMyOrders`, `GetOrderDetails`) | `UnitTests/MediatorHandlers/OrdersTests/*` | |
| Admin pages / Blazor host | *(none)* | **Gap** |
| Caching (`CachedCatalogViewModelService`) | `UnitTests/Web/Extensions/CacheHelpersTests/*` | Cache key generation only, not invalidation behavior |

### 3.4 PublicApi

| Area | Tested via | Notes |
|---|---|---|
| Authenticate endpoint | `FunctionalTests/PublicApi/AuthEndpoints/AuthenticateEndpoint.cs`, `PublicApiIntegrationTests/AuthEndpoints/AuthenticateEndpointTest.cs` | |
| Catalog item CRUD | `PublicApiIntegrationTests/CatalogItemEndpoints/*` (Get, ListPaged, Create, Delete) | Update endpoint has no dedicated test — **gap** |
| Role/JWT authorization | Covered per-endpoint (401/403/200 assertions) | Consistent pattern (`ApiTokenHelper.GetAdminUserToken()` / `GetNormalUserToken()`) — reuse for any new admin-only endpoint |

### 3.5 BlazorAdmin / BlazorShared

**No test coverage at all today.** This is the largest structural gap: the admin catalog-management UI (`List.razor`, `Create.razor`, `Edit.razor`, `Delete.razor`, `Details.razor`) and its `CachedCatalogItemServiceDecorator` have zero automated tests. Given this is the only interface for catalog administration, regressions here are currently caught by manual testing only.

## 4. Cross-Cutting Concerns

| Concern | Current state | Recommendation |
|---|---|---|
| **Authentication/Authorization** | Covered well at the API layer (401/403 per role); Web layer covered only for anonymous-redirect cases | Add signed-in-but-wrong-owner assertions — see the IDOR note below |
| **IDOR / ownership checks** | `GetOrderDetailsHandler` accepts a `UserName` but does not filter by it — any signed-in user can view another user's order by ID (see [NewFeaturePlan.md](NewFeaturePlan.md) Risk 12) | Add a regression test asserting cross-user order access is denied, once fixed |
| **Data integrity constraints** | Not exercised (InMemory ignores unique indexes/FKs) | Add at least one SQL Server (or Testcontainers) integration suite for constraint-dependent features |
| **Secrets** | `JWT_SECRET_KEY` and `DEFAULT_PASSWORD` are hardcoded constants in `AuthorizationConstants.cs` | Out of scope for test plan, but flagged — no test should assert on the literal secret value continuing to work |
| **Performance/Load** | None | Not currently required at this scale; revisit if traffic assumptions change or before a cloud migration cutover |
| **Browser-driven E2E** | None (all "functional" tests are in-process via `WebApplicationFactory`, not a real browser) | Consider Playwright for the 4-5 critical user journeys (browse → add to basket → checkout → view order) before any major release |

## 5. Gap Summary (priority order)

1. **BlazorAdmin has zero tests** — highest-risk gap given it's the sole admin interface.
2. **No test proves a real EF migration applies against SQL Server** — InMemory silently skips this; a broken migration could ship undetected.
3. **`OrderService.CreateOrderAsync`** (order placement — the core business transaction) has no unit test.
4. **Cross-user authorization gaps** in `Web` (the `GetOrderDetailsHandler` IDOR).
5. **No browser-driven E2E** for the primary purchase journey.
6. **No load/performance baseline** — relevant if this app is being sized for Azure before a migration decision.

## 6. Test Strategy Going Forward

- **Keep the pyramid shape**: most assertions in `UnitTests` (fast, no I/O), fewer in `IntegrationTests` (real EF, still fast via InMemory), fewer still in `FunctionalTests`/`PublicApiIntegrationTests` (full host boot), and a handful of true E2E if introduced.
- **New entity → new builder.** Every new aggregate (as in the reviews/wishlist/order-status plan) should get a `tests/UnitTests/Builders/<Entity>Builder.cs` mirroring `BasketBuilder`/`OrderBuilder`, referenced from both `UnitTests` and `IntegrationTests`.
- **New admin-only endpoint → reuse `ApiTokenHelper`** for the standard 401/403/200 triad; don't hand-roll JWTs per test.
- **New Razor Page → reuse `WebTestFixture.TestApplication`** rather than standing up a new `WebApplicationFactory`.
- **Before any Azure/modernization cutover**, close gaps #2 and #3 above at minimum — they're the ones most likely to hide a regression that only surfaces in production.

## 7. How to Run

```bash
dotnet test                                   # all four test projects
dotnet test tests/UnitTests                   # fastest feedback loop
dotnet test tests/IntegrationTests
dotnet test tests/FunctionalTests
dotnet test tests/PublicApiIntegrationTests
dotnet test --collect:"XPlat Code Coverage" --settings CodeCoverage.runsettings   # coverage report (repo already has a runsettings file)
```

Current test file count: 45 across all four projects (verified against the working tree at plan time).
