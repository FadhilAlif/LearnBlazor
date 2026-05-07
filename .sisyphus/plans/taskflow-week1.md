# TaskFlow Week 1 — Foundation & Auth

## TL;DR

> Build the foundational architecture and authentication layer for TaskFlow, a Blazor Server task management app. Week 1 establishes Clean Architecture project structure, PostgreSQL + EF Core with ASP.NET Identity, role-based authorization, Tailwind CSS pipeline, and working login/register pages.
>
> **Deliverables**:
> - 4-layer Clean Architecture solution (Domain, Application, Infrastructure, Web)
> - PostgreSQL database with EF Core migrations and Identity schema
> - ASP.NET Identity with custom ApplicationUser and 3 roles (Owner, Member, Viewer)
> - Tailwind CSS build pipeline integrated with Blazor
> - Login/Register pages with validation
> - Base layout with navigation and auth-aware UI
> - Health check endpoint with DB connectivity verification
> - Production-ready middleware and configuration
>
> **Estimated Effort**: Medium (~3-4 days of focused work)
> **Parallel Execution**: YES — 3 waves
> **Critical Path**: Task 1 → Task 2 → Task 3 → Task 7 → Task 12 → Task 15 → Task 18

---

## Context

### Original Request
Create a Task Manager App using .NET 8.0 Core + Blazor based on the provided PRD. Focus on Week 1 milestone: Foundation & Auth.

### Interview Summary
**Key Decisions**:
- **Scope**: Week 1 only (Foundation & Auth). No project/board/card features yet.
- **Database**: PostgreSQL (full setup, not SQLite)
- **Tests**: No automated tests for now
- **File Storage**: Local disk (wwwroot/uploads) — setup only, no upload UI yet
- **Goal**: Production-ready reference project for learning Blazor
- **Email**: Console/file logging for development (MailKit configured but not sending real emails)

**Research Findings**:
- Existing project is a fresh .NET 8 Blazor Web App template
- Current structure: `LearnBlazor.csproj`, `Program.cs`, basic `Components/` folder
- No packages beyond the template defaults installed

### Metis Review
**Identified Gaps** (addressed in this plan):
- Identity UI approach: Custom Blazor components (not scaffolded or built-in) to practice EditForm/DataAnnotations
- Role seeding: Seed 3 roles + default admin user for immediate policy testing
- Migration strategy: Manual `dotnet ef database update` (not auto-apply on startup)
- Tailwind toolchain: npm-based with package.json scripts
- Authorization scope: Global roles only for Week 1 (per-project permissions deferred to Week 2)
- "Production-ready" baseline: Secure defaults, config separation, health checks, no debug-only shortcuts

---

## Work Objectives

### Core Objective
Establish the foundational architecture, database layer, authentication system, and UI infrastructure for TaskFlow, enabling future feature development in subsequent weeks.

### Concrete Deliverables
- `TaskFlow.sln` with 4 projects: Domain, Application, Infrastructure, Web
- PostgreSQL database with EF Core migrations (Identity tables + custom schema)
- Custom ApplicationUser with ASP.NET Identity
- Role-based authorization with policies for Owner, Member, Viewer
- Tailwind CSS integrated with Blazor components
- Login, Register, and Logout Blazor pages
- MainLayout with navigation and auth state
- Health check endpoint (`/health`) with DB connectivity
- Production-ready middleware pipeline and configuration

### Definition of Done
- [ ] Solution builds without errors: `dotnet build` → exit code 0
- [ ] Migrations apply successfully: `dotnet ef database update` → success
- [ ] App starts and health check returns 200: `curl /health` → HTTP 200
- [ ] User can register and login via Blazor pages
- [ ] Protected page redirects anonymous users to login
- [ ] Role-based access control returns 403 for unauthorized roles
- [ ] Tailwind CSS is generated and served

### Must Have
- Clean Architecture with 4 projects
- PostgreSQL + EF Core with working migrations
- ASP.NET Identity with custom user entity
- Role seeding (Owner, Member, Viewer)
- Tailwind CSS pipeline
- Login/Register pages
- Health check endpoint
- Production-ready config separation

### Must NOT Have (Guardrails)
- No project/board/card CRUD (Week 2+)
- No SignalR (Week 3)
- No file upload UI (Week 4)
- No email sending infrastructure beyond dev logging
- No admin/role-management UI
- No per-project permissions (global roles only for Week 1)
- No external OAuth providers
- No 2FA, password reset, email confirmation flows
- No MediatR, CQRS, or repository abstractions
- No Docker, CI/CD, or deployment scripts
- No automated tests (explicitly excluded)

---

## Verification Strategy

> **ZERO HUMAN INTERVENTION** — ALL verification is agent-executed via commands.

### Test Decision
- **Infrastructure exists**: NO (not setting up test infrastructure per user request)
- **Automated tests**: None
- **Framework**: N/A
- **Agent-Executed QA**: ALL tasks include agent-executable QA scenarios using `dotnet`, `ef`, `npm`, `curl`, and `bash` commands.

### QA Policy
Every task MUST include agent-executed QA scenarios. Evidence saved to `.sisyphus/evidence/task-{N}-{scenario-slug}.{ext}`.

- **Build verification**: `dotnet build`
- **Database verification**: `dotnet ef migrations list`, `dotnet ef database update`
- **API verification**: `curl` for health checks and auth endpoints
- **Frontend verification**: `curl` for page accessibility and CSS serving

---

## Execution Strategy

### Parallel Execution Waves

```
Wave 1 (Foundation — can all start immediately):
├── Task 1: Clean Architecture project structure
├── Task 2: PostgreSQL + EF Core setup + DbContext
├── Task 3: Tailwind CSS pipeline setup
└── Task 4: appsettings configuration + User Secrets

Wave 2 (Auth Core — depends on Wave 1):
├── Task 5: ASP.NET Identity + custom ApplicationUser
├── Task 6: Role seeding + default admin user
├── Task 7: Authorization policies + claims
└── Task 8: Identity UI components (Login, Register)

Wave 3 (UI & Integration — depends on Wave 2):
├── Task 9: MainLayout + navigation with auth state
├── Task 10: Landing page + protected pages
├── Task 11: Health check endpoint with DB check
└── Task 12: Middleware pipeline + production config

Wave FINAL (Verification & Review):
├── Task F1: Plan compliance audit (oracle)
├── Task F2: Real manual QA (unspecified-high)
└── Task F3: Scope fidelity check (deep)
```

### Dependency Matrix

- **Task 1**: None → Blocks: Task 2, Task 3, Task 4
- **Task 2**: Task 1 → Blocks: Task 5, Task 11
- **Task 3**: Task 1 → Blocks: Task 9
- **Task 4**: Task 1 → Blocks: Task 11, Task 12
- **Task 5**: Task 2 → Blocks: Task 6, Task 7, Task 8
- **Task 6**: Task 5 → Blocks: Task 7, Task 10
- **Task 7**: Task 5, Task 6 → Blocks: Task 10
- **Task 8**: Task 5 → Blocks: Task 9
- **Task 9**: Task 3, Task 8 → Blocks: None
- **Task 10**: Task 6, Task 7 → Blocks: None
- **Task 11**: Task 2, Task 4 → Blocks: None
- **Task 12**: Task 4 → Blocks: None

---

## TODOs

- [ ] 1. Clean Architecture Project Structure

  **What to do**:
  - Restructure the solution from single-project to 4-layer Clean Architecture:
    - `TaskFlow.Domain` — Class library, zero dependencies. Contains entities, enums, interfaces.
    - `TaskFlow.Application` — Class library, depends on Domain. Contains DTOs, service interfaces, use case abstractions.
    - `TaskFlow.Infrastructure` — Class library, depends on Domain + Application. Contains EF Core DbContext, Identity stores, repositories, email/file service implementations.
    - `TaskFlow.Web` — Web project (existing `LearnBlazor` renamed or restructured), depends on Infrastructure. Contains Blazor components, pages, Hubs, middleware, wwwroot.
  - Move existing `Components/`, `wwwroot/`, `Program.cs` into Web project.
  - Update `TaskFlow.sln` with all 4 projects.
  - Set up project references: Web → Infrastructure → Application → Domain.
  - Add `<ProjectReference>` entries in `.csproj` files.

  **Must NOT do**:
  - Do NOT add actual entity classes yet (those go in Task 2).
  - Do NOT add NuGet packages yet (those go in subsequent tasks).
  - Do NOT modify Program.cs beyond renaming namespaces.
  - Do NOT create MediatR handlers, CQRS classes, or generic repositories.

  **Recommended Agent Profile**:
  - **Category**: `quick`
    - Reason: File creation, project setup, and reference configuration are straightforward tasks.
  - **Skills**: []

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 1 (with Tasks 2, 3, 4)
  - **Blocks**: Task 2, Task 3, Task 4
  - **Blocked By**: None

  **References**:
  - Existing project: `LearnBlazor.csproj`, `Program.cs`, `Components/` — baseline to preserve
  - PRD Section 5.2: Clean Architecture folder structure pattern
  - .NET docs: Class library project creation (`dotnet new classlib`)

  **Acceptance Criteria**:
  - [ ] `TaskFlow.sln` references all 4 projects
  - [ ] `dotnet build` succeeds with 0 errors
  - [ ] `TaskFlow.Domain.csproj` has zero `<ProjectReference>`
  - [ ] `TaskFlow.Application.csproj` references only Domain
  - [ ] `TaskFlow.Infrastructure.csproj` references Domain + Application
  - [ ] `TaskFlow.Web.csproj` references Infrastructure
  - [ ] Existing Components/App.razor still compiles

  **QA Scenarios**:

  ```
  Scenario: Solution builds successfully
    Tool: Bash
    Preconditions: All 4 projects created with correct references
    Steps:
      1. Run: dotnet build TaskFlow.sln
      2. Assert: exit code 0, 0 errors, 0 warnings
    Expected Result: Build succeeds
    Evidence: .sisyphus/evidence/task-1-build-success.txt
  ```

  **Commit**: YES
  - Message: `feat(arch): Clean Architecture project structure`
  - Files: All `.csproj`, `.sln`, moved files

- [ ] 2. PostgreSQL + EF Core Setup + DbContext

  **What to do**:
  - Add NuGet packages to Infrastructure project:
    - `Npgsql.EntityFrameworkCore.PostgreSQL` (v8.x)
    - `Microsoft.AspNetCore.Identity.EntityFrameworkCore` (v8.x)
    - `Microsoft.EntityFrameworkCore.Tools` (v8.x, for migrations)
  - Create `TaskFlowDbContext` in Infrastructure that inherits from `IdentityDbContext<ApplicationUser>`.
    - Placeholder: `ApplicationUser` class will be defined in Task 5.
    - For now, create a stub or forward-reference the class location.
  - Configure `TaskFlowDbContext` with PostgreSQL provider in `OnConfiguring` or via `DbContextOptions`.
  - Create initial migration: `InitialCreate` that sets up Identity tables.
  - Configure connection string in `appsettings.Development.json`:
    ```json
    "ConnectionStrings": {
      "DefaultConnection": "Host=localhost;Database=taskflow;Username=postgres;Password=postgres"
    }
    ```
  - Register `DbContext` in `Program.cs`:
    ```csharp
    builder.Services.AddDbContext<TaskFlowDbContext>(options =>
        options.UseNpgsql(connectionString));
    ```
  - Install `dotnet-ef` tool globally if not present.

  **Must NOT do**:
  - Do NOT create custom entity classes beyond the Identity user stub.
  - Do NOT add Project, Board, Card, or other domain entities yet.
  - Do NOT auto-apply migrations on startup (manual only).
  - Do NOT use SQLite (user explicitly chose PostgreSQL).

  **Recommended Agent Profile**:
  - **Category**: `quick`
    - Reason: Package installation, DbContext creation, and migration setup are standard operations.
  - **Skills**: []

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 1 (with Tasks 1, 3, 4)
  - **Blocks**: Task 5, Task 11
  - **Blocked By**: Task 1 (Infrastructure project must exist)

  **References**:
  - PRD Section 5.1: Tech stack specifies EF Core 8 + PostgreSQL
  - PRD Section 5.3: Data model shows Identity User entity
  - Npgsql EF Core docs: `https://www.npgsql.org/efcore/`

  **Acceptance Criteria**:
  - [ ] `dotnet ef migrations list` shows `InitialCreate`
  - [ ] `dotnet ef database update` applies successfully
  - [ ] PostgreSQL database `taskflow` exists with Identity tables (`AspNetUsers`, `AspNetRoles`, etc.)
  - [ ] `TaskFlowDbContext` is registered in DI container

  **QA Scenarios**:

  ```
  Scenario: EF migration creates and applies
    Tool: Bash
    Preconditions: PostgreSQL running locally, connection string valid
    Steps:
      1. Run: dotnet ef migrations add InitialCreate --project TaskFlow.Infrastructure --startup-project TaskFlow.Web
      2. Run: dotnet ef database update --project TaskFlow.Infrastructure --startup-project TaskFlow.Web
      3. Connect to PostgreSQL and query: SELECT tablename FROM pg_tables WHERE schemaname='public';
      4. Assert: Tables include AspNetUsers, AspNetRoles, AspNetUserRoles, etc.
    Expected Result: Migration created and applied, Identity tables exist
    Failure Indicators: Migration fails, tables missing, connection error
    Evidence: .sisyphus/evidence/task-2-migration-success.txt
  ```

  **Commit**: YES (groups with Task 1)
  - Message: `feat(db): PostgreSQL + EF Core with Identity migration`

- [ ] 3. Tailwind CSS Pipeline Setup

  **What to do**:
  - Initialize npm in the Web project root:
    - `npm init -y`
  - Install Tailwind CSS and dependencies:
    - `npm install -D tailwindcss postcss autoprefixer`
  - Create `tailwind.config.js` with content paths pointing to `.razor` files:
    ```js
    module.exports = {
      content: [
        "./Components/**/*.razor",
        "./Components/**/*.cshtml",
        "./wwwroot/**/*.html"
      ],
      theme: { extend: {} },
      plugins: []
    }
    ```
  - Create `postcss.config.js`:
    ```js
    module.exports = {
      plugins: {
        tailwindcss: {},
        autoprefixer: {},
      }
    }
    ```
  - Create `wwwroot/css/app.css` with Tailwind directives:
    ```css
    @tailwind base;
    @tailwind components;
    @tailwind utilities;
    ```
  - Add npm scripts to `package.json`:
    ```json
    "scripts": {
      "build:css": "tailwindcss -i ./wwwroot/css/app.css -o ./wwwroot/css/app.min.css --minify",
      "watch:css": "tailwindcss -i ./wwwroot/css/app.css -o ./wwwroot/css/app.min.css --watch"
    }
    ```
  - Reference `app.min.css` in `App.razor` (or `index.html` if present) via `<link rel="stylesheet" href="css/app.min.css" />`.
  - Run `npm run build:css` to generate the CSS file.
  - Ensure `.gitignore` ignores `node_modules/` and `wwwroot/css/app.min.css` (generated file).

  **Must NOT do**:
  - Do NOT create custom Tailwind plugins or themes yet.
  - Do NOT add `@tailwindcss/forms` or other plugins yet.
  - Do NOT modify component styles beyond ensuring the CSS file is loaded.
  - Do NOT use the Tailwind CDN (must be build pipeline).

  **Recommended Agent Profile**:
  - **Category**: `quick`
    - Reason: npm initialization, Tailwind config, and build scripts are standard frontend tooling.
  - **Skills**: []

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 1 (with Tasks 1, 2, 4)
  - **Blocks**: Task 9
  - **Blocked By**: Task 1 (Web project must exist)

  **References**:
  - PRD Section 5.1: Tailwind CSS + CSS Isolation specified
  - PRD Section 4.1: Login/Register forms need styling
  - Tailwind docs: `https://tailwindcss.com/docs/installation`

  **Acceptance Criteria**:
  - [ ] `npm run build:css` executes without errors
  - [ ] `wwwroot/css/app.min.css` is generated and > 10KB
  - [ ] CSS file contains Tailwind utility classes (e.g., search for `.flex`, `.bg-blue-500`)
  - [ ] `App.razor` references the generated CSS file

  **QA Scenarios**:

  ```
  Scenario: Tailwind CSS builds and is served
    Tool: Bash
    Preconditions: npm packages installed
    Steps:
      1. Run: npm run build:css
      2. Assert: Exit code 0, wwwroot/css/app.min.css exists
      3. Run: grep -c "flex" wwwroot/css/app.min.css
      4. Assert: Count > 0 (Tailwind utilities present)
      5. Start app: dotnet run --project TaskFlow.Web
      6. Run: curl -I http://localhost:5000/css/app.min.css
      7. Assert: HTTP 200, Content-Length > 10000
    Expected Result: Tailwind CSS generated, accessible via HTTP
    Failure Indicators: Build fails, file missing, HTTP 404
    Evidence: .sisyphus/evidence/task-3-tailwind-success.txt
  ```

  **Commit**: YES (groups with Task 1)
  - Message: `feat(ui): Tailwind CSS build pipeline`

- [ ] 4. appsettings Configuration + User Secrets

  **What to do**:
  - Create `appsettings.Development.json` in Web project with:
    - Connection string for PostgreSQL (localhost, dev credentials)
    - Logging configuration
    - SignalR placeholder config (for future use)
    - Storage placeholder config
    - Email placeholder config (SMTP localhost:1025 for dev)
  - Create `appsettings.Production.json` with:
    - No connection string (will come from env vars)
    - Reduced logging
    - HSTS enabled
  - Set up User Secrets for sensitive dev config:
    - `dotnet user-secrets init --project TaskFlow.Web`
    - `dotnet user-secrets set "ConnectionStrings:DefaultConnection" "Host=localhost;Database=taskflow;Username=postgres;Password=postgres" --project TaskFlow.Web`
  - Ensure `appsettings.Development.json` connection string is a fallback, not the primary secret source.
  - Add configuration validation in `Program.cs` to fail fast if required config is missing in Production.

  **Must NOT do**:
  - Do NOT commit real passwords or secrets to `appsettings.json`.
  - Do NOT hardcode connection strings in `Program.cs`.
  - Do NOT add production secrets to any checked-in file.

  **Recommended Agent Profile**:
  - **Category**: `quick`
    - Reason: Configuration files and secrets setup are straightforward.
  - **Skills**: []

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 1 (with Tasks 1, 2, 3)
  - **Blocks**: Task 11, Task 12
  - **Blocked By**: Task 1 (Web project must exist)

  **References**:
  - PRD Section 10.3: Environment & Configuration example
  - .NET docs: Safe storage of app secrets in development

  **Acceptance Criteria**:
  - [ ] `appsettings.Development.json` exists with all required sections
  - [ ] `appsettings.Production.json` exists with secure defaults
  - [ ] `dotnet user-secrets list --project TaskFlow.Web` shows connection string
  - [ ] App reads config correctly in Development mode

  **QA Scenarios**:

  ```
  Scenario: Configuration loads correctly in Development
    Tool: Bash
    Preconditions: User secrets initialized
    Steps:
      1. Run: dotnet user-secrets list --project TaskFlow.Web
      2. Assert: Output contains "ConnectionStrings:DefaultConnection"
      3. Run: ASPNETCORE_ENVIRONMENT=Development dotnet run --project TaskFlow.Web &
      4. Wait for app to start
      5. Run: curl -i http://localhost:5000/health
      6. Assert: HTTP 200 (proves config loaded)
    Expected Result: Config loads, app starts, health check passes
    Evidence: .sisyphus/evidence/task-4-config-success.txt
  ```

  **Commit**: YES (groups with Task 1)
  - Message: `feat(config): appsettings + User Secrets for dev/prod`

- [ ] 5. ASP.NET Identity + Custom ApplicationUser

  **What to do**:
  - Create `ApplicationUser` class in `TaskFlow.Domain/Entities/` that inherits from `IdentityUser`:
    ```csharp
    public class ApplicationUser : IdentityUser
    {
        public string DisplayName { get; set; } = string.Empty;
        public string? AvatarUrl { get; set; }
        public DateTime CreatedAt { get; set; } = DateTime.UtcNow;
    }
    ```
  - Update `TaskFlowDbContext` to inherit from `IdentityDbContext<ApplicationUser>`.
  - Add `DbSet<ApplicationUser>` property to `TaskFlowDbContext`.
  - Configure `ApplicationUser` entity in `OnModelCreating` (optional, but good practice for column lengths).
  - Register Identity services in `Program.cs`:
    ```csharp
    builder.Services.AddIdentity<ApplicationUser, IdentityRole>(options => {
        options.Password.RequireDigit = true;
        options.Password.RequireLowercase = true;
        options.Password.RequireUppercase = true;
        options.Password.RequireNonAlphanumeric = true;
        options.Password.RequiredLength = 8;
        options.User.RequireUniqueEmail = true;
    })
    .AddEntityFrameworkStores<TaskFlowDbContext>()
    .AddDefaultTokenProviders();
    ```
  - Add `AddAuthentication()` and `AddCookie()` configuration.
  - Create a new migration for the custom user fields: `dotnet ef migrations add CustomUserFields`.
  - Update database: `dotnet ef database update`.

  **Must NOT do**:
  - Do NOT add external login providers (Google, GitHub, etc.).
  - Do NOT implement 2FA, email confirmation, or password reset.
  - Do NOT add custom claims or complex profile management.

  **Recommended Agent Profile**:
  - **Category**: `unspecified-high`
    - Reason: Identity configuration requires careful setup to avoid auth footguns.
  - **Skills**: []

  **Parallelization**:
  - **Can Run In Parallel**: NO (depends on Task 2)
  - **Parallel Group**: Wave 2 (sequential within wave)
  - **Blocks**: Task 6, Task 7, Task 8
  - **Blocked By**: Task 2 (DbContext must exist)

  **References**:
  - PRD Section 4.1: Auth module requirements
  - PRD Section 5.3: User entity definition
  - .NET docs: `https://learn.microsoft.com/en-us/aspnet/core/security/authentication/identity`

  **Acceptance Criteria**:
  - [ ] `ApplicationUser` class exists with custom fields
  - [ ] `TaskFlowDbContext` inherits from `IdentityDbContext<ApplicationUser>`
  - [ ] Identity services registered in DI
  - [ ] Migration `CustomUserFields` created and applied
  - [ ] PostgreSQL `AspNetUsers` table has `DisplayName`, `AvatarUrl`, `CreatedAt` columns

  **QA Scenarios**:

  ```
  Scenario: Identity tables exist with custom fields
    Tool: Bash
    Preconditions: PostgreSQL running, migrations applied
    Steps:
      1. Run: psql -U postgres -d taskflow -c "\d AspNetUsers"
      2. Assert: Output contains DisplayName, AvatarUrl, CreatedAt columns
    Expected Result: Custom user fields present in database
    Evidence: .sisyphus/evidence/task-5-identity-tables.txt
  ```

  **Commit**: YES (groups with Task 5-8)
  - Message: `feat(auth): ASP.NET Identity with custom ApplicationUser`

- [ ] 6. Role Seeding + Default Admin User

  **What to do**:
  - Create `RoleSeeder` class in Infrastructure that seeds the 3 roles:
    ```csharp
    public static class RoleSeeder
    {
        public static async Task SeedRolesAsync(RoleManager<IdentityRole> roleManager)
        {
            string[] roles = { "Owner", "Member", "Viewer" };
            foreach (var role in roles)
            {
                if (!await roleManager.RoleExistsAsync(role))
                    await roleManager.CreateAsync(new IdentityRole(role));
            }
        }
    }
    ```
  - Create `UserSeeder` that creates a default admin user:
    ```csharp
    public static class UserSeeder
    {
        public static async Task SeedAdminUserAsync(UserManager<ApplicationUser> userManager)
        {
            var adminEmail = "admin@taskflow.local";
            if (await userManager.FindByEmailAsync(adminEmail) == null)
            {
                var user = new ApplicationUser
                {
                    UserName = adminEmail,
                    Email = adminEmail,
                    DisplayName = "System Admin",
                    EmailConfirmed = true
                };
                await userManager.CreateAsync(user, "Admin123!");
                await userManager.AddToRoleAsync(user, "Owner");
            }
        }
    }
    ```
  - Create an `IDbSeeder` interface and implementation.
  - Register seeders in DI and execute during app startup (dev environment only).
  - Add a static helper method in `Program.cs`:
    ```csharp
    if (app.Environment.IsDevelopment())
    {
        using var scope = app.Services.CreateScope();
        var roleManager = scope.ServiceProvider.GetRequiredService<RoleManager<IdentityRole>>();
        var userManager = scope.ServiceProvider.GetRequiredService<UserManager<ApplicationUser>>();
        await RoleSeeder.SeedRolesAsync(roleManager);
        await UserSeeder.SeedAdminUserAsync(userManager);
    }
    ```

  **Must NOT do**:
  - Do NOT seed in Production environment.
  - Do NOT create a role management UI.
  - Do NOT add per-project permissions (global roles only for Week 1).

  **Recommended Agent Profile**:
  - **Category**: `quick`
    - Reason: Seeding logic is straightforward but critical for testing.
  - **Skills**: []

  **Parallelization**:
  - **Can Run In Parallel**: NO (depends on Task 5)
  - **Parallel Group**: Wave 2
  - **Blocks**: Task 7, Task 10
  - **Blocked By**: Task 5 (Identity must be configured)

  **References**:
  - PRD Section 2.3: Target users (Owner, Member, Viewer)
  - PRD Section 4.1: Role-based access
  - .NET docs: RoleManager and UserManager usage

  **Acceptance Criteria**:
  - [ ] 3 roles exist in `AspNetRoles` table after startup
  - [ ] Default admin user exists in `AspNetUsers` table
  - [ ] Admin user has "Owner" role in `AspNetUserRoles`
  - [ ] Seeding only runs in Development environment

  **QA Scenarios**:

  ```
  Scenario: Roles and admin user seeded
    Tool: Bash
    Preconditions: App started in Development mode
    Steps:
      1. Run: psql -U postgres -d taskflow -c "SELECT Name FROM AspNetRoles"
      2. Assert: Output contains Owner, Member, Viewer
      3. Run: psql -U postgres -d taskflow -c "SELECT Email FROM AspNetUsers WHERE Email='admin@taskflow.local'"
      4. Assert: Output contains admin@taskflow.local
      5. Run: psql -U postgres -d taskflow -c "SELECT r.Name FROM AspNetRoles r JOIN AspNetUserRoles ur ON r.Id = ur.RoleId JOIN AspNetUsers u ON ur.UserId = u.Id WHERE u.Email='admin@taskflow.local'"
      6. Assert: Output contains Owner
    Expected Result: All roles and admin user seeded correctly
    Evidence: .sisyphus/evidence/task-6-seeding-success.txt
  ```

  **Commit**: YES (groups with Task 5-8)
  - Message: `feat(auth): Seed roles and default admin user`

- [ ] 7. Authorization Policies + Claims

  **What to do**:
  - Create `Policies.cs` in Web project (or Application) with policy definitions:
    ```csharp
    public static class Policies
    {
        public const string RequireOwner = "RequireOwner";
        public const string RequireMember = "RequireMember";
        public const string RequireViewer = "RequireViewer";

        public static void Configure(AuthorizationOptions options)
        {
            options.AddPolicy(RequireOwner, policy => policy.RequireRole("Owner"));
            options.AddPolicy(RequireMember, policy => policy.RequireRole("Owner", "Member"));
            options.AddPolicy(RequireViewer, policy => policy.RequireRole("Owner", "Member", "Viewer"));
        }
    }
    ```
  - Register policies in `Program.cs`:
    ```csharp
    builder.Services.AddAuthorization(options => Policies.Configure(options));
    ```
  - Create a test page `/admin-only` that requires `RequireOwner` policy.
  - Create a test page `/member-area` that requires `RequireMember` policy.
  - Ensure `[Authorize]` attribute works with roles on Blazor pages.
  - Add `CascadingAuthenticationState` and `AuthorizeRouteView` in `Routes.razor`.

  **Must NOT do**:
  - Do NOT implement per-project permissions (deferred to Week 2).
  - Do NOT add complex claim transformations.
  - Do NOT add resource-based authorization (e.g., "can only edit own projects").

  **Recommended Agent Profile**:
  - **Category**: `unspecified-high`
    - Reason: Authorization misconfiguration can lead to security holes.
  - **Skills**: []

  **Parallelization**:
  - **Can Run In Parallel**: NO (depends on Tasks 5, 6)
  - **Parallel Group**: Wave 2
  - **Blocks**: Task 10
  - **Blocked By**: Task 5 (Identity configured), Task 6 (roles exist)

  **References**:
  - PRD Section 4.1: Role-based access
  - PRD Section 10.1: AuthorizeView, IAuthorizationService, Policy
  - .NET docs: `https://learn.microsoft.com/en-us/aspnet/core/security/authorization/policies`

  **Acceptance Criteria**:
  - [ ] 3 authorization policies defined and registered
  - [ ] Protected page returns 302/403 for anonymous users
  - [ ] Protected page returns 403 for users without required role
  - [ ] Protected page returns 200 for users with required role

  **QA Scenarios**:

  ```
  Scenario: Anonymous user redirected from protected page
    Tool: Bash (curl)
    Preconditions: App running
    Steps:
      1. Run: curl -i http://localhost:5000/admin-only
      2. Assert: HTTP 302 with Location header pointing to login
    Expected Result: Anonymous user redirected to login
    Evidence: .sisyphus/evidence/task-7-auth-anonymous.txt

  Scenario: Role enforcement works
    Tool: Bash (curl with cookies)
    Preconditions: Admin user exists, app running
    Steps:
      1. Login as admin (Owner role): curl -c cookies.txt -d "Input.Email=admin@taskflow.local&Input.Password=Admin123!" http://localhost:5000/account/login
      2. Run: curl -b cookies.txt -i http://localhost:5000/admin-only
      3. Assert: HTTP 200
      4. Register a new user without Owner role
      5. Login as non-owner: curl -c cookies2.txt -d "Input.Email=user@example.com&Input.Password=User123!" http://localhost:5000/account/login
      6. Run: curl -b cookies2.txt -i http://localhost:5000/admin-only
      7. Assert: HTTP 403
    Expected Result: Owner can access, non-owner gets 403
    Evidence: .sisyphus/evidence/task-7-auth-roles.txt
  ```

  **Commit**: YES (groups with Task 5-8)
  - Message: `feat(auth): Authorization policies for Owner, Member, Viewer`

- [ ] 8. Identity UI Components (Login, Register, Logout)

  **What to do**:
  - Create `LoginPage.razor` in `Components/Pages/Account/`:
    - Use `EditForm` with `InputModel` class
    - Fields: Email, Password, RememberMe
    - Use `SignInManager<ApplicationUser>` for authentication
    - Redirect to returnUrl or dashboard on success
    - Show validation errors
  - Create `RegisterPage.razor` in `Components/Pages/Account/`:
    - Use `EditForm` with `InputModel` class
    - Fields: Email, Password, ConfirmPassword, DisplayName
    - Use `UserManager<ApplicationUser>` to create user
    - Auto-confirm email (for dev): `EmailConfirmed = true`
    - Redirect to login on success
    - Show validation errors (duplicate email, password policy)
  - Create `Logout.razor` page that calls `SignInManager.SignOutAsync()`.
  - Add route attributes: `@page "/account/login"`, `@page "/account/register"`, `@page "/account/logout"`.
  - Add `DataAnnotations` validation attributes on input models.
  - Ensure antiforgery tokens are handled (Blazor Server handles this automatically with `EditForm`).
  - Style forms with Tailwind CSS classes.

  **Must NOT do**:
  - Do NOT implement email confirmation flow.
  - Do NOT implement password reset or forgot password.
  - Do NOT implement external login providers.
  - Do NOT implement profile management or avatar upload.

  **Recommended Agent Profile**:
  - **Category**: `unspecified-high`
    - Reason: Auth UI requires careful handling of SignInManager, validation, and redirects.
  - **Skills**: []

  **Parallelization**:
  - **Can Run In Parallel**: NO (depends on Task 5)
  - **Parallel Group**: Wave 2
  - **Blocks**: Task 9
  - **Blocked By**: Task 5 (Identity configured)

  **References**:
  - PRD Section 4.1: Auth module requirements (AUTH-01 to AUTH-04)
  - PRD Section 10.1: Blazor Concepts Coverage Map (Login & Register)
  - .NET docs: `https://learn.microsoft.com/en-us/aspnet/core/blazor/security/server/`

  **Acceptance Criteria**:
  - [ ] Login page accessible at `/account/login`
  - [ ] Register page accessible at `/account/register`
  - [ ] Registration creates user in database
  - [ ] Login authenticates user and sets auth cookie
  - [ ] Logout clears auth cookie and redirects
  - [ ] Validation works (empty fields, invalid email, password mismatch)

  **QA Scenarios**:

  ```
  Scenario: User registration works
    Tool: Bash (curl)
    Preconditions: App running, database accessible
    Steps:
      1. Run: curl -i -c cookies.txt -d "Input.Email=test@example.com&Input.Password=Test123!&Input.ConfirmPassword=Test123!&Input.DisplayName=Test User" http://localhost:5000/account/register
      2. Assert: HTTP 302 redirect (or 200 with success indicator)
      3. Run: psql -U postgres -d taskflow -c "SELECT Email FROM AspNetUsers WHERE Email='test@example.com'"
      4. Assert: Output contains test@example.com
    Expected Result: User created successfully
    Evidence: .sisyphus/evidence/task-8-register-success.txt

  Scenario: User login works
    Tool: Bash (curl)
    Preconditions: User registered
    Steps:
      1. Run: curl -i -c cookies.txt -d "Input.Email=test@example.com&Input.Password=Test123!" http://localhost:5000/account/login
      2. Assert: HTTP 302 redirect with auth cookie set
      3. Run: curl -b cookies.txt -i http://localhost:5000/dashboard
      4. Assert: HTTP 200 (not redirected to login)
    Expected Result: Login succeeds, user authenticated
    Evidence: .sisyphus/evidence/task-8-login-success.txt

  Scenario: Invalid login rejected
    Tool: Bash (curl)
    Preconditions: App running
    Steps:
      1. Run: curl -i -c cookies.txt -d "Input.Email=nonexistent@example.com&Input.Password=Wrong123!" http://localhost:5000/account/login
      2. Assert: Response contains error message (e.g., "Invalid login attempt")
    Expected Result: Login fails with clear error
    Evidence: .sisyphus/evidence/task-8-login-fail.txt
  ```

  **Commit**: YES (groups with Task 5-8)
  - Message: `feat(auth): Login, Register, Logout Blazor pages`

- [ ] 9. MainLayout + Navigation with Auth State

  **What to do**:
  - Create `MainLayout.razor` in `Components/Layout/`:
    - Use `CascadingAuthenticationState`
    - Include `AuthorizeView` to conditionally show Login/Register vs User menu
    - Show user DisplayName and avatar when authenticated
    - Logout button that navigates to `/account/logout`
    - Navigation links: Home, Dashboard (protected), Projects (protected)
  - Create `NavMenu.razor` component:
    - Responsive navigation with mobile hamburger menu
    - Auth-aware links (hide/show based on auth state)
    - Active link styling
  - Create `RedirectToLogin.razor` component for unauthorized access handling.
  - Update `App.razor` to use `CascadingAuthenticationState` and `AuthorizeRouteView`.
  - Style layout and navigation with Tailwind CSS.

  **Must NOT do**:
  - Do NOT create complex dashboard widgets.
  - Do NOT add project-specific navigation yet.
  - Do NOT add search or notifications UI.

  **Recommended Agent Profile**:
  - **Category**: `visual-engineering`
    - Reason: Layout and navigation require good UI/UX implementation with Tailwind.
  - **Skills**: []

  **Parallelization**:
  - **Can Run In Parallel**: NO (depends on Tasks 3, 8)
  - **Parallel Group**: Wave 3
  - **Blocks**: None
  - **Blocked By**: Task 3 (Tailwind CSS), Task 8 (Login/Register pages)

  **References**:
  - PRD Section 4.1: CascadingAuthState, NavigationManager
  - PRD Section 10.1: Global State (CascadingValue, scoped service)
  - Existing file: `Components/Layout/MainLayout.razor` — preserve and enhance

  **Acceptance Criteria**:
  - [ ] MainLayout shows login/register links when anonymous
  - [ ] MainLayout shows user name and logout when authenticated
  - [ ] Navigation menu renders correctly
  - [ ] Tailwind CSS styles applied
  - [ ] Mobile responsive (hamburger menu works)

  **QA Scenarios**:

  ```
  Scenario: Layout shows auth state correctly
    Tool: Bash (curl)
    Preconditions: App running
    Steps:
      1. Run: curl -s http://localhost:5000/ | grep -c "Login"
      2. Assert: Count >= 1 (login link visible when anonymous)
      3. Login: curl -c cookies.txt -d "Input.Email=admin@taskflow.local&Input.Password=Admin123!" http://localhost:5000/account/login
      4. Run: curl -b cookies.txt -s http://localhost:5000/ | grep -c "System Admin"
      5. Assert: Count >= 1 (display name visible when authenticated)
    Expected Result: Layout reflects authentication state
    Evidence: .sisyphus/evidence/task-9-layout-auth.txt
  ```

  **Commit**: YES (groups with Task 9-12)
  - Message: `feat(ui): MainLayout with auth-aware navigation`

- [ ] 10. Landing Page + Protected Pages

  **What to do**:
  - Create `Home.razor` (landing page):
    - Publicly accessible
    - Hero section with app name and description
    - Links to login/register
    - Styled with Tailwind CSS
  - Create `Dashboard.razor`:
    - Protected with `[Authorize]`
    - Simple placeholder content ("Welcome to TaskFlow Dashboard")
    - Show authenticated user info
  - Create `Unauthorized.razor`:
    - Shown when user lacks permissions
    - Friendly message with link to home
  - Update `Routes.razor` to handle authorization and redirects.
  - Ensure routing works: `/` → Home, `/dashboard` → Dashboard (protected), `/account/login` → Login.

  **Must NOT do**:
  - Do NOT add real dashboard widgets or data.
  - Do NOT add project listing or board views.
  - Do NOT add settings or profile pages.

  **Recommended Agent Profile**:
  - **Category**: `visual-engineering`
    - Reason: Landing page requires good visual design with Tailwind.
  - **Skills**: []

  **Parallelization**:
  - **Can Run In Parallel**: NO (depends on Tasks 6, 7)
  - **Parallel Group**: Wave 3
  - **Blocks**: None
  - **Blocked By**: Task 6 (roles seeded), Task 7 (policies configured)

  **References**:
  - PRD Section 7.1: User Stories (US-01 to US-05)
  - Existing file: `Components/Pages/Home.razor` — enhance

  **Acceptance Criteria**:
  - [ ] Home page loads at `/`
  - [ ] Dashboard page requires authentication
  - [ ] Unauthorized page shows for forbidden access
  - [ ] All pages styled with Tailwind CSS

  **QA Scenarios**:

  ```
  Scenario: Protected dashboard requires login
    Tool: Bash (curl)
    Preconditions: App running
    Steps:
      1. Run: curl -i http://localhost:5000/dashboard
      2. Assert: HTTP 302 redirect to login page
      3. Login: curl -c cookies.txt -d "Input.Email=admin@taskflow.local&Input.Password=Admin123!" http://localhost:5000/account/login
      4. Run: curl -b cookies.txt -i http://localhost:5000/dashboard
      5. Assert: HTTP 200
    Expected Result: Dashboard protected, accessible after login
    Evidence: .sisyphus/evidence/task-10-protected-pages.txt
  ```

  **Commit**: YES (groups with Task 9-12)
  - Message: `feat(ui): Landing page, Dashboard, and routing`

- [ ] 11. Health Check Endpoint with DB Connectivity

  **What to do**:
  - Add health checks in `Program.cs`:
    ```csharp
    builder.Services.AddHealthChecks()
        .AddDbContextCheck<TaskFlowDbContext>("database");
    ```
  - Map health check endpoint:
    ```csharp
    app.MapHealthChecks("/health");
    ```
  - Configure health check response to include detailed status (JSON).
  - Ensure health check runs database query to verify connectivity.
  - Test with PostgreSQL running and stopped.

  **Must NOT do**:
  - Do NOT add custom health check UI (like HealthChecksUI package).
  - Do NOT add checks for external services (email, file storage) yet.

  **Recommended Agent Profile**:
  - **Category**: `quick`
    - Reason: Health checks are a standard, well-documented pattern.
  - **Skills**: []

  **Parallelization**:
  - **Can Run In Parallel**: NO (depends on Tasks 2, 4)
  - **Parallel Group**: Wave 3
  - **Blocks**: None
  - **Blocked By**: Task 2 (DbContext), Task 4 (config)

  **References**:
  - PRD Section 10.2: Health check endpoint specified
  - PRD Section 6: Non-functional requirements (availability)
  - .NET docs: `https://learn.microsoft.com/en-us/aspnet/core/host-and-deploy/health-checks`

  **Acceptance Criteria**:
  - [ ] `/health` endpoint returns HTTP 200 when DB is healthy
  - [ ] Response includes "database" check with status "Healthy"
  - [ ] Response time < 2 seconds

  **QA Scenarios**:

  ```
  Scenario: Health check passes with DB up
    Tool: Bash (curl)
    Preconditions: App running, PostgreSQL accessible
    Steps:
      1. Run: curl -s http://localhost:5000/health
      2. Assert: Response contains "Healthy"
      3. Assert: Response contains "database"
    Expected Result: Health check returns healthy status
    Evidence: .sisyphus/evidence/task-11-health-ok.txt

  Scenario: Health check fails with DB down
    Tool: Bash (curl)
    Preconditions: App running, PostgreSQL stopped
    Steps:
      1. Stop PostgreSQL service
      2. Run: curl -s http://localhost:5000/health
      3. Assert: Response contains "Unhealthy" or returns 503
    Expected Result: Health check reflects DB unavailability
    Evidence: .sisyphus/evidence/task-11-health-fail.txt
  ```

  **Commit**: YES (groups with Task 9-12)
  - Message: `feat(ops): Health check endpoint with DB connectivity`

- [ ] 12. Middleware Pipeline + Production Config

  **What to do**:
  - Configure production-ready middleware in `Program.cs`:
    ```csharp
    if (!app.Environment.IsDevelopment())
    {
        app.UseExceptionHandler("/Error", createScopeForErrors: true);
        app.UseHsts();
    }
    app.UseHttpsRedirection();
    app.UseStaticFiles();
    app.UseRouting();
    app.UseAuthentication();
    app.UseAuthorization();
    app.UseAntiforgery();
    ```
  - Ensure middleware order is correct (routing before auth, auth before endpoints).
  - Add `ForwardedHeaders` middleware for reverse proxy support:
    ```csharp
    app.UseForwardedHeaders(new ForwardedHeadersOptions
    {
        ForwardedHeaders = ForwardedHeaders.XForwardedFor | ForwardedHeaders.XForwardedProto
    });
    ```
  - Configure cookie policy for production:
    ```csharp
    builder.Services.ConfigureApplicationCookie(options =>
    {
        options.Cookie.HttpOnly = true;
        options.Cookie.SecurePolicy = CookieSecurePolicy.Always;
        options.Cookie.SameSite = SameSiteMode.Lax;
        options.ExpireTimeSpan = TimeSpan.FromDays(7);
        options.SlidingExpiration = true;
    });
    ```
  - Add DataProtection key persistence (file system for dev, blob for prod later):
    ```csharp
    builder.Services.AddDataProtection()
        .PersistKeysToFileSystem(new DirectoryInfo(@"keys"))
        .SetApplicationName("TaskFlow");
    ```
  - Create `keys/` directory and add to `.gitignore`.
  - Ensure `appsettings.Production.json` has secure defaults.
  - Add environment-specific logging configuration.

  **Must NOT do**:
  - Do NOT add rate limiting yet.
  - Do NOT add CSP headers yet.
  - Do NOT add CORS configuration yet (not needed for Blazor Server).
  - Do NOT configure Azure Key Vault or cloud secrets yet.

  **Recommended Agent Profile**:
  - **Category**: `unspecified-high`
    - Reason: Middleware ordering and security configuration are critical for production readiness.
  - **Skills**: []

  **Parallelization**:
  - **Can Run In Parallel**: NO (depends on Task 4)
  - **Parallel Group**: Wave 3
  - **Blocks**: None
  - **Blocked By**: Task 4 (config setup)

  **References**:
  - PRD Section 6: Non-functional requirements (security, availability)
  - PRD Section 10.3: Environment & Configuration
  - .NET docs: Middleware order, cookie policy, forwarded headers

  **Acceptance Criteria**:
  - [ ] Middleware pipeline configured in correct order
  - [ ] Production config has secure defaults
  - [ ] Cookie policy is secure
  - [ ] DataProtection keys persisted to file system
  - [ ] App starts successfully in both Development and Production modes

  **QA Scenarios**:

  ```
  Scenario: App starts in Production mode
    Tool: Bash
    Preconditions: App built
    Steps:
      1. Run: ASPNETCORE_ENVIRONMENT=Production dotnet run --project TaskFlow.Web &
      2. Wait for startup
      3. Run: curl -i http://localhost:5000/health
      4. Assert: HTTP 200
      5. Run: curl -i http://localhost:5000/account/login
      6. Assert: HTTP 200
    Expected Result: App runs correctly in Production mode
    Evidence: .sisyphus/evidence/task-12-prod-mode.txt
  ```

  **Commit**: YES (groups with Task 9-12)
  - Message: `feat(ops): Production-ready middleware and security config`

---

## Final Verification Wave

> 3 review agents run in PARALLEL. ALL must APPROVE. Present consolidated results to user and get explicit "okay" before completing.

- [ ] F1. **Plan Compliance Audit** — `oracle`
  Read the plan end-to-end. For each "Must Have": verify implementation exists (read file, curl endpoint, run command). For each "Must NOT Have": search codebase for forbidden patterns — reject with file:line if found. Check evidence files exist in `.sisyphus/evidence/`.
  Output: `Must Have [12/12] | Must NOT Have [0/N] | Tasks [12/12] | VERDICT: APPROVE/REJECT`

- [ ] F2. **Real Manual QA** — `unspecified-high`
  Execute EVERY QA scenario from EVERY task — follow exact steps, capture evidence. Test integration: auth → protected pages → role enforcement. Test edge cases: invalid login, duplicate email, DB unreachable, password policy violation.
  Output: `Scenarios [24/24 pass] | Integration [3/3] | Edge Cases [4/4] | VERDICT`

- [ ] F3. **Scope Fidelity Check** — `deep`
  For each task: read "What to do", read actual diff (git log/diff). Verify 1:1 — everything in spec was built, nothing beyond spec was built. Check "Must NOT do" compliance. Detect cross-task contamination.
  Output: `Tasks [12/12 compliant] | Scope Creep [CLEAN] | VERDICT`

---

## Commit Strategy

- **Wave 1**: `feat(arch): Clean Architecture project structure + EF Core + Tailwind`
- **Wave 2**: `feat(auth): ASP.NET Identity with roles + Login/Register pages`
- **Wave 3**: `feat(ui): Layout, navigation, health checks, production config`
- **Wave FINAL**: `chore(review): Week 1 verification and fixes`

---

## Success Criteria

### Verification Commands
```bash
# Build
dotnet build
# Expected: Build succeeded with 0 errors

# Database
dotnet ef migrations list
# Expected: At least one migration listed

dotnet ef database update
# Expected: Done.

# Health check
curl -i http://localhost:5000/health
# Expected: HTTP 200, includes "database" check status

# Auth pages
curl -i http://localhost:5000/account/login
# Expected: HTTP 200

curl -i http://localhost:5000/account/register
# Expected: HTTP 200

# Protected page rejects anonymous
curl -i http://localhost:5000/dashboard
# Expected: HTTP 302 to login page

# Tailwind CSS
curl -I http://localhost:5000/css/app.css
# Expected: HTTP 200, content-length > 10000
```

### Final Checklist
- [ ] All "Must Have" present
- [ ] All "Must NOT Have" absent
- [ ] All QA scenarios pass with evidence captured
- [ ] Solution builds cleanly
- [ ] Database migrations apply successfully
- [ ] Health check returns 200 with DB status
- [ ] Login/Register pages functional
- [ ] Role-based access enforced
- [ ] Tailwind CSS generated and served
