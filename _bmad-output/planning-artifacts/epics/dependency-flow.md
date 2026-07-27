# Dependency Flow

```
Epic 1 (Auth) → Epic 2 (Campaigns) → [Epic 3 (Redirect) || Epic 4 (Analytics)]
```

- **Epic 1** is standalone (no dependencies)
- **Epic 2** depends on Epic 1 (auth required for dashboard access). Also establishes the complete domain model (Campaign, Scan, UrlChangeLog, User entities) in the Domain layer.
- **Epic 3** depends on Epic 2 (needs Campaign entity to redirect; redirect endpoint itself is public). Populates the Scan entity defined in Epic 2.
- **Epic 4** depends on Epic 2 (campaigns + Scan entity definition) and benefits from Epic 3 (scan data). Can be developed in parallel with Epic 3 once the domain model is established — the analytics UI can use mock scan data while Epic 3 implements the real capture.
- **Parallelization opportunity:** After Epic 2 completes, Epics 3 and 4 can be developed concurrently. Epic 3 populates Scan data; Epic 4 reads it. The data model is already defined in Epic 2's Domain layer.
- **Infrastructure prerequisites:** Frontend (React/Vite/Tailwind/shadcn/ui project setup) and backend (.NET solution, EF Core, database connection, migrations) must be established as foundational stories in Epic 1 before any frontend/backend work can proceed.
- Each epic delivers complete, testable functionality for its domain
