---
title: "Product Brief: CineForge"
status: final
created: 2026-07-25T08:57:00+05:30
updated: 2026-07-25T13:22:00+05:30
---

# Product Brief: CineForge

## Executive Summary

CineForge is a dynamic QR redirect and analytics platform built for film directors and cinema marketers. It solves a fundamental tension in physical marketing: a QR code printed on a popcorn bucket or displayed on a cinema screen is permanent, but the campaign a director wants to run changes weekly.

A director prints a single QR code on every bucket in a production run, or displays it on-screen during a show. That QR code never changes. But what it points to — a Facebook page, a poll, a giveaway landing page, a live interview — changes whenever the director wants. One click in the dashboard, and the entire audience is redirected to a new destination. No reprinting. No new artwork. No lead time.

Behind the redirect, CineForge captures every scan: IP address, country, timestamp, and — using the phone's location permission — which cinema hall the scan came from. The director sees real-time analytics, gets daily digests, and can export raw data for deeper analysis. It turns a static printed surface into a dynamic, measurable marketing channel.

## The Problem

Film marketing has a physical blind spot. When a director prints QR codes on promotional materials — popcorn buckets, posters, screen interstitials — they are locked into that destination for the entire print run. If the campaign changes mid-run (a surprise guest announced, a poll going live, a giveaway ending), the QR code is obsolete. The marketing team either wastes the remaining inventory or accepts that the QR leads nowhere useful.

Even when the QR works, the director gets near-zero signal. How many people scanned? From which cities? Which cinema halls? Which showtimes drove the most engagement? Standard QR generators offer basic click counts at best. The director has no way to connect physical-world scans back to specific locations or times.

No single tool ties a static printed QR to a dynamic campaign that a non-technical director can manage alone. URL shorteners, separate analytics tools, and manual schedule cross-referencing are piecemeal alternatives.

## The Solution

CineForge is a web application with two halves:

**1. The Director Dashboard** — the admin panel where the director:
- Sets the single redirect URL (pastes any link — Facebook, a poll, a signup form, a live stream — and all scanners go there)
- Changes the destination at any time with immediate effect
- Views analytics: total scans, scans over time, geographic distribution, cinema-hall attribution
- Configures daily email digests
- Exports raw scan data (CSV/JSON)

**2. The Redirect Service** — the public-facing endpoint the static QR code points to:
- Captures scan metadata (IP, user-agent, timestamp, location if permitted)
- Redirects the user to the current campaign URL (302 redirect)
- Tracks all data for the dashboard

The key insight: the QR code is permanent. It always points to `cineforge.io/r/{campaign-id}`. The destination behind that URL changes with a click. No reprinting. No re-deployment. Just a redirect swap.

## What Makes This Different

- **Static QR, dynamic destination.** Every alternative forces a QR code change to change the destination. CineForge decouples the printed code from the live link.
- **Hall-level attribution without RFID or custom hardware.** By combining phone location data with scan timestamps, the director traces engagement to specific cinema halls and showtimes.
- **Single-director simplicity.** No team permissions, no workflow approvals, no role management. One login, one dashboard, one person in control.
- **Full analytics in one place.** Not a redirector *and* an analytics tool stitched together. One system from scan to insight.

## Who This Serves

**Primary: The film director.** Non-technical, time-poor during a release cycle, needs to change campaign destinations on the fly. Success for them means: "I changed the redirect in 30 seconds and the next scan went to the new page." They care about engagement data but don't want to become an analytics expert.

**Secondary: Cinema marketing partners.** The director may share dashboards or reports with cinema chains, but partners do not manage the system.

## Success Criteria

1. **Redirect latency:** The destination change propagates in under 10 seconds (TTL on the redirect logic).
2. **Scan attribution accuracy:** At least 80% of scans with location permission enabled are correctly mapped to a cinema hall.
3. **Director autonomy:** 100% of campaign destination changes are made by the director through the UI — zero developer involvement.
4. **Dashboard usability:** A first-time director can change the redirect URL and view scan counts within 2 minutes of logging in (measured in user testing).
5. **Data completeness:** Every scan is recorded. Raw data export includes all captured fields with no aggregation loss.

## Scope

**In scope — v1:**
- Single-user web dashboard with password auth
- One active redirect URL per campaign (paste-a-link; no hosted pages)
- QR code generation for the static CineForge redirect URL
- Scan analytics: IP, country, city, timestamp, user-agent, device type
- Hall-level attribution via browser Geolocation API
- Real-time analytics dashboard (last 24h, 7d, 30d, all-time)
- Daily email digest (configurable: on/off, time of day)
- Raw data export (CSV and JSON)
- Responsive web UI for the dashboard (React/Vite/Tailwind)
- Mobile-first redirect page (lightweight, fast, collects geolocation)

**Explicitly out of scope — v1:**
- Multi-user / role-based access
- Hosted campaign pages (polls, forms, giveaways)
- A/B testing or multi-destination redirects
- CRM integration or audience segmentation
- Native mobile app
- Custom QR code design (colors, logos, frames)
- Showtime schedule ingestion or calendar sync

## Vision

In 2-3 years, CineForge becomes the standard engagement layer for physical cinema marketing. The QR code on a popcorn bucket is expected, not novel. Directors plan campaigns in advance, schedule redirect changes to align with release milestones, and get automated reports sent to their distributor and cinema partners. The analytics layer expands to cover multiple cinema chains simultaneously, with per-hall and per-showtime performance benchmarking. CineForge also expands beyond cinema into live events, concerts, and exhibitions — anywhere a static QR meets a dynamic campaign.

But the core stays the same: one director, one dashboard, one QR code that never needs reprinting.