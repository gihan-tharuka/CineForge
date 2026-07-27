# Requirements Inventory

## Functional Requirements

FR-1: Director login — The director can log in using their email address and password.
FR-2: Session management — The system maintains an authenticated session for the director.
FR-3: Create campaign — The director can create a new campaign with a name and an initial redirect URL.
FR-4: Update redirect URL — The director can change the redirect URL of an active campaign at any time.
FR-5: View campaign list — The director can see all campaigns in a list view.
FR-6: Delete campaign — The director can delete a campaign.
FR-7: Handle scan request — The system accepts GET requests to /r/{campaign-id} and redirects the user to the campaign's current redirect URL.
FR-8: Capture scan metadata — The system captures metadata for every scan request.
FR-9: Request location permission — The redirect page requests the user's geolocation permission before redirecting.
FR-10: View campaign analytics — The director can view analytics for a specific campaign.
FR-11: Export scan data — The director can export raw scan data for a campaign.
FR-12: Generate QR code — The system generates a QR code image for a campaign's redirect URL.

## NonFunctional Requirements

NFR-1: Redirect latency — P95 response time under 200ms for the redirect endpoint.
NFR-2: Scan data completeness — 100% of scans recorded with no data loss.
NFR-3: Director autonomy — 100% of campaign destination changes made through the UI without developer involvement.
NFR-4: Dashboard usability — A first-time director can change the redirect URL and view scan counts within 2 minutes of logging in.
NFR-5: Geolocation opt-in rate — At least 40% of scans include geolocation coordinates.
NFR-6: Session timeout — Session expires after 24 hours of inactivity.
NFR-7: Account lockout — 5 consecutive failed login attempts trigger a 15-minute lockout.
NFR-8: Cookie security — Session token stored as HTTP-only, secure, SameSite=Strict cookie.
NFR-9: Input validation — Campaign name required (1-100 characters); redirect URL must be valid HTTP/HTTPS.
NFR-10: Export limit — Raw data export limited to 100,000 scans per request.
NFR-11: QR code spec — Generated as PNG at 300x300px minimum.
NFR-12: Interstitial performance — Adds no more than 500ms to the user experience.
NFR-13: Rate limiting — 100 requests/second per IP on the redirect endpoint.
NFR-14: Data retention — Scan data retained indefinitely for v1.
NFR-15: Interstitial page weight — Total page weight under 5KB, first paint under 100ms on 3G.

## Additional Requirements

- Starter template: Clean/Hexagonal Architecture with 4 layers (Presentation, Application, Domain, Infrastructure) — zero dependencies from Domain outward.
- Backend: .NET 10 with ASP.NET Core, EF Core (code-first migrations), ASP.NET Core Identity (cookie auth), MaxMind GeoLite2 (in-memory GeoIP), QRCoder (QR generation).
- Frontend: React 19 SPA with Vite 6, Tailwind CSS 4, TanStack Router/Table/Query, shadcn/ui components.
- Database: PostgreSQL 16 (Neon) with entities: User, Campaign, Scan, UrlChangeLog.
- Deployment: Single .NET web project on Render (admin + redirect controllers), React SPA on Vercel (frontend), shared PostgreSQL on Neon.
- API conventions: RESTful [ApiController] classes (not Minimal APIs), kebab-case plural routes (/api/campaigns), PascalCase entities, UTC ISO 8601 dates, RFC 7807 Problem Details for errors.
- Interstitial: Served by .NET backend as Razor view (not SPA), inline CSS/JS, no framework bundle.
- Source tree: src/CineForge.Web/, src/CineForge.Application/, src/CineForge.Domain/, src/CineForge.Infrastructure/, frontend/.
- Deferred post-v1: background queue for scan writes, separate redirect service deployment, scheduled URL changes, email digest, multi-user access, Redis session store, Serilog sinks, CI/CD pipeline.

## UX Design Requirements

UX-DR-1: Implement design token system — deep slate (#0F172A) primary, warm amber (#F59E0B) accent, surface/ink color palette, 4-based spacing scale (4,8,12,16,24,32,48,64), Inter font system with mono fallback, rounded scale (sm:4px, md:6px, lg:8px, full:9999px).
UX-DR-2: Implement brand-layer-overridden shadcn/ui components — Button (primary/secondary/ghost/destructive variants), Input (URL field with validation), Campaign card, Interstitial, Status pill.
UX-DR-3: Implement shadcn/ui base components — Card, Dialog, Sheet, DropdownMenu, Toast, Tabs, Avatar, Separator, Table.
UX-DR-4: Implement responsive layout — max-w-3xl dashboard, single-column card list, 2-column grid on md+ for analytics, sidebar collapses to icons on md, Sheet on sm.
UX-DR-5: Implement accessibility floor — WCAG 2.1 AA compliance, screen reader page announcements, keyboard navigation with Tab order, focus rings using accent color, aria-live for errors, tap targets ≥ 44px on mobile.
UX-DR-6: Implement state patterns — loading skeletons (4-6 cards), empty states ("No campaigns yet"), error states (inline URL validation), offline handling (toast + disabled inputs), session expired redirect with ?expired=true.
UX-DR-7: Implement interaction primitives — click-to-edit redirect URL, click-to-navigate campaign card, copy-to-clipboard for URL/ID, download QR code PNG, export with date range and format selector, theme toggle (light/dark), logout via session menu.
UX-DR-8: Implement interstitial page — flat HTML, inline CSS/JS, < 5KB total weight, < 100ms first paint on 3G, geolocation permission requested on load, auto-redirect after permission flow.
UX-DR-9: Implement voice and tone — professional, direct, no exclamation marks, no emojis, specific microcopy for all states (success toasts, error messages, empty states, lockout countdown).
UX-DR-10: Implement mockup pages — login.html, campaign-list.html, campaign-detail.html, analytics.html, interstitial.html.

## FR Coverage Map

| FR | Epic | Brief Description |
|---|---|---|
| FR-1 | Epic 1 | Director login with email/password |
| FR-2 | Epic 1 | Session management with cookie auth |
| FR-3 | Epic 2 | Create campaign with name and initial redirect URL |
| FR-4 | Epic 2 | Update redirect URL of active campaign |
| FR-5 | Epic 2 | View all campaigns in list view |
| FR-6 | Epic 2 | Delete campaign with confirmation |
| FR-7 | Epic 3 | Handle GET /r/{campaign-id} and issue 302 redirect |
| FR-8 | Epic 3 | Capture scan metadata (IP, GeoIP, user-agent, device) |
| FR-9 | Epic 3 | Request geolocation permission on interstitial page |
| FR-10 | Epic 4 | View campaign analytics (time-series, geography, device) |
| FR-11 | Epic 4 | Export raw scan data as CSV/JSON |
| FR-12 | Epic 2 | Generate QR code PNG for campaign redirect URL |

## NFR Coverage Map

| NFR | Epic | Addressed By |
|---|---|---|
| NFR-1 | Epic 3 | Redirect latency (200ms P95) — rate limiting, inline GeoIP |
| NFR-2 | Epic 3 | Scan data completeness — async write, durable to Neon |
| NFR-3 | Epic 2 | Director autonomy — UI-based redirect URL changes |
| NFR-4 | Epic 4 | Dashboard usability — TanStack Query polling, loading states |
| NFR-5 | Epic 3 | Geolocation opt-in — interstitial permission prompt |
| NFR-6 | Epic 1 | Session timeout (24h inactivity) — cookie sliding expiration |
| NFR-7 | Epic 1 | Account lockout (5 attempts / 15min) — ASP.NET Core Identity |
| NFR-8 | Epic 1 | Cookie security — httpOnly, secure, SameSite=Strict |
| NFR-9 | Epic 2 | Input validation — campaign name (1-100 chars), URL validation |
| NFR-10 | Epic 4 | Export limit (100k scans) — server-side pagination |
| NFR-11 | Epic 2 | QR code spec (300x300px PNG) — QRCoder |
| NFR-12 | Epic 3 | Interstitial performance (<500ms) — lightweight Razor view |
| NFR-13 | Epic 3 | Rate limiting (100 req/s per IP) — middleware |
| NFR-14 | Epic 3 | Data retention (indefinite) — PostgreSQL storage |
| NFR-15 | Epic 3 | Interstitial page weight (<5KB, <100ms paint) — inline CSS/JS |

## UX-DR Coverage Map

| UX-DR | Epic | Addressed By |
|---|---|---|
| UX-DR-1 | Epic 1 | Design token system (colors, spacing, typography, rounded) |
| UX-DR-2 | Epic 1 | Login form component (brand-layer override) |
| UX-DR-2 | Epic 2 | Campaign card, Input (URL field), Status pill |
| UX-DR-3 | Epic 1 | Base shadcn/ui components (Card, Dialog, Sheet, etc.) |
| UX-DR-3 | Epic 2 | Base components for campaign list/detail |
| UX-DR-3 | Epic 4 | Base components for analytics page |
| UX-DR-4 | Epic 1 | Responsive layout foundation (max-w-3xl, breakpoints) |
| UX-DR-4 | Epic 4 | Analytics 2-column grid on md+ |
| UX-DR-5 | All | Accessibility floor (WCAG 2.1 AA, keyboard nav, aria-live) |
| UX-DR-6 | Epic 2 | Loading skeletons, empty states, error states, offline |
| UX-DR-6 | Epic 4 | Analytics loading states |
| UX-DR-7 | Epic 2 | Click-to-edit, click-to-navigate, copy-to-clipboard, download QR |
| UX-DR-7 | Epic 4 | Export with date range and format selector |
| UX-DR-8 | Epic 3 | Interstitial page (flat HTML, <5KB, geolocation on load) |
| UX-DR-9 | All | Voice and tone (microcopy, no exclamation marks) |
| UX-DR-10 | Epic 1 | login.html mockup |
| UX-DR-10 | Epic 2 | campaign-list.html, campaign-detail.html mockups |
| UX-DR-10 | Epic 3 | interstitial.html mockup |
| UX-DR-10 | Epic 4 | analytics.html mockup |
