# eShopOnWeb — Dependency Analysis

## 1. Project Reference Graph

```
BlazorShared            (no project references)
        ^
        |
        +--------------------------------------------+
        |                                             |
ApplicationCore  <-----------------------+       BlazorAdmin
        ^                                |             ^
        |                                |             |
        +---------------+                |             |
        |                |               |             |
Infrastructure       PublicApi           |             |
        ^                ^               |             |
        |                |               |             |
        +----------------+---------------+-------------+
                          |
                         Web
```

**Direct `ProjectReference` edges (as declared in .csproj files):**

| Project | References |
|---|---|
| **BlazorShared** | *(none)* |
| **ApplicationCore** | BlazorShared |
| **Infrastructure** | ApplicationCore |
| **BlazorAdmin** | BlazorShared |
| **PublicApi** | ApplicationCore, Infrastructure |
| **Web** | ApplicationCore, Infrastructure, BlazorAdmin, BlazorShared |

**Resulting build order (topological):**
`BlazorShared → ApplicationCore → Infrastructure → { PublicApi, BlazorAdmin } → Web`

### ⚠️ Architectural anomaly

`ApplicationCore.csproj` carries a `ProjectReference` to **BlazorShared**, but no `.cs` file under `ApplicationCore` actually uses the `BlazorShared` namespace (0 matches). This is a **dangling/unused reference** and also an architectural smell: in Clean Architecture, `ApplicationCore` is meant to be the innermost layer with **no outward dependencies** on presentation/shared-UI concerns. `BlazorShared` (auth constants, DTOs for the admin API) is a presentation-adjacent contracts project and shouldn't be a dependency of the domain layer.

- **Actual effect:** because `ApplicationCore` pulls in `BlazorShared`, every downstream project (`Infrastructure`, `PublicApi`, `Web`) gets `BlazorShared` transitively — which is why `PublicApi.csproj` compiles even though it uses `BlazorShared.Authorization.Constants` and `BlazorShared.Models` without declaring its own `ProjectReference` to it.
- **Recommendation:** either remove the unused reference from `ApplicationCore` (if truly unused) or, if the intent was for `PublicApi` to use `BlazorShared` types, add an explicit `ProjectReference` in `PublicApi.csproj` rather than relying on the transitive leak through the domain layer.

---

## 2. Test Project Dependencies

| Test Project | References | Notes |
|---|---|---|
| **UnitTests** | ApplicationCore, Web | Web reference pulls in the full app graph transitively (ApplicationCore, Infrastructure, BlazorAdmin, BlazorShared) |
| **IntegrationTests** | Infrastructure, **UnitTests** | Depends on another test project (unusual but works for shared test builders/fixtures) |
| **FunctionalTests** | ApplicationCore, PublicApi, Web | Exercises the app end-to-end via `WebApplicationFactory` |
| **PublicApiIntegrationTests** | PublicApi, Web | MSTest-based, separate from the xUnit test suite |

`IntegrationTests → UnitTests` means changes to `UnitTests` test builders (e.g. `OrderBuilder`, `PaymentBuilder`) ripple into `IntegrationTests` compilation.

---

## 3. Runtime / Namespace-Level Dependencies (cross-project code usage)

Beyond project references, actual `using`/namespace usage confirms the runtime coupling:

- **Infrastructure → ApplicationCore**: 21 files (repositories, identity services, `MockPaymentService` implementing `IPaymentService`, EF configs mapping domain entities)
- **Web → ApplicationCore**: 17 files; **Web → Infrastructure**: 10 files; **Web → BlazorShared**: 5 files
- **PublicApi → ApplicationCore**: 11 files; **PublicApi → BlazorShared**: 5 files (transitive, see anomaly above); **PublicApi → Infrastructure**: 2 files
- **BlazorAdmin → BlazorShared**: 12 files (DTOs/interfaces shared between the Blazor client and the API)

This confirms `ApplicationCore` is the most depended-upon module (used directly by Infrastructure, Web, and PublicApi), consistent with its role as the domain core — aside from the outward BlazorShared reference noted above.

---

## 4. External NuGet Package Dependencies (centrally managed via `Directory.Packages.props`)

All package **versions** are pinned centrally (`ManagePackageVersionsCentrally = true`, target framework `net8.0`); each `.csproj` only declares which packages it uses.

### ApplicationCore
- Ardalis.GuardClauses, Ardalis.Result, Ardalis.Specification
- System.Security.Claims, System.Text.Json

### Infrastructure
- Ardalis.Specification.EntityFrameworkCore
- Microsoft.AspNetCore.Identity.EntityFrameworkCore
- Microsoft.EntityFrameworkCore.InMemory, Microsoft.EntityFrameworkCore.SqlServer
- System.IdentityModel.Tokens.Jwt

### Web
- Ardalis.ListStartupServices, Ardalis.Specification
- AutoMapper.Extensions.Microsoft.DependencyInjection
- Azure.Extensions.AspNetCore.Configuration.Secrets, Azure.Identity
- MediatR
- BuildBundlerMinifier (Release only), Microsoft.Web.LibraryManager.Build
- Microsoft.AspNetCore.Components.WebAssembly.Server (hosts the Blazor admin)
- Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore, Microsoft.EntityFrameworkCore.{InMemory,SqlServer,Tools}
- Microsoft.AspNetCore.Identity.{EntityFrameworkCore,UI}
- Microsoft.AspNetCore.Authentication.JwtBearer, System.IdentityModel.Tokens.Jwt
- Microsoft.Azure.AppConfiguration.AspNetCore, Microsoft.FeatureManagement.AspNetCore
- Microsoft.VisualStudio.Web.CodeGeneration.Design

### PublicApi
- Ardalis.ApiEndpoints, MinimalApi.Endpoint
- AutoMapper.Extensions.Microsoft.DependencyInjection
- Swashbuckle.AspNetCore(+SwaggerUI, +Annotations)
- Microsoft.AspNetCore.Authentication.JwtBearer, System.IdentityModel.Tokens.Jwt
- Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore
- Microsoft.AspNetCore.Identity.{EntityFrameworkCore,UI}
- Microsoft.EntityFrameworkCore.{InMemory,SqlServer,Tools}
- Microsoft.VisualStudio.Azure.Containers.Tools.Targets, Microsoft.VisualStudio.Web.CodeGeneration.Design

### BlazorAdmin
- Blazored.LocalStorage, BlazorInputFile
- Microsoft.AspNetCore.Components.{Authorization,WebAssembly,WebAssembly.Authentication,WebAssembly.DevServer}
- Microsoft.Extensions.Identity.Core, Microsoft.Extensions.Logging.Configuration
- System.Net.Http.Json

### BlazorShared
- BlazorInputFile, FluentValidation

### Test Projects
- **UnitTests**: Microsoft.NET.Test.Sdk, NSubstitute(+Analyzers), xunit(+runner.visualstudio, +runner.console)
- **IntegrationTests**: same xUnit/NSubstitute stack + Microsoft.EntityFrameworkCore.InMemory
- **FunctionalTests**: Microsoft.AspNetCore.Mvc.Testing, Microsoft.EntityFrameworkCore.InMemory, xunit stack
- **PublicApiIntegrationTests**: Microsoft.AspNetCore.Mvc.Testing, MSTest.TestAdapter/TestFramework, coverlet.collector *(the only project using MSTest instead of xUnit)*

**Cross-cutting/shared packages** appearing in multiple projects (version-consistency risk if central management were ever disabled): `Ardalis.Specification`, EF Core (`InMemory`/`SqlServer`/`Tools`), `Microsoft.AspNetCore.Identity.EntityFrameworkCore`, `Microsoft.AspNetCore.Authentication.JwtBearer`, `System.IdentityModel.Tokens.Jwt`, `Microsoft.EntityFrameworkCore.Diagnostics/Diagnostics.EntityFrameworkCore`.

---

## 5. Notable Dependency-Related Observations

1. **Dangling/leaky reference**: `ApplicationCore → BlazorShared` (see §1) — recommend cleanup.
2. **Inconsistent test framework**: `PublicApiIntegrationTests` uses MSTest while every other test project uses xUnit — a maintenance inconsistency, not a bug.
3. **Test-project chaining**: `IntegrationTests` depends on `UnitTests` rather than a shared `TestCommon`/`TestHelpers` project — couples two conceptually distinct test suites.
4. **Web is the composition root**: it references all four non-test src projects (`ApplicationCore`, `Infrastructure`, `BlazorAdmin`, `BlazorShared`) and hosts the Blazor WebAssembly admin app (`Microsoft.AspNetCore.Components.WebAssembly.Server`), meaning `Web` and `BlazorAdmin` are deployed together while `PublicApi` is a separate deployable.
5. **No circular project references** were found — the graph is a clean DAG aside from the questionable `ApplicationCore → BlazorShared` edge.
6. **New Payment feature (WIP on `Feature-01`)** introduces `IPaymentService`/`MockPaymentService` inside the existing `ApplicationCore`/`Infrastructure` boundary — no new project or package dependency was introduced for it.
