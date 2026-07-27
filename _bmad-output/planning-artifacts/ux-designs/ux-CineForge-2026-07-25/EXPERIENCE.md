---
name: CineForge
status: final
sources:
  - '../../../briefs/brief-CineForge-2026-07-25/brief.md'
  - '../../../prds/prd-CineForge-2026-07-25/prd.md'
  - '../../../architecture/architecture-CineForge-2026-07-25/ARCHITECTURE-SPINE.md'
updated: 2026-07-25
---

# CineForge — Experience Spine

> Dynamic QR redirect and analytics platform for film directors. Web dashboard (desktop-first, responsive) + backend-served redirect interstitial (mobile-first). Paired with `DESIGN.md` (CineForge visual identity). Demonstrates: two-surface architecture (SPA dashboard + backend interstitial), paste-a-URL redirect workflow, hall-level attribution via geolocation, single-director simplicity.

## Foundation

Web dashboard (desktop-first, responsive) built with shadcn/ui on React 19 + Vite 6 + Tailwind CSS 4 + TanStack Router/Table/Query. [ASSUMPTION: shadcn/ui not explicitly named in architecture spine but is the de facto standard for React + Tailwind.] The dashboard is the director's surface — where they manage campaigns, view analytics, and export data.

The redirect interstitial is a separate surface — served by the .NET backend as a Razor view (AD-2), not by the React SPA. It is a flat, framework-free HTML page that requests geolocation, sends coordinates to the API, then redirects. No bundle, no framework, no chrome. This is the moviegoer's surface — they scan, they're redirected, they're gone.

`DESIGN.md` is the visual identity reference and names the override surface; this spine is the experience. Single-director system — no multi-user, no roles, no permissions. One login, one dashboard, one person in control.

## Information Architecture

| Surface | Route | Reached from | Purpose |
|---|---|---|---|
| Login | `/login` | App open (unauthenticated) | Enter email + password. 5 failed attempts → 15-minute lockout. |
| Campaign list | `/` | App open (authenticated) / nav | List all campaigns with scan counts. Create new campaign. |
| Campaign detail | `/campaigns/:id` | Campaign list row | Edit redirect URL (primary action). View QR code. See scan count and last scan date. |
| Analytics | `/campaigns/:id/analytics` | Campaign detail → Analytics tab | View scan analytics: time-series, geography, device breakdown, hall-level attribution. Export data. |
| Settings | `/settings` | Nav | Theme toggle (light/dark). [Future: email digest config.] |
| Redirect interstitial | `/r/:campaignId` | QR scan (public, unauthenticated) | Request geolocation permission. Redirect to campaign URL. Served by backend. |

Navigation: top bar with app name (CineForge), theme toggle, and session menu (logout). Sidebar collapses to icons on `md`; on `sm` it becomes a Sheet triggered from the top bar. Modal stacks one level deep — never two.

→ Composition reference: `mockups/login.html`, `mockups/campaign-list.html`, `mockups/campaign-detail.html`, `mockups/analytics.html`, `mockups/interstitial.html`. Spine wins on conflict.

## Voice and Tone

Microcopy. Brand voice and aesthetic posture live in `DESIGN.md`.

| Do | Don't |
|---|---|
| "Redirect updated. Next scan goes to your new destination." | "Success! Your URL has been updated." |
| "No campaigns yet. Create your first campaign." | "You haven't created any campaigns. Click below to get started!" |
| "Invalid URL. Please enter a valid http:// or https:// address." | "Error: URL validation failed." |
| "Are you sure? This will deactivate the redirect URL." | "Confirm deletion" |
| "Redirecting to your campaign…" (interstitial) | "Please wait while we redirect you…" |
| "Location permission helps us show which cinema hall the scan came from." | "We need your location to function." |

Tone is professional, direct, and calm. No exclamation marks. No emojis. No streaks, badges, or achievement notifications. The tool gets out of the way.

## Component Patterns

Behavioral. Visual specs live in `DESIGN.md.Components`.

| Component | Use | Behavioral rules |
|---|---|---|
| Login form | `/login` | Email input (type=email), password input (type=password). Submit disabled until both fields are non-empty. On submit, show spinner. On error, show generic "Invalid email or password" (no user enumeration). 5 failed attempts → 15-minute lockout with countdown timer. |
| Campaign card | Campaign list | Click anywhere on card opens campaign detail. Shows: campaign name, truncated redirect URL (mono, ellipsized), total scan count, last scan date. Hover (md+) reveals "Edit" quick-action button. Deleted campaigns show a "Deleted" status pill and are not clickable. |
| Create campaign form | Campaign list (modal) | Name input (required, 1-100 chars). Redirect URL input (required, valid HTTP/HTTPS). "Create" button disabled until both valid. On success, closes modal and navigates to the new campaign detail. |
| Redirect URL editor | Campaign detail | Primary surface. URL input pre-filled with current redirect URL. "Save" button (accent) disabled until URL changes and is valid. On save: optimistic update, show toast "Redirect updated. Next scan goes to your new destination." Invalid URL → inline error, save disabled. |
| QR code display | Campaign detail | PNG image (300x300px minimum) encoding `https://cineforge.io/r/{campaign-id}`. Download button below. Click-to-copy campaign URL. |
| Analytics time-series | Analytics | Chart.js or Recharts line chart. Configurable range: 24h, 7d, 30d, all-time. Updates in real-time (TanStack Query polling every 30s). Shows scan count per time bucket. |
| Analytics geography | Analytics | Country/city breakdown as a table. If geolocation data available, hall-level clusters on a map (Leaflet or Google Maps embed). |
| Analytics device breakdown | Analytics | Pie chart: mobile, desktop, tablet. Counts and percentages. |
| Export modal | Analytics → Export button | Format selector (CSV / JSON). Date range picker (from / to). "Export" button generates file server-side. Limited to 100,000 scans per request. Shows estimated row count before export. |
| Interstitial | `/r/:campaignId` (backend) | Flat HTML page. Brief message: "Redirecting to your campaign…" Geolocation permission requested immediately on load. If granted: coordinates sent to API, then redirect. If denied/unsupported: redirect without coordinates. No framework, no bundle, no nav. |
| Session menu | Top bar | Avatar or initials dropdown. "Logout" item. "Settings" item. |

## State Patterns

| State | Surface | Treatment |
|---|---|---|
| Unauthenticated | Any dashboard route | Redirect to `/login`. |
| Loading (cold) | Campaign list | shadcn `Skeleton` cards (4-6) match expected layout. Resolves on data. |
| Loading (analytics) | Analytics | shadcn `Skeleton` for chart and summary stats. Resolves on data. |
| Empty campaign list | Campaign list | `display-sm`: "No campaigns yet." Body: "Create your first campaign to get started." Single primary button: "Create campaign." |
| Invalid URL | Redirect URL editor | Inline error below input: "Invalid URL. Please enter a valid http:// or https:// address." Save button disabled. Border turns `{colors.destructive}`. |
| Login error | Login form | Toast (destructive): "Invalid email or password." No field-specific error. |
| Login lockout | Login form | "Too many failed attempts. Try again in {countdown}." Countdown timer updates every second. Submit disabled until lockout expires. |
| Session expired | Any dashboard route | Redirect to `/login` with query `?expired=true`. Login form shows: "Your session has expired. Please log in again." |
| Redirect saved | Campaign detail | Toast (success): "Redirect updated. Next scan goes to your new destination." Save button returns to idle state. |
| Delete confirmation | Campaign detail | Modal: "Are you sure? This will deactivate the redirect URL." Two buttons: "Cancel" (ghost) and "Delete campaign" (destructive). |
| Campaign deleted | Campaign list | Row shows "Deleted" status pill. Scan data retained for analytics. Redirect URL returns 410 Gone. |
| Offline | Global | Toast (destructive): "You're offline. Changes won't save until you reconnect." Form inputs disabled. Retry on reconnect. |
| Geolocation granted | Interstitial | Coordinates sent to API (fire-and-forget). Redirect proceeds. |
| Geolocation denied | Interstitial | Redirect proceeds without coordinates. No error shown. |
| Geolocation unsupported | Interstitial | Redirect proceeds without coordinates. No error shown. |
| Interstitial error | Interstitial | If campaign not found: 404 page. If deleted: 410 page. Both served by backend. |

## Interaction Primitives

- **Click to edit.** The redirect URL input is the primary interaction on the campaign detail page. It is pre-filled and ready to edit on focus. Tab or click Save to persist.
- **Click to navigate.** Click anywhere on a campaign card to open its detail. Click the Analytics tab to switch views.
- **Copy to clipboard.** Click the copy icon next to the redirect URL or campaign ID to copy to clipboard. Toast: "Copied."
- **Download QR code.** Click the download button below the QR code to download the PNG.
- **Export with date range.** Select format (CSV/JSON), set date range, click Export. File downloads server-side.
- **Theme toggle.** Click the sun/moon icon in the top bar to toggle light/dark mode. Persists across sessions.
- **Logout.** Click the session menu in the top bar, then "Logout." Clears session cookie, redirects to `/login`.
- **Interstitial redirect.** The interstitial page auto-requests geolocation on load. No user action required beyond the browser's permission prompt. Redirect happens automatically after the permission flow completes (or immediately if denied/unsupported).

**Banned everywhere:** infinite scroll (pagination only), hover-only affordances on `sm` viewports, modal stacks > 1 level deep, celebratory animations, streaks, badges, auto-playing media.

## Accessibility Floor

Behavioral. Visual contrast lives in `DESIGN.md` (all color pairs verified to meet WCAG 2.1 AA at both light and dark modes).

- WCAG 2.1 AA across the dashboard surface. The interstitial meets WCAG 2.1 AA on mobile.
- Screen reader announces page surface on navigation: "Campaign list, 3 campaigns" / "Campaign detail, {campaign name}" / "Analytics, {campaign name}."
- Keyboard navigation: Tab order matches reading order. Focus rings use `{colors.accent}` at AA contrast against `{colors.surface-base}`.
- All interactive elements have visible focus states. Esc closes the topmost modal/popover.
- Form labels are associated with inputs via `htmlFor`/`id`. Error messages are announced via `aria-live`.
- QR code image has `alt` text: "QR code for {campaign name}. Scans redirect to cineforge.io/r/{campaign-id}."
- Interstitial: the geolocation permission prompt is the browser's native dialog — no custom modal. The "Redirecting…" message is announced via `aria-live`.
- Color is never the only indicator of state — error states also use text and iconography.
- Tap targets ≥ 44px on mobile (interstitial buttons, dashboard touch targets).

## Responsive & Platform

| Breakpoint | Behavior |
|---|---|
| `≥ lg` (1024px+) | Full dashboard layout. Campaign list is a single-column card list. Analytics uses a 2-column grid (chart + summary stats). |
| `md` (768–1023px) | Sidebar collapses to icons. Analytics stacks to single column. Campaign cards show less detail (no hover quick-actions). |
| `< md` (`sm`) | Sidebar becomes a Sheet triggered from top bar. Analytics stacks fully. Campaign list shows name + scan count only. |

The dashboard is responsive web — works on phones for quick checks but the primary surface is desktop/laptop. The redirect interstitial is mobile-first — it is the only surface a moviegoer sees, and it must load instantly on a phone.

The interstitial is served by the .NET backend (Razor view), not the React SPA. It uses inline CSS and inline JavaScript — no external stylesheets, no framework bundle. Total page weight < 5KB. First paint < 100ms on 3G.

## Inspiration & Anti-patterns

- **Lifted from URL shorteners (Bitly, etc.):** the paste-a-URL-and-save workflow. CineForge adds campaign management and analytics on top.
- **Lifted from analytics dashboards (Plausible, Simple Analytics):** the calm, data-forward surface. No streaks, no badges, no celebratory animation.
- **Lifted from developer tools (Vercel, Render):** the flat, focused surface. One primary action per screen. No chrome.
- **Rejected — Streaks, badges, achievement notifications:** CineForge is a tool, not a habit app. Scan counts are the metric, not daily login streaks.
- **Rejected — A/B testing or multi-destination redirects:** One campaign, one redirect URL. The director changes it, everyone gets the new one.
- **Rejected — Hosted campaign pages (polls, forms, giveaways):** CineForge redirects to external URLs only. The tool is a redirector + analytics, not a landing page builder.
- **Rejected — Multi-user / role-based access:** One director, one dashboard, one login. No team permissions, no workflow approvals.

## Key Flows

### Flow 1 — Launch a campaign (David, film director, final week before release)

1. David opens cineforge.app in his browser.
2. He is redirected to `/login`. He enters his email and password, clicks Sign in.
3. The campaign list loads. He clicks "Create campaign."
4. The create modal opens. He enters "Popcorn Bucket QR" as the name and pastes the YouTube interview URL as the redirect URL.
5. He clicks Create. The modal closes, and the campaign detail page loads.
6. **Climax:** David sees the campaign detail page. The redirect URL field is pre-filled with the YouTube link. Below it, the QR code is displayed. He downloads the QR code PNG and sends it to his designer: "Put this on every bucket." He closes the browser. The campaign is live.

Failure: David pastes an invalid URL → inline error appears below the field, Create button stays disabled. He corrects the URL and clicks Create again.

### Flow 2 — Change a redirect mid-campaign (David, three days later, surprise interview published)

1. David opens cineforge.app. He is already logged in (session cookie).
2. The campaign list shows "Popcorn Bucket QR" with 847 scans. He clicks the card.
3. The campaign detail page loads. The redirect URL field shows the old Facebook page URL.
4. He replaces it with the new YouTube interview URL.
5. He clicks Save.
6. **Climax:** A success toast appears: "Redirect updated. Next scan goes to your new destination." The Save button returns to idle. David closes the browser. The next person who scans the QR code is sent to the YouTube interview — no reprinting needed.

Failure: David pastes an invalid URL → inline error appears, Save button disabled. He corrects it and saves.

### Flow 3 — Check campaign performance (David, mid-week, on his phone)

1. David opens cineforge.app on his phone. He is logged in.
2. The campaign list shows "Popcorn Bucket QR" with 1,247 scans. He taps the card.
3. The campaign detail page loads. He taps the Analytics tab.
4. The analytics page loads. He sees: 1,247 total scans, 312 from the past 24 hours. A time-series chart shows the spike from opening weekend. A table shows geographic breakdown: Colombo (412), Kandy (187), Galle (98). A pie chart shows device breakdown: mobile (89%), desktop (11%).
5. He taps "Export." The export modal opens. He selects CSV, sets the date range to "Last 7 days," and clicks Export.
6. **Climax:** The CSV downloads. David opens it and sees all scan records with IP, country, city, timestamp, device type, and geolocation coordinates. He attaches it to an email to his distributor: "Here's the weekend data."

Failure: David's phone loses connection → toast: "You're offline. Changes won't save until you reconnect." He moves to a better signal and the data loads.

### Flow 4 — Scan the QR code (Amaya, moviegoer, Scope Cinema)

1. Amaya buys popcorn at Scope Cinema. She notices the QR code on the bucket and scans it with her phone camera.
2. Her browser opens `cineforge.io/r/campaign-abc123`.
3. The interstitial page loads instantly — a flat page with the message "Redirecting to your campaign…" and a geolocation permission prompt.
4. Amaya taps "Allow." Her phone sends coordinates to the API.
5. **Climax:** Amaya is redirected to the YouTube interview. Her scan is recorded: IP, country (Sri Lanka), city (Colombo), timestamp, device type (mobile), and location coordinates (Colombo, near Scope Cinema). The director sees this scan in the hall-level attribution on the analytics dashboard.

Failure: Amaya denies location permission → she is still redirected to the YouTube interview. No error is shown. Her scan is recorded without geolocation coordinates.

Failure: The campaign was deleted → Amaya sees a 410 Gone page: "This campaign is no longer active."
