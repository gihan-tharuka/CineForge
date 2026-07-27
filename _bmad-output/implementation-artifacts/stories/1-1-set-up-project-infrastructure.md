# Story 1.1: Set up project infrastructure

Status: ready-for-dev

## Story

As a developer,
I want the .NET backend solution and React frontend project initialized with all required dependencies,
so that subsequent stories can build features on a stable foundation.

## Acceptance Criteria

1. **Given** no project exists
   **When** the infrastructure story is complete
   **Then** a .NET 10 solution exists with CineForge.Web, CineForge.Application, CineForge.Domain, and CineForge.Infrastructure projects

2. **And** a React 19 + Vite 6 project exists in `frontend/` with Tailwind CSS 4, TanStack Router/Table/Query, and shadcn/ui installed

3. **And** EF Core is configured with a connection to PostgreSQL (Neon)

4. **And** ASP.NET Core Identity is configured with cookie authentication

5. **And** the source tree matches the architecture spine specification

## Tasks / Subtasks

- [ ] Initialize .NET 10 solution with Clean Architecture (AC: #1, #5)
  - [ ] Create solution `CineForge.sln` with `dotnet new sln`
  - [ ] Create `CineForge.Domain` class library project
  - [ ] Create `CineForge.Application` class library project
  - [ ] Create `CineForge.Infrastructure` class library project
  - [ ] Create `CineForge.Web` ASP.NET Core web project
  - [ ] Add project references: Web → Application, Application → Domain, Infrastructure → Application, Infrastructure → Domain
  - [ ] Add solution references for all projects

- [ ] Configure EF Core + PostgreSQL (AC: #3)
  - [ ] Add `Npgsql.EntityFrameworkCore.PostgreSQL` NuGet package to Infrastructure
  - [ ] Create `AppDbContext` in Infrastructure/Data
  - [ ] Register DbContext in Infrastructure with connection string from configuration
  - [ ] Add connection string to `appsettings.json` (placeholder for Neon)

- [ ] Configure ASP.NET Core Identity with cookie auth (AC: #4)
  - [ ] Add `Microsoft.AspNetCore.Identity.EntityFrameworkCore` NuGet package
  - [ ] Create `CineForgeUser` identity entity in Infrastructure/Identity
  - [ ] Configure Identity services in Infrastructure with cookie authentication
  - [ ] Set cookie options: httpOnly, secure, SameSite=Strict, sliding expiration 24h

- [ ] Initialize React 19 + Vite 6 frontend (AC: #2)
  - [ ] Scaffold Vite 6 React project in `frontend/`
  - [ ] Install and configure Tailwind CSS 4
  - [ ] Install TanStack Router v1, TanStack Table v8, TanStack Query v5
  - [ ] Install and configure shadcn/ui components
  - [ ] Set up project structure: `src/routes/`, `src/components/`, `src/lib/`, `src/hooks/`

- [ ] Verify source tree matches architecture spine (AC: #5)
  - [ ] Validate folder structure against architecture specification
  - [ ] Ensure dependency direction is correct (no circular references)

## Dev Notes

### Architecture Compliance

**Stack versions (from architecture spine):**
- .NET 10 (LTS channel) — backend runtime
- ASP.NET Core 10 — web framework
- Entity Framework Core 10 — ORM
- PostgreSQL 16 (Neon) — database
- React 19 — frontend SPA
- Vite 6 — frontend build tool
- Tailwind CSS 4 — utility CSS
- TanStack Query v5 — server state / API client
- TanStack Router v1 — type-safe routing
- TanStack Table v8 — data tables
- shadcn/ui — component library (latest)

**Source tree structure (must match exactly):**
```
CineForge/
├── src/
│   ├── CineForge.Web/            # Presentation — ASP.NET project
│   │   ├── Controllers/          # AuthController, CampaignsController,
│   │   │                         # AnalyticsController, RedirectController
│   │   ├── Views/Redirect/       # Interstitial.cshtml (Razor view)
│   │   ├── Middleware/           # RateLimiting.cs
│   │   ├── Program.cs
│   │   └── appsettings.json
│   ├── CineForge.Application/    # Use cases / business logic
│   │   ├── Common/               # Interfaces, DTOs, Behaviors
│   │   ├── Auth/                 # LoginCommand, LogoutCommand
│   │   ├── Campaigns/            # CreateCampaignCommand, etc.
│   │   ├── Analytics/            # GetAnalyticsQuery, etc.
│   │   └── DependencyInjection.cs
│   ├── CineForge.Domain/         # Enterprise business rules
│   │   ├── Entities/             # Campaign, Scan, UrlChangeLog
│   │   ├── ValueObjects/         # Email, RedirectUrl, DeviceType
│   │   └── Interfaces/           # ICampaignRepository, IScanRepository
│   └── CineForge.Infrastructure/ # External concerns
│       ├── Data/                 # AppDbContext, Migrations
│       ├── Identity/             # CineForgeUser, IdentityConfig
│       ├── Services/             # GeoIpService, QrCodeService
│       └── DependencyInjection.cs
├── frontend/                     # React SPA
│   ├── src/
│   │   ├── routes/               # TanStack Router route tree
│   │   ├── components/           # shadcn/ui + custom components
│   │   ├── lib/                  # API client, utils
│   │   └── hooks/                # Custom hooks
│   ├── index.html
│   ├── vite.config.ts
│   ├── tailwind.config.ts
│   └── package.json
└── CineForge.sln
```

**Dependency direction (strict — no violations allowed):**
- Web → Application → Domain
- Infrastructure → Application
- Infrastructure → Domain
- Domain: zero dependencies on any other project
- Application: depends only on Domain

**Design paradigm:** Clean/Hexagonal Architecture. Domain layer has zero external dependencies. Application layer orchestrates use cases. Infrastructure implements interfaces defined in Domain/Application.

### Security Requirements

- ASP.NET Core Identity with cookie authentication
- Cookie options: httpOnly=true, secure=true (in production), SameSite=Strict
- Sliding expiration: 24 hours of inactivity
- Identity configured with lockout: 5 attempts, 15-minute lockout duration

### Testing Standards

- Backend: xUnit with FluentAssertions (preferred)
- Test projects mirror source structure: `tests/CineForge.Web.Tests/`, `tests/CineForge.Application.Tests/`, etc.
- Frontend: Vitest for unit tests, Testing Library for component tests
- This story is infrastructure setup — no functional tests required, but verify solution builds cleanly

### Project Structure Notes

- All source code under `src/` directory
- Frontend SPA in `frontend/` directory (separate from backend)
- Test projects in `tests/` directory
- No circular dependencies between projects
- Domain project must not reference any external NuGet packages (pure C#)

### References

- [Source: epics/epic-1-authentication-session-management.md#story-11-set-up-project-infrastructure]
- [Source: architecture/ARCHITECTURE-SPINE/structural-seed.md]
- [Source: architecture/ARCHITECTURE-SPINE/stack.md]
- [Source: architecture/ARCHITECTURE-SPINE/dependency-direction.md]
- [Source: architecture/ARCHITECTURE-SPINE/design-paradigm.md]
- [Source: architecture/ARCHITECTURE-SPINE/invariants-rules.md]
- [Source: architecture/ARCHITECTURE-SPINE/consistency-conventions.md]
- [Source: prd/prd-CineForge-2026-07-25/prd.md]
- [Source: ux-designs/ux-CineForge-2026-07-25/DESIGN.md]

## Dev Agent Record

### Agent Model Used

Claude (BMAD create-story workflow)

### Debug Log References

### Completion Notes List

- Story created from sprint status auto-discovery: first backlog story (1-1-set-up-project-infrastructure)
- No previous story intelligence available (first story in epic)
- No project-context.md found — using architecture spine and epic documents as primary sources
- Git history shows only initial commit and BMAD install — no prior implementation patterns

### File List

- `_bmad-output/implementation-artifacts/stories/1-1-set-up-project-infrastructure.md` (this file)