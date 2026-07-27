---
title: "PRD: CineForge"
status: final
created: 2026-07-25T09:14:00+05:30
updated: 2026-07-25T13:55:00+05:30
---

# PRD: CineForge

## 0. Document Purpose

This PRD defines the requirements for CineForge v1 — a dynamic QR redirect and analytics platform for film directors. It is the authoritative reference for downstream workflows: UX design, system architecture, and story/epic creation. It builds on the CineForge Product Brief (see `_bmad-output/planning-artifacts/briefs/brief-CineForge-2026-07-25/brief.md`) and does not duplicate it.

The document is structured as: Vision → Target User → Glossary → Features (grouped with globally numbered FRs) → Non-Goals → MVP Scope → Success Metrics → Open Questions → Assumptions Index. Inline `[ASSUMPTION]` tags mark inferred decisions awaiting confirmation.

## 1. Vision

CineForge turns a static printed QR code into a dynamic, measurable marketing channel for film directors. A director prints a single QR code on every popcorn bucket in a production run, or displays it on-screen during a show. That QR code never changes. But what it points to — a Facebook page, a poll, a giveaway landing page, a live interview — changes whenever the director wants. One click in the dashboard, and the entire audience is redirected to a new destination. No reprinting. No new artwork. No lead time.

Behind the redirect, CineForge captures every scan: IP address, country, timestamp, and — using the phone's location permission — which cinema hall the scan came from. The director sees real-time analytics and can export raw data for deeper analysis.

In 2-3 years, CineForge becomes the standard engagement layer for physical cinema marketing — and beyond into live events, concerts, and exhibitions. But v1 stays focused: one director, one dashboard, one QR code that never needs reprinting.

## 2. Target User

### 2.1 Jobs To Be Done

- **Change a campaign destination without reprinting materials.** The director needs to swap what a QR code points to (e.g., from a Facebook page to a giveaway form) in seconds, not days.
- **Know how many people scanned and from where.** The director needs scan counts, geographic distribution, and hall-level attribution to measure campaign reach.
- **Export data for reporting.** The director needs raw scan data to share with distributors, cinema partners, or for their own analysis.
- **Manage everything alone.** The director is a single operator — no team, no permissions, no approvals.

### 2.2 Non-Users (v1)

- Cinema staff or marketing partners (read-only report sharing is possible, but they do not manage the system).
- Multiple directors or multi-tenant usage.
- End consumers scanning the QR (they are the *subject* of the analytics, not users of the system).

### 2.3 Key User Journeys

**UJ-1. The director launches a new campaign.**

> David, a film director in the final week before release, printed 10,000 popcorn buckets with a QR code three weeks ago. The QR points to the film's Facebook page. Today, a surprise interview with the lead actor was published on YouTube. David logs into CineForge (email + password), navigates to the dashboard, pastes the YouTube link into the redirect URL field, and clicks Save. He sees a confirmation: "Redirect updated. Next scan goes to your new destination." He closes the browser. **Edge case:** if David pastes an invalid URL, the system shows an error before saving.

**UJ-2. The director checks campaign performance mid-week.**

> David opens CineForge on his phone during a break. The dashboard shows: 1,247 total scans, 843 from the opening weekend, 312 from the past 24 hours. A map shows clusters in Colombo, Kandy, and Galle. He taps a cluster and sees hall-level breakdowns: Scope Cinema (Colombo) — 412 scans, Liberty Cinema (Colombo) — 198 scans. He exports the week's data as CSV to share with his distributor.

**UJ-3. A moviegoer scans the QR code.**

> Amaya buys popcorn at Scope Cinema, notices the QR code on the bucket, and scans it with her phone camera. The browser opens `cineforge.io/r/campaign-abc123`. A lightweight page loads, requests location permission ("Allow to see nearby offers"), and immediately redirects to the current campaign URL (the YouTube interview). The entire flow takes under 2 seconds. Her scan is recorded: IP, country (Sri Lanka), city (Colombo), timestamp, device type, and location coordinates (if permitted).

## 3. Glossary

- **Campaign** — A named redirect configuration. A campaign has one active redirect URL, a creation date, and a scan log. A director can have multiple campaigns, each with its own static QR code.
- **Redirect URL** — The destination URL that scanners are sent to. The director can change this at any time. Changes take effect immediately.
- **Scan** — A single visit to the CineForge redirect endpoint. Each scan captures: IP address, country, city, timestamp, user-agent, device type, and optionally geolocation coordinates.
- **Hall-level attribution** — The process of mapping a scan to a specific cinema hall using the phone's geolocation coordinates combined with the scan timestamp.
- **Director Dashboard** — The password-protected admin web UI where the director manages campaigns and views analytics.
- **Redirect Service** — The public-facing HTTP endpoint (`cineforge.io/r/{campaign-id}`) that captures scan data and issues a 302 redirect to the current campaign URL.
- **Static QR Code** — The printed QR code that never changes. It always encodes the CineForge redirect URL for a specific campaign.

## 4. Features

### 4.1 Authentication

**Description:** The director logs in to the dashboard using email and password. Single-user system — no registration flow, no multi-user, no roles. The account is provisioned during deployment/setup. Realizes UJ-1.

**Functional Requirements:**

#### FR-1: Director login

The director can log in using their email address and password.

**Consequences (testable):**
- System accepts valid email + password combination and returns a session token (JWT or cookie).
- System rejects invalid credentials with a generic "Invalid email or password" error (no user enumeration).
- System locks the account after 5 consecutive failed attempts for 15 minutes.
- Session expires after 24 hours of inactivity.

**Out of Scope:**
- Registration, password reset, OAuth, MFA, social login.

#### FR-2: Session management

The system maintains an authenticated session for the director.

**Consequences (testable):**
- Authenticated requests to the API return 200; unauthenticated requests return 401.
- Session token is stored as an HTTP-only secure cookie.
- Director can log out, which invalidates the session.

### 4.2 Campaign Management

**Description:** The director creates and manages campaigns. Each campaign has a static QR code and a single active redirect URL that the director can change at any time. Realizes UJ-1.

**Functional Requirements:**

#### FR-3: Create campaign

The director can create a new campaign with a name and an initial redirect URL.

**Consequences (testable):**
- System generates a unique campaign ID and a static redirect URL (`/r/{campaign-id}`).
- System generates a QR code image encoding the full redirect URL.
- System stores the campaign with status "active".
- Campaign name is required (1-100 characters).
- Initial redirect URL is required and must be a valid HTTP/HTTPS URL.

#### FR-4: Update redirect URL

The director can change the redirect URL of an active campaign at any time.

**Consequences (testable):**
- New URL takes effect immediately — the next scan is redirected to the new destination.
- System logs the URL change with a timestamp (for audit).
- URL must be a valid HTTP/HTTPS URL.
- Invalid URLs are rejected with a descriptive error message.

#### FR-5: View campaign list

The director can see all campaigns in a list view.

**Consequences (testable):**
- List shows: campaign name, current redirect URL, total scan count, created date, last scan date.
- Campaigns are sorted by most recently updated first.
- Director can click into any campaign to see its detail view.

#### FR-6: Delete campaign

The director can delete a campaign.

**Consequences (testable):**
- Deleting a campaign deactivates its redirect URL (returns HTTP 410 Gone).
- Scan data is retained for historical analytics.
- Director confirms deletion before it executes.

### 4.3 Redirect Service

**Description:** The public-facing endpoint that scanners hit. It captures scan metadata and issues a 302 redirect to the current campaign URL. Realizes UJ-3.

**Functional Requirements:**

#### FR-7: Handle scan request

The system accepts GET requests to `/r/{campaign-id}` and redirects the user to the campaign's current redirect URL.

**Consequences (testable):**
- System returns HTTP 302 with the `Location` header set to the campaign's current redirect URL.
- System records the scan before issuing the redirect (fire-and-forget or async to avoid adding latency).
- Total response time (request received → redirect issued) is under 200ms at P95.
- If the campaign does not exist or was deleted, system returns HTTP 404 or 410 respectively.

#### FR-8: Capture scan metadata

The system captures metadata for every scan request.

**Consequences (testable):**
- Captured fields: IP address, timestamp (UTC), user-agent, referer header (if present), device type (parsed from user-agent), country and city (via GeoIP lookup on IP).
- If the browser sends geolocation coordinates (via a client-side script on the redirect page), those are captured as well.
- All fields are stored in the database associated with the campaign.
- No scan data is lost — writes are durable (acknowledged write to Neon).

#### FR-9: Request location permission

The redirect page requests the user's geolocation permission before redirecting.

**Consequences (testable):**
- A lightweight interstitial page loads before the redirect.
- The page requests geolocation via the browser's Geolocation API.
- If the user grants permission, coordinates are sent to the API and stored with the scan.
- If the user denies or the browser doesn't support geolocation, the redirect proceeds without coordinates.
- The interstitial is minimal — a brief message ("Redirecting...") — and adds no more than 500ms to the user experience.

### 4.4 Analytics Dashboard

**Description:** The director views scan analytics in real-time, with filtering and export. Realizes UJ-2.

**Functional Requirements:**

#### FR-10: View campaign analytics

The director can view analytics for a specific campaign.

**Consequences (testable):**
- Dashboard shows: total scans, scans today, scans this week, scans this month.
- Time-series chart of scans over time (configurable: 24h, 7d, 30d, all-time).
- Geographic breakdown: country, city (aggregated counts).
- Device type breakdown: mobile, desktop, tablet.
- Hall-level attribution: if geolocation data is available, scans are clustered and displayed on a map.

#### FR-11: Export scan data

The director can export raw scan data for a campaign.

**Consequences (testable):**
- Export formats: CSV and JSON.
- Export includes all captured fields for every scan.
- Date range filter is available before export.
- Export is generated server-side and downloaded as a file.
- Export is limited to the most recent 100,000 scans per request.

### 4.5 QR Code Generation

**Description:** The system generates QR code images for each campaign's static redirect URL.

**Functional Requirements:**

#### FR-12: Generate QR code

The system generates a QR code image for a campaign's redirect URL.

**Consequences (testable):**
- QR code encodes the full URL: `https://cineforge.io/r/{campaign-id}`.
- QR code is generated as a PNG image at 300x300px minimum.
- QR code is downloadable from the campaign detail page.
- QR code is regenerated automatically if the campaign ID changes (it won't, but the system handles it).

**Out of Scope:**
- Custom QR code colors, logos, or frames.

## 5. Non-Goals (Explicit)

- CineForge will **not** host campaign landing pages (polls, forms, giveaways, etc.). It redirects to external URLs only.
- CineForge will **not** support multi-user access, roles, or permissions.
- CineForge will **not** provide A/B testing or multi-destination redirects.
- CineForge will **not** integrate with CRM systems or provide audience segmentation.
- CineForge will **not** ingest showtime schedules or cinema calendars.
- CineForge will **not** provide a native mobile app (the dashboard is responsive web).

## 6. MVP Scope

### 6.1 In Scope

- Single-user web dashboard with email/password authentication
- Campaign CRUD (create, read, update redirect URL, delete)
- Static QR code generation per campaign (PNG download)
- Redirect service with scan metadata capture (IP, user-agent, timestamp, GeoIP)
- Geolocation permission request on the redirect interstitial page
- Hall-level attribution via browser Geolocation API
- Real-time analytics dashboard (total scans, time-series, geography, device breakdown)
- Raw data export (CSV and JSON)
- Responsive web UI (React/Vite/Tailwind)
- Mobile-first redirect interstitial page
- Deployment: Vercel (frontend), Render (.NET backend), Neon (PostgreSQL)

### 6.2 Out of Scope for MVP

- Email digest (deferred — revisit post-v1)
- Custom QR code design (colors, logos, frames)
- Password reset flow (account is provisioned during setup)

## 7. Success Metrics

**Primary**

- **SM-1**: Redirect latency — P95 response time under 200ms for the redirect endpoint. Validates FR-7.
- **SM-2**: Scan data completeness — 100% of scans recorded with no data loss. Validates FR-8.
- **SM-3**: Director autonomy — 100% of campaign destination changes made through the UI without developer involvement. Validates FR-4.

**Secondary**

- **SM-4**: Dashboard usability — a first-time director can change the redirect URL and view scan counts within 2 minutes of logging in (measured in user testing). Validates FR-4, FR-10.
- **SM-5**: Geolocation opt-in rate — at least 40% of scans include geolocation coordinates. Validates FR-9.

**Counter-metrics (do not optimize)**

- **SM-C1**: Redirect interstitial dwell time — do not optimize for users spending time on the interstitial page. The goal is to get them to the campaign destination as fast as possible.

## 8. Open Questions

*All resolved during review.*

1. **GeoIP service** — ✅ MaxMind GeoLite2 (free, self-hosted).
2. **QR code library** — ✅ QRCoder for .NET (MIT-licensed).
3. **Rate limiting** — ✅ 100 requests/second per IP on the redirect endpoint.
4. **Data retention** — ✅ Indefinitely for v1, with a future archive/delete mechanism.

## 9. Assumptions Index

*All confirmed during review.*

- §4.1 FR-1: Account is provisioned during deployment/setup — no registration flow in v1.
- §4.1 FR-1: 5 failed attempts trigger a 15-minute lockout.
- §4.1 FR-2: Session token stored as HTTP-only secure cookie.
- §4.3 FR-8: GeoIP lookup uses MaxMind GeoLite2 (free, self-hosted).
- §4.3 FR-8: Scan recording is async (fire-and-forget) to avoid adding redirect latency.
- §4.5 FR-12: QR code generation uses QRCoder for .NET.
- §6.1: Backend deployed on Render (free tier, .NET support).
- §8.3: Rate limit of 100 requests/second per IP on the redirect endpoint.
- §8.4: Scan data retained indefinitely in v1.
