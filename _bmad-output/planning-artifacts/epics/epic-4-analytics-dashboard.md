# Epic 4: Analytics Dashboard

The director can view real-time scan analytics (time-series, geography, device breakdown, hall-level attribution) and export raw scan data in CSV or JSON format.

**FRs covered:** FR-10, FR-11
**NFRs addressed:** NFR-4, NFR-10
**Stories:** 4.1, 4.2, 4.3, 4.4, 4.5, 4.6

## Story 4.1: Implement analytics overview (total scans, time-series chart)

As a director,
I want to see an overview of my campaign's scan analytics,
So that I can understand how my campaign is performing.

**Acceptance Criteria:**

**Given** the director is on the analytics page for a campaign
**When** the page loads
**Then** it displays: total scans, scans today, scans this week, scans this month
**And** a time-series chart shows scan counts over time
**And** the chart is configurable: 24h, 7d, 30d, all-time
**And** the data updates in real-time (TanStack Query polling every 30s)
**Given** there are no scans yet
**When** the page loads
**Then** a skeleton chart displays while loading
**And** after loading, the chart shows zero data with an appropriate empty state
**Given** the director selects "7d" range
**When** the chart updates
**Then** it shows scan counts for the past 7 days

## Story 4.2: Implement geographic breakdown (country, city)

As a director,
I want to see where my scans come from geographically,
So that I can understand my campaign's reach.

**Acceptance Criteria:**

**Given** the director is on the analytics page
**When** the geographic breakdown loads
**Then** it displays a table with country, city, and scan count
**And** countries are sorted by scan count (descending)
**Given** geolocation data is available for some scans
**When** the hall-level attribution loads
**Then** scans are clustered on a map (Leaflet or Google Maps embed)
**And** each cluster shows the cinema hall name and scan count
**Given** no geolocation data is available
**When** the map section loads
**Then** it displays a message: "No geolocation data available yet."
**Given** the director is on a mobile device
**When** the geographic breakdown loads
**Then** the table and map stack to single column

## Story 4.3: Implement device type breakdown

As a director,
I want to see the device types of my scanners,
So that I can understand how my audience is accessing the campaign.

**Acceptance Criteria:**

**Given** the director is on the analytics page
**When** the device breakdown loads
**Then** it displays a pie chart with mobile, desktop, and tablet
**And** each segment shows the count and percentage
**And** the data is derived from user-agent parsing during scan capture
**Given** all scans are from mobile devices
**When** the pie chart renders
**Then** it shows 100% mobile with no other segments
**Given** there are no scans
**When** the device breakdown loads
**Then** a skeleton displays while loading, then shows zero data

## Story 4.4: Implement hall-level attribution map

As a director,
I want to see which cinema halls my scans came from on a map,
So that I can attribute engagement to specific locations.

**Acceptance Criteria:**

**Given** geolocation data is available for scans
**When** the hall-level attribution map loads
**Then** scans are clustered and displayed on a map
**And** each cluster shows the cinema hall name and scan count
**And** clicking a cluster shows the individual scan locations
**Given** geolocation data is not available
**When** the map section loads
**Then** it displays: "No geolocation data available yet."
**Given** the director is on a desktop
**When** the map loads
**Then** it uses a 2-column grid layout (chart + summary stats)
**Given** the director is on a mobile device
**When** the map loads
**Then** it stacks to single column

## Story 4.5: Implement data export (CSV and JSON)

As a director,
I want to export raw scan data for a campaign,
So that I can share it with distributors or do deeper analysis.

**Acceptance Criteria:**

**Given** the director is on the analytics page
**When** they click "Export"
**Then** an export modal opens with format selector (CSV / JSON) and date range picker
**Given** the director selects CSV and sets a date range
**When** they click "Export"
**Then** a CSV file is generated server-side and downloaded
**And** the file includes all captured fields for every scan in the date range
**Given** the director selects JSON
**When** they click "Export"
**Then** a JSON file is generated server-side and downloaded
**And** the file includes all captured fields for every scan in the date range
**Given** the campaign has more than 100,000 scans
**When** the director attempts to export
**Then** the export is limited to the most recent 100,000 scans
**And** an estimated row count is shown before export

## Story 4.6: Implement export with date range filter

As a director,
I want to filter my export by date range,
So that I can export only the data I need.

**Acceptance Criteria:**

**Given** the director is on the export modal
**When** they set a "from" date and "to" date
**Then** the date range is validated (from ≤ to)
**And** the estimated row count updates to reflect the filtered range
**Given** the director sets an invalid date range (from > to)
**When** they attempt to export
**Then** an error displays: "Invalid date range. 'From' date must be before 'To' date."
**And** the export button is disabled
**Given** the director sets no date range
**When** they export
**Then** all scans (up to 100,000) are exported
**Given** the director sets a date range of "Last 7 days"
**When** they export
**Then** only scans from the past 7 days are included
