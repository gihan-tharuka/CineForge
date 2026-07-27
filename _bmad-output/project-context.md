---
project_name: 'CineForge'
user_name: 'Gihan'
date: '2026-07-26'
sections_completed: ['technology_stack']
existing_patterns_found: 12
---

# Project Context for AI Agents

_This file contains critical rules and patterns that AI agents must follow when implementing code in this project. Focus on unobvious details that agents might otherwise miss._

---

## Technology Stack & Versions

| Category | Technology | Version | Notes |
|----------|-----------|---------|-------|
| Backend Runtime | .NET (ASP.NET Core) | 10 LTS | LTS channel |
| ORM | Entity Framework Core | 10 | Code-first migrations |
| Database | PostgreSQL (Neon) | 16 | Shared across backend & admin |
| Frontend SPA | React | 19 | Vite 6 build tool |
| Utility CSS | Tailwind CSS | 4 | |
| Server State | TanStack Query | 5 | API client abstraction |
| Type-safe Routing | TanStack Router | 1 | File-based route tree |
| Data Tables | TanStack Table | 8 | |
| UI Components | shadcn/ui | latest | Installed via CLI |
| GeoIP | MaxMind GeoLite2 | current (free) | In-memory lookup |
| QR Generation | QRCoder | latest stable | |
| Backend Testing | xUnit + FluentAssertions | latest | |
| Frontend Testing | Vitest + Testing Library | latest | |
| Authentication | ASP.NET Core Identity | 10 | Cookie-based, not JWT |
| Logging | ILogger<T> | console | Serilog deferred to v2 |

## Critical Implementation Rules

### Architecture & Dependency Rules

- **Strict Clean/Hexagonal Architecture:** 4 layers — Web, Application, Domain, Infrastructure
- **Dependency direction (no violations allowed):**
  - Web → Application → Domain
  - Infrastructure → Application, Infrastructure → Domain
  - Domain: **zero external dependencies** (pure C# only, no NuGet packages)
  - Application: depends only on Domain
  - Infrastructure: implements interfaces from Domain and Application
- **All source code** under `src/` directory
- **Frontend SPA** in `frontend/` directory (separate from backend — different deployment)
- **Test projects** in `tests/` directory, mirroring source structure
- No circular dependencies between projects

### Security Rules

- **Authentication:** ASP.NET Core Identity with cookie authentication (NOT JWT)
- **Cookie options:** httpOnly=true, secure=true (production), SameSite=Strict
- **Sliding expiration:** 24 hours of inactivity
- **Account lockout:** 5 failed attempts, 15-minute lockout duration
- **Login errors:** Generic "Invalid email or password" — no user enumeration
- **Session expiry redirect:** `/login?expired=true` with flash message "Your session has expired. Please log in again."

### Frontend Rules

- **Routing:** TanStack Router v1 with file-based route tree under `src/routes/`
- **Server state:** TanStack Query v5 for all API calls
- **Data display:** TanStack Table v8 for tabular data
- **UI components:** shadcn/ui installed via CLI — use existing components, create custom only when needed
- **Styling:** Tailwind CSS 4 utility classes — no raw CSS files
- **Build tool:** Vite 6

### Design Tokens (must follow exactly)

| Token | Light | Dark | Usage |
|-------|-------|------|-------|
| Primary (Deep Slate) | #0F172A | #0F172A | Primary button, active nav, dark-mode canvas |
| Accent (Warm Amber) | #F59E0B | #D4A574 | Primary actions only — Save, Create, Export. Never decorative. |
| Surface Base | #FFFFFF | #0F172A | Canvas |
| Surface Raised | #F8FAFC | #1E293B | Cards, inputs, elevated surfaces |
| Ink Primary | #0F172A | #F1F5F9 | Body text, headings |
| Ink Secondary | #64748B | #94A3B8 | Secondary text |
| Destructive | #EF4444 | #EF4444 | Delete confirmation, error states only |
| Primary Foreground | #FFFFFF | — | Text on primary |
| Accent Foreground | #1A1A2E | — | Text on accent |
| Font | Inter | Inter | Body and headings |
| Border radius | 6px (default), 8px (cards), 4px (inputs) | | |

### Testing Rules

- **Backend:** xUnit with FluentAssertions
- **Frontend:** Vitest (unit), Testing Library (component tests)
- Test files mirror source structure in `tests/` directory
- No functional tests required for infrastructure setup — verify builds cleanly
- Domain layer requires pure unit tests (no infrastructure)

### Backend Project Structure

```
src/CineForge.Web/          # Controllers/, Views/Redirect/, Middleware/, Program.cs
src/CineForge.Application/  # Common/, Auth/, Campaigns/, Analytics/, DependencyInjection.cs
src/CineForge.Domain/        # Entities/, ValueObjects/, Interfaces/ (pure C#)
src/CineForge.Infrastructure/ # Data/, Identity/, Services/, DependencyInjection.cs
```

### Frontend Project Structure

```
frontend/src/
  routes/     # TanStack Router route tree
  components/ # shadcn/ui + custom components
  lib/        # API client, utils
  hooks/      # Custom hooks