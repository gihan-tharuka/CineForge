# Design Paradigm

**Clean / Hexagonal Architecture** — the backend is organised into four layers with strict dependency rules:

```
┌─────────────────────────────────────────────┐
│               Presentation                   │
│   (Controllers, Middleware, Razor Views)     │
│         ↓ depends on                        │
├─────────────────────────────────────────────┤
│               Application                    │
│   (Use Cases, DTOs, Port Interfaces)         │
│         ↓ depends on                        │
├─────────────────────────────────────────────┤
│                Domain                        │
│   (Entities, Value Objects, Domain Logic)    │
│         ↓ depends on                        │
├─────────────────────────────────────────────┤
│             Infrastructure                   │
│   (EF Core, GeoIP, Repos, Email — if added) │
└─────────────────────────────────────────────┘
```

Rules:
- **Domain** has zero dependencies on any other layer or external framework.
- **Application** depends only on Domain and its own port interfaces.
- **Infrastructure** implements interfaces defined in Application/Domain.
- **Presentation** depends on Application (not directly on Infrastructure).
