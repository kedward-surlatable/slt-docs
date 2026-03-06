# SLT Platform Documentation Repository

Internal documentation for the Sur La Table e-commerce platform. This is a **docs-only repo** — no application code, no build system, no tests.

## Repository Structure

```
slt-docs/
├── architecture/       # System design, SaaS inventory, data flows
├── workflow/           # 8 role-based operational workflows
├── code-uml/          # 10 draw.io UML diagrams
├── features/          # Feature-specific docs with diagrams
├── salesforce/        # Salesforce-specific notes
├── LOCAL_DEVELOPMENT_SETUP.md
├── SSR.md
└── README.md
```

## Platform Context

Four repositories make up the SLT platform:

| Repo | Stack | Purpose |
|------|-------|---------|
| `slt-sfcc-react` | React 18, TypeScript, Node 22, Chakra UI, Tailwind | PWA frontend (~75% of pages) |
| `slt-sfcc` | SFCC ISML/Rhino, jQuery | Legacy SiteGenesis (~25% of pages) |
| `slt-sfcc-bff` | NestJS 11, TypeScript, Node 22 | BFF API gateway (47+ integrations) |
| `slt-sfcc-configs` | Git-based configs | Environment configuration management |

### Key Architecture

- **Commerce Platform**: Salesforce Commerce Cloud (SFCC) — migrating off in 18-24 months
- **Rendering**: SSR via Express + React hydration (PWA Kit), proxy on :8000 routes to SSR on :3000
- **API Pattern**: BFF gateway on :1337, all external APIs go through BFF modules
- **Search**: Bloomreach (search, suggest, recommendations)
- **Integrations**: SVS, Cybersource, PayPal, Afterpay, Apple Pay, UPS, GoLocal, TurnTo, Radar, Emarsys/Braze, Narvar, Extend, Heap, ContentSquare

## Writing & Editing Docs

- All documentation is **Markdown** (`.md`)
- Diagrams are **draw.io** (`.drawio`) — open with draw.io desktop app or VS Code extension
- Architecture docs are numbered sequentially (`01-`, `02-`, etc.)
- Workflow docs follow the same numbering convention
- UML diagrams in `code-uml/` are numbered 01-10

## Conventions

- Keep Markdown files concise with tables for structured data
- Use draw.io diagrams for visual architecture, not ASCII art
- Feature docs go in `features/<feature-name>/` with both `.md` and `.drawio` files
- Reference specific repos (`slt-sfcc-react`, `slt-sfcc-bff`, etc.) when discussing code
- Environment tiers: development (sandbox1-7) → staging → production