# Epic 2: Campaign Management

The director can create, view, update, and delete campaigns, and generate QR codes for each campaign's static redirect URL. This epic also establishes the complete domain model (Campaign, Scan, UrlChangeLog, User entities) in the Domain layer, enabling parallel development of the redirect service and analytics in subsequent epics.

**FRs covered:** FR-3, FR-4, FR-5, FR-6, FR-12
**NFRs addressed:** NFR-3, NFR-9, NFR-11
**Stories:** 2.1, 2.2, 2.3, 2.4, 2.5, 2.6, 2.7, 2.8

## Story 2.1: Create campaign entity and database migration

As a developer,
I want the Campaign entity and its database migration defined in the Domain layer,
So that campaign data can be persisted and managed.

**Acceptance Criteria:**

**Given** the Domain layer is being built
**When** the Campaign entity is created
**Then** it has fields: Id (GUID), Name (required, 1-100 chars), CurrentRedirectUrl (required, valid HTTP/HTTPS URL), Status (active/deleted), CreatedAt, UpdatedAt
**And** a UrlChangeLog entity exists with fields: Id, CampaignId, PreviousUrl, NewUrl, ChangedAt
**And** an EF Core migration creates the Campaigns and UrlChangeLogs tables
**And** the entity maps 1:1 to the architecture spine specification

## Story 2.2: Implement create campaign with name and initial redirect URL

As a director,
I want to create a new campaign with a name and an initial redirect URL,
So that I can start capturing scans for a new marketing effort.

**Acceptance Criteria:**

**Given** the director is on the campaign list page
**When** they click "Create campaign" and enter a name and valid redirect URL
**Then** a new campaign is created with status "active"
**And** a unique campaign ID is generated
**And** the campaign appears in the list
**And** the director is navigated to the campaign detail page
**Given** the director enters an invalid URL
**When** they click "Create"
**Then** the button remains disabled
**And** an inline error shows: "Invalid URL. Please enter a valid http:// or https:// address."
**Given** the director enters a name longer than 100 characters
**When** they try to create
**Then** the button is disabled and an error shows: "Name must be 1-100 characters."

## Story 2.3: Implement view campaign list

As a director,
I want to see all my campaigns in a list view,
So that I can quickly find and manage any campaign.

**Acceptance Criteria:**

**Given** the director is authenticated and on the campaign list page
**When** the page loads
**Then** all active campaigns are displayed as cards
**And** each card shows: campaign name, truncated redirect URL (mono), total scan count, last scan date
**And** campaigns are sorted by most recently updated first
**Given** there are no campaigns
**When** the page loads
**Then** an empty state displays: "No campaigns yet." with a "Create campaign" button
**Given** the director clicks on a campaign card
**When** the click registers
**Then** they are navigated to the campaign detail page
**Given** the page is loading
**When** data is being fetched
**Then** 4-6 skeleton cards display matching the expected layout

## Story 2.4: Implement update redirect URL

As a director,
I want to change the redirect URL of an active campaign at any time,
So that scanners are sent to a new destination without reprinting materials.

**Acceptance Criteria:**

**Given** the director is on the campaign detail page
**When** they change the redirect URL and click "Save"
**Then** the new URL takes effect immediately (next scan is redirected to the new destination)
**And** an audit record is appended to UrlChangeLog with previous URL, new URL, and timestamp
**And** a success toast displays: "Redirect updated. Next scan goes to your new destination."
**Given** the director enters an invalid URL
**When** they attempt to save
**Then** the save button is disabled
**And** an inline error shows: "Invalid URL. Please enter a valid http:// or https:// address."
**Given** the director has not changed the URL
**When** they view the save button
**Then** the save button is disabled (no changes to save)

## Story 2.5: Implement delete campaign with confirmation

As a director,
I want to delete a campaign with a confirmation step,
So that I can clean up campaigns I no longer need without accidental deletion.

**Acceptance Criteria:**

**Given** the director is on the campaign detail page
**When** they click "Delete campaign"
**Then** a confirmation modal appears: "Are you sure? This will deactivate the redirect URL."
**And** the modal has "Cancel" (ghost) and "Delete campaign" (destructive) buttons
**Given** the director clicks "Cancel"
**When** the modal closes
**Then** the campaign remains active and no changes are made
**Given** the director clicks "Delete campaign"
**When** the deletion completes
**Then** the campaign status changes to "deleted"
**And** the redirect URL returns HTTP 410 Gone
**And** scan data is retained for historical analytics
**And** the campaign list shows a "Deleted" status pill

## Story 2.6: Generate QR code for campaign redirect URL

As a director,
I want a QR code image generated for my campaign's static redirect URL,
So that I can print it on physical materials.

**Acceptance Criteria:**

**Given** a campaign exists
**When** the QR code is generated
**Then** it encodes the full URL: `https://cineforge.io/r/{campaign-id}`
**And** it is generated as a PNG image at 300x300px minimum
**And** it is downloadable from the campaign detail page
**Given** the director clicks the download button
**When** the download completes
**Then** a PNG file is saved to the director's device
**Given** the campaign ID changes (edge case)
**When** the QR code is regenerated
**Then** it encodes the new redirect URL

## Story 2.7: Implement campaign detail page with QR code display

As a director,
I want to see the campaign detail page with the redirect URL editor and QR code,
So that I can manage the campaign and download its QR code.

**Acceptance Criteria:**

**Given** the director navigates to `/campaigns/{id}`
**When** the page loads
**Then** the redirect URL editor is pre-filled with the current redirect URL
**And** the QR code image is displayed below the editor
**And** a download button is below the QR code
**And** a "Copy" icon is next to the redirect URL and campaign ID
**Given** the director clicks the copy icon
**When** the copy action completes
**Then** a toast displays: "Copied."
**Given** the director clicks the "Analytics" tab
**When** the tab switches
**Then** the analytics page loads (with placeholder data if no scans yet)

## Story 2.8: Define Scan entity in Domain layer

As a developer,
I want the Scan entity defined in the Domain layer,
So that the redirect service (Epic 3) can populate it and the analytics dashboard (Epic 4) can read it.

**Acceptance Criteria:**

**Given** the Domain layer is being built in Epic 2
**When** the Scan entity is created
**Then** it has fields: Id (bigint PK), CampaignId (FK), IpAddress, Country, City, Latitude (nullable), Longitude (nullable), UserAgent, DeviceType, Timestamp (UTC)
**And** a database migration creates the Scans table
**And** the entity is referenced by the Campaign entity (one-to-many)
**And** the entity is available for use by Epic 3 (redirect service) and Epic 4 (analytics)
