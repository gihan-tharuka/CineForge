# Capability → Architecture Map

| Capability / Area | Lives in | Governed by |
| --- | --- | --- |
| Authentication (FR-1, FR-2) | `AuthController` → `CineForge.Web` | AD-6 (Cookie sessions) |
| Campaign CRUD (FR-3, FR-4, FR-5, FR-6) | `CampaignsController` → `CampaignService` → `Campaign` entity | AD-3, AD-4 |
| Redirect Service (FR-7, FR-8, FR-9) | `RedirectController` → `ScanService` → `Scan` entity + `GeoIpService` | AD-1, AD-2, AD-5, AD-7 |
| Analytics (FR-10, FR-11) | `AnalyticsController` → `AnalyticsService` | AD-4 |
| QR Code (FR-12) | `CampaignsController` → QRCoder | AD-8 (EF Core for storage) |
| Data persistence | EF Core → `AppDbContext` → Neon PostgreSQL | AD-8 |
| GeoIP lookup | `GeoIpService` (Infrastructure) | AD-7 |
