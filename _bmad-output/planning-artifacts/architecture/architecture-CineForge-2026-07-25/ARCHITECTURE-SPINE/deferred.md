# Deferred

| Decision | Reason to wait |
| --- | --- |
| Separate redirect service deployment | If redirect volume exceeds single-instance capacity, split into two Render services sharing the same DB. Easy post-v1. |
| Background queue for scan writes | If DB write latency becomes a bottleneck at scale. Add Channel<T> or Hangfire. |
| Scheduled URL changes | Not in v1 scope. Would require Separate RedirectConfig table (Option B from AD-3). |
| Email digest service | Deferred from v1 per PRD decision. |
| Multi-user / role-based access | Deferred from v1 per PRD decision. |
| Redis session store | If single-instance session storage becomes a limitation when scaling horizontally. |
| Serilog / structured logging sinks | v1 uses `ILogger<T>` console output. Log aggregation deferred. |
| CI/CD pipeline definition | Manual deploy via Render / Vercel dashboards is sufficient for v1. Automate post-v1. |