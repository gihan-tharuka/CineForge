# Epic 3: Redirect Service

Scanners who visit the public redirect endpoint are captured (IP, GeoIP, device, optional geolocation) and redirected to the campaign's current URL via a lightweight interstitial page.

**FRs covered:** FR-7, FR-8, FR-9
**NFRs addressed:** NFR-1, NFR-2, NFR-5, NFR-12, NFR-13, NFR-15
**Stories:** 3.1, 3.2, 3.3, 3.4, 3.5

## Story 3.1: Implement redirect endpoint that issues 302 redirect

As a moviegoer,
I want to be redirected to the campaign's current URL when I visit the redirect endpoint,
So that I reach the campaign destination without delay.

**Acceptance Criteria:**

**Given** a campaign exists with a current redirect URL
**When** a GET request is made to `/r/{campaign-id}`
**Then** the system returns HTTP 302 with the `Location` header set to the campaign's current redirect URL
**And** the scan record is written asynchronously (fire-and-forget) before the redirect
**Given** the campaign does not exist
**When** a GET request is made to `/r/{invalid-id}`
**Then** the system returns HTTP 404
**Given** the campaign was deleted
**When** a GET request is made to `/r/{deleted-id}`
**Then** the system returns HTTP 410 Gone
**Given** the redirect endpoint is under load
**When** 100 requests/second are received from a single IP
**Then** the rate limiter allows the requests without error (rate limit is 100 req/s per IP)

## Story 3.2: Capture scan metadata (IP, GeoIP, user-agent, device type)

As a system,
I want to capture metadata for every scan request,
So that the director can analyze where scans come from.

**Acceptance Criteria:**

**Given** a scan request is received
**When** the redirect is processed
**Then** the system captures: IP address, timestamp (UTC), user-agent, referer header (if present), device type (parsed from user-agent), country and city (via GeoIP lookup on IP)
**And** all fields are stored in the Scans table associated with the campaign
**And** the GeoIP lookup uses MaxMind GeoLite2 loaded in memory at startup
**And** the lookup overhead is under 1ms per request
**Given** the GeoIP lookup fails
**When** the scan is recorded
**Then** the scan is still recorded with country and city as null
**Given** the database write fails
**When** the scan is being recorded
**Then** the scan is lost (acceptable for v1 per AD-5)

## Story 3.3: Implement interstitial page with geolocation permission

As a moviegoer,
I want a lightweight interstitial page that requests my location before redirecting,
So that the director can see which cinema hall I scanned from.

**Acceptance Criteria:**

**Given** a moviegoer visits `/r/{campaign-id}`
**When** the interstitial page loads
**Then** it displays a brief message: "Redirecting to your campaign…"
**And** it requests geolocation via the browser's Geolocation API immediately on load
**Given** the moviegoer grants location permission
**When** the coordinates are received
**Then** they are sent to the API and stored with the scan
**And** the redirect proceeds to the campaign URL
**Given** the moviegoer denies location permission
**When** the permission flow completes
**Then** the redirect proceeds without coordinates (no error shown)
**Given** the browser doesn't support geolocation
**When** the interstitial loads
**Then** the redirect proceeds without coordinates (no error shown)
**Given** the interstitial page is loaded on a 3G connection
**When** the page renders
**Then** the total page weight is under 5KB
**And** the first paint occurs within 100ms

## Story 3.4: Implement rate limiting middleware

As a system administrator,
I want the redirect endpoint to be rate-limited per IP,
So that abusive traffic doesn't overwhelm the service.

**Acceptance Criteria:**

**Given** the redirect endpoint is deployed
**When** requests are received
**Then** the rate limit is 100 requests/second per IP
**And** requests exceeding the limit receive HTTP 429 Too Many Requests
**And** the rate limiter is implemented as middleware in the .NET backend
**Given** the redirect endpoint is under normal load
**When** 50 requests/second are received from a single IP
**Then** all requests are processed successfully
**Given** the redirect endpoint is under attack
**When** 200 requests/second are received from a single IP
**Then** requests above 100/second receive HTTP 429

## Story 3.5: Handle deleted/non-existent campaigns (404/410)

As a system,
I want to return appropriate HTTP status codes for invalid and deleted campaigns,
So that scanners and search engines understand the campaign state.

**Acceptance Criteria:**

**Given** a campaign ID that doesn't exist in the database
**When** a GET request is made to `/r/{invalid-id}`
**Then** the system returns HTTP 404 Not Found
**And** a simple "Campaign not found" page is displayed
**Given** a campaign that was deleted (status = "deleted")
**When** a GET request is made to `/r/{deleted-id}`
**Then** the system returns HTTP 410 Gone
**And** a simple "This campaign is no longer active." page is displayed
**Given** a valid campaign ID
**When** a GET request is made to `/r/{valid-id}`
**Then** the system returns HTTP 302 with the redirect
