# CineForge

**Dynamic QR redirect and analytics platform for film directors.**

CineForge turns a static printed QR code into a dynamic, measurable marketing channel. A single QR code printed on a popcorn bucket, poster, or cinema screen stays permanent — but what it points to changes whenever the director wants. One click in the dashboard, and every subsequent scan goes to a new destination. No reprinting. No lead time.

## The Problem

Film marketing has a physical blind spot. When a director prints QR codes on promotional materials, they are locked into that destination for the entire print run. If the campaign changes mid-run, the QR code is obsolete. Even when the QR works, the director gets near-zero signal — standard QR generators offer basic click counts at best, with no way to connect scans back to specific cinema halls, cities, or showtimes.

## The Solution

CineForge is a web application with two halves:

**1. The Director Dashboard** — the admin panel where the director:
- Sets the single redirect URL (any link — Facebook, a poll, a signup form, a live stream)
- Changes the destination at any time with immediate effect
- Views analytics: total scans, scans over time, geographic distribution, hall-level attribution
- Exports raw scan data (CSV/JSON)

**2. The Redirect Service** — the public-facing endpoint the static QR code points to:
- Captures scan metadata (IP, user-agent, timestamp, geolocation if permitted)
- Redirects the user to the current campaign URL via HTTP 302
- Tracks all data for the dashboard

The key insight: the QR code is permanent. It always points to `cineforge.io/r/{campaign-id}`. The destination behind that URL changes with a click.

## Who This Serves

**Primary: The film director.** Non-technical, time-poor during a release cycle, needs to change campaign destinations on the fly. Success for them means: "I changed the redirect in 30 seconds and the next scan went to the new page."

**Secondary: Cinema marketing partners.** The director may share dashboards or reports with cinema chains, but partners do not manage the system.

## What Makes This Different

- **Static QR, dynamic destination.** Every alternative forces a QR code change to change the destination. CineForge decouples the printed code from the live link.
- **Hall-level attribution without RFID or custom hardware.** By combining phone location data with scan timestamps, the director traces engagement to specific cinema halls.
- **Single-director simplicity.** One login, one dashboard, one person in control.
- **Full analytics in one place.** Not a redirector *and* an analytics tool stitched together. One system from scan to insight.

## Features (v1)

### Authentication
- Email/password login with session management
- Single-user — account provisioned during deployment
- Session expiry after 24 hours of inactivity
- Account lockout after 5 failed attempts

### Campaign Management
- Create campaigns with a name and initial redirect URL
- Change the redirect URL at any time — takes effect immediately
- View all campaigns in a list with scan counts and timestamps
- Delete campaigns (scan data retained for analytics)

### Redirect Service
- Public GET endpoint at `/r/{campaign-id}` issues HTTP 302 redirect
- Captures scan metadata: IP, timestamp, user-agent, device type, GeoIP (country/city)
- Optional geolocation via browser Geolocation API for hall-level attribution
- Rate-limited to 100 requests/second per IP
- Response time under 200ms at P95

### Analytics Dashboard
- Total scans, scans today/week/month
- Time-series chart (24h, 7d, 30d, all-time)
- Geographic breakdown by country and city
- Device type breakdown (mobile, desktop, tablet)
- Map with hall-level attribution clusters
- Raw data export in CSV and JSON

### QR Code Generation
- PNG download at 300x300px minimum
- Encodes the permanent CineForge redirect URL

## Tech Stack

| Component | Technology |
| --- | --- |
| **Frontend** | React 19, Vite 6, Tailwind CSS 4 |
| **State / API Client** | TanStack Query 5 |
| **Routing** | TanStack Router 1 |
| **Data Tables** | TanStack Table 8 |
| **Backend** | .NET 10, ASP.NET Core 10 |
| **ORM** | Entity Framework Core 10 |
| **Database** | PostgreSQL 16 (Neon) |
| **GeoIP** | MaxMind GeoLite2 |
| **QR Generation** | QRCoder for .NET |

## Deployment

- **Frontend:** Vercel
- **Backend:** Render (.NET)
- **Database:** Neon (PostgreSQL)

## Vision

In 2–3 years, CineForge becomes the standard engagement layer for physical cinema marketing. The QR code on a popcorn bucket is expected, not novel. Directors plan campaigns in advance, schedule redirect changes to align with release milestones, and get automated reports for their distributor and cinema partners. The analytics layer expands to cover multiple cinema chains simultaneously, with per-hall and per-showtime performance benchmarking. CineForge also expands beyond cinema into live events, concerts, and exhibitions — anywhere a static QR meets a dynamic campaign.

## License

MIT