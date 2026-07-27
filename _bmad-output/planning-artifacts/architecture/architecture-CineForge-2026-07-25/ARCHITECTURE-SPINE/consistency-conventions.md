# Consistency Conventions

| Concern | Convention |
| --- | --- |
| Naming (entities) | PascalCase, singular. `Campaign`, `Scan`, `User`, `UrlChangeLog` |
| Naming (API routes) | kebab-case plural. `/api/campaigns`, `/api/campaigns/{id}/analytics` |
| Naming (database tables) | Pluralised PascalCase. `Campaigns`, `Scans`, `Users`, `UrlChangeLogs` |
| IDs | GUID (string) for campaign IDs (public-facing). int/bigint for internal PKs. |
| Dates | UTC ISO 8601. Stored as `timestamptz` in PostgreSQL. |
| Error responses | Standard Problem Details (RFC 7807): `{ type, title, status, detail, instance }` |
| API versioning | URL prefix `/api/v1/` for future-proofing. v1 uses `/api/` (no version prefix). |
| Logging | Structured logging via `ILogger<T>` (Serilog sink deferred). |
