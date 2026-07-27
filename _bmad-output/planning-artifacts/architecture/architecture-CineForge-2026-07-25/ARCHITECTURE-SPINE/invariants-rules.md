# Invariants & Rules

## AD-1 — Single deployment unit, two controller groups

- **Binds:** FR-3, FR-4, FR-5, FR-6, FR-7, FR-12
- **Prevents:** Separate deployable services for admin API vs redirect service in v1
- **Rule:** Admin endpoints live under `/api/` prefix with authentication middleware. Redirect endpoint lives under `/r/{campaignId}` with no authentication middleware. Both in the same .NET web project, deployed as one Render service.

## AD-2 — Interstitial page served by backend, not SPA

- **Binds:** FR-9
- **Prevents:** Loading the full React SPA bundle for the redirect interstitial page
- **Rule:** The `/r/{campaignId}` route returns a lightweight HTML page (Razor view or static HTML with inline JS) that requests geolocation via the Geolocation API, sends coordinates to the backend, then issues a client-side redirect to the campaign URL. No framework, no bundle.

## AD-3 — CurrentRedirectUrl as column on Campaign table

- **Binds:** FR-3, FR-4
- **Prevents:** Separate RedirectConfig table for v1
- **Rule:** `Campaign.CurrentRedirectUrl` (string, required) stores the active destination. URL changes update this column synchronously and append an audit row to `UrlChangeLog` (campaign ID, previous URL, new URL, changed at timestamp).

## AD-4 — RESTful Controllers, not Minimal APIs

- **Binds:** FR-1 through FR-12
- **Prevents:** Mixing controller and minimal API patterns
- **Rule:** All API endpoints use `[ApiController]` classes with explicit route attributes. One controller per resource group: `AuthController`, `CampaignsController`, `AnalyticsController`, `RedirectController`.

## AD-5 — Inline async scan recording (fire-and-forget)

- **Binds:** FR-7, FR-8
- **Prevents:** Background queue, message buffer, or Redis-based scan recording in v1
- **Rule:** The redirect controller writes the scan record asynchronously (`await db.Scans.AddAsync(...)`) and issues the 302 redirect without waiting for database acknowledgement. If the write fails, the scan is lost (acceptable for v1).

## AD-6 — Cookie-based authentication sessions

- **Binds:** FR-1, FR-2
- **Prevents:** JWT-based auth, OAuth, social login, refresh token flows
- **Rule:** Use ASP.NET Core Identity with cookie authentication. Login sets an httpOnly, secure, SameSite=Strict cookie. Session expires after 24 hours of inactivity. 5 failed attempts trigger a 15-minute lockout.

## AD-7 — Inline GeoIP lookup

- **Binds:** FR-8
- **Prevents:** Post-redirect GeoIP enrichment, external GeoIP API calls
- **Rule:** MaxMind GeoLite2 database loaded into memory at application startup via the official .NET client. Lookup runs inline in the redirect request path. Expected overhead <1ms per lookup.

## AD-8 — Entity Framework Core as ORM

- **Binds:** All data-access FRs (FR-1 through FR-12)
- **Prevents:** Dapper, raw ADO.NET, or alternative ORMs
- **Rule:** All database access uses EF Core with code-first migrations. Entities map 1:1 to domain entities in the Domain layer. Repository pattern implemented in Infrastructure layer.
