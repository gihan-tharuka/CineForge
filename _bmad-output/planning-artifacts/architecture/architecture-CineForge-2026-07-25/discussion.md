---
title: "CineForge Architecture — Discussion & Rationale"
status: final
created: 2026-07-25T14:23:00+05:30
updated: 2026-07-25T14:23:00+05:30
audience: "Technical team, stakeholders, future contributors"
---

# CineForge Architecture — Discussion & Rationale

This document explains *why* the architecture decisions were made. It's meant for discussion, review, and onboarding — not as a binding contract (that's the [Architecture Spine](./ARCHITECTURE-SPINE.md)).

## What We're Building

CineForge is a dynamic QR redirect + analytics platform. A film director prints a static QR code on popcorn buckets. That QR code never changes, but what it redirects to changes with a click in the dashboard. Behind the redirect, every scan is captured with geolocation for hall-level attribution.

**Two user-facing surfaces:**
1. **The Director Dashboard** — a React SPA where the director manages campaigns and views analytics
2. **The Redirect Service** — a lightweight endpoint that scans hit, captures data, and redirects

**Hosting:**
- Frontend → **Vercel**
- Backend API → **Render**
- Database → **Neon (PostgreSQL)**

---

## Key Architectural Decisions (and Why)

### 1. Clean Architecture

**Decision:** Organise the .NET backend into four layers: Domain, Application, Infrastructure, Presentation.

**Why not Vertical Slices?** CineForge has two very different entry points (authenticated admin API and public redirect endpoint) that share the same domain logic — campaigns, scans, users. Clean Architecture lets them share that core cleanly without coupling their delivery mechanisms. Vertical Slices would duplicate entity logic across slices.

**Why not a simple flat structure?** As the system grows (more campaign types, richer analytics, scheduled redirects), the layer boundaries prevent the kind of tangled dependencies that make small changes risky. The cost of the extra project files is negligible at setup time.

### 2. Single Deployment, Two Controller Groups

**Decision:** One .NET project deployed to Render, with admin controllers under `/api/` and the redirect controller under `/r/`.

**Why not split them?** In v1, the admin API and redirect service have similar infrastructure requirements (same database, same GeoIP database, same host). Splitting them would mean two Render services, two deploys, two domains to manage — complexity without benefit at this scale. If the redirect service becomes a bottleneck (e.g., thousands of scans per minute), it's trivial to extract into a separate service with a shared database.

### 3. Interstitial Served by Backend, Not SPA

**Decision:** The redirect interstitial page (the page scanners see before being redirected) is served by the .NET backend as a lightweight Razor view, not by the React SPA.

**Why?** The interstitial needs to load as fast as possible — FR-9 says <500ms added latency. A React SPA loads the entire framework bundle before rendering anything. A plain HTML page with inline JavaScript is instant. Since the .NET backend already handles the `/r/{campaign-id}` route, serving the interstitial from there costs nothing extra.

**The flow:**
1. Moviegoer scans QR code → phone opens `api.cineforge.io/r/campaign-abc123`
2. .NET backend returns a tiny HTML page (Razor view)
3. Page runs inline JS: requests geolocation permission → sends coordinates to backend API → redirects to campaign URL
4. Backend records the scan (IP, GeoIP, user-agent, coordinates if granted) asynchronously

### 4. Data Model: One Table, One Column

**Decision:** The current redirect URL is stored directly on the Campaign table (`Campaign.CurrentRedirectUrl`). URL changes update this column and log to a separate `UrlChangeLog` table.

**Why not a separate RedirectConfig table?** A separate table with active/inactive configs would support scheduled URL changes. That's not in v1 scope. The simpler column approach means the redirect path is a single database query: `SELECT CurrentRedirectUrl FROM Campaigns WHERE Id = @id`. A join or subquery for the "active" config would add unnecessary complexity for v1.

### 5. Cookie-Based Auth, Not JWT

**Decision:** ASP.NET Core Identity with cookie authentication. No JWTs, no refresh tokens.

**Why?** CineForge has a single user (the director) logging in from a browser. Cookies are:
- **Simpler** — no token storage in JavaScript, no refresh token rotation
- **More secure** — httpOnly cookies can't be read by XSS, same-origin policy applies automatically
- **Revocable** — server can invalidate sessions immediately (not possible with JWTs without a blocklist)

JWTs shine for mobile apps, third-party API access, and stateless authentication at scale. CineForge has none of these requirements.

### 6. Scan Recording: Fire-and-Forget

**Decision:** The redirect controller writes the scan to the database asynchronously and issues the 302 redirect without waiting for the database write to complete.

**Why not a background queue?** A background queue (Hangfire, Channel<T>, RabbitMQ) would decouple scan recording from the redirect, but adds infrastructure complexity. For v1, the expected scan volume is low (hundreds to low thousands per screening). The async write is "best effort" — if the write fails, the scan is lost. This is acceptable for v1. If scan volume becomes a concern, adding a background worker is straightforward.

### 7. GeoIP Lookup: Inline

**Decision:** GeoIP lookup (MaxMind GeoLite2) runs inline in the redirect request path, not post-redirect.

**Why?** MaxMind GeoLite2 with the .NET client loads the entire database into memory. Lookups are consistently under 1ms — far below noise level compared to the 200ms P95 response time target. Doing it inline means the scan record is complete on first write, which simplifies the analytics dashboard and export queries.

### 8. EF Core as ORM

**Decision:** Entity Framework Core with code-first migrations. No Dapper.

**Why?** The data model has clear relationships (Campaign → Scans, Campaign → UrlChangeLogs, User → Sessions). EF Core handles migrations, relationship navigation, and LINQ queries naturally. Dapper would be faster for individual queries but adds boilerplate for CRUD operations and migrations.

---

## How Data Flows

### Campaign Update Flow

```
Director logs in (cookie auth)
  → Dashboard SPA (Vercel)
    → PUT /api/campaigns/{id}/redirect-url (Render)
      → AuthMiddleware validates session cookie
      → CampaignsController validates URL
      → CampaignService updates CurrentRedirectUrl
      → EF Core writes: UPDATE Campaigns + INSERT UrlChangeLog
    → 200 OK
  → TanStack Query invalidates cache
  → Dashboard shows updated URL
```

### Scan Flow

```
Moviegoer scans QR code
  → GET /r/campaign-abc123 (Render, no auth)
    → RedirectController:
        1. Lookup campaign → get CurrentRedirectUrl
        2. Initiate async write: INSERT Scan (IP, GeoIP, UA, timestamp)
        3. Return HTML interstitial page (immediately)
    → Browser renders interstitial (5ms)
    → Browser JS: request geolocation → POST /api/scans/{id}/location
    → Browser JS: window.location = redirect URL
  → Moviegoer lands on campaign destination
```

---

## Risks and Mitigations

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| Geolocation API permission denied (most mobile users) | High | Reduced hall-level attribution | Graceful fallback — redirect works without location. Only country/city from GeoIP. |
| Single-user auth is too restrictive | Low (per PRD) | Director needs to delegate | Add user table + roles post-v1. Architecture supports it. |
| Scan volume spikes on opening weekend | Medium | Redirect latency increases | Fire-and-forget write prevents DB backpressure. Monitor and add queue if needed. |
| QR code printed with wrong URL | Low | Campaign broken | Director can test the QR before printing. URL can be changed after printing (the whole point). |
| MaxMind GeoLite2 data is imprecise | Low | Wrong city/country attribution | Free tier is accurate to city level. Upgrade to paid MaxMind if precision matters. |

---

## What's Deferred

These decisions are consciously pushed to post-v1:

- **Email digests** — deferred from PRD scope
- **Scheduled URL changes** — would require data model migration (Option B RedirectConfig)
- **Separate redirect service deployment** — easy to extract when needed
- **Background queue for scans** — add when scan volume demands it
- **Redis for session storage** — add when scaling to multiple instances
- **CI/CD pipeline** — manual deploy via Render/Vercel dashboards is sufficient for v1