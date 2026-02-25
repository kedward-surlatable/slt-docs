# Sur La Table - Architecture Documentation

Architecture documentation for the Sur La Table e-commerce platform, extracted from team discussions, codebase analysis, and existing documentation.

---

## Documents

| # | Document | Description |
|---|----------|-------------|
| 01 | [SaaS & Third-Party Inventory](01-saas-inventory.md) | Complete inventory of all SaaS products, third-party services, and vendor platforms with costs and notes |
| 02 | [High-Level Architecture](02-high-level-architecture.md) | Platform architecture overview: frontends, BFF, backend services, deployment, repositories, and migration roadmap |
| 03 | [Data & Request Flows](03-data-and-request-flows.md) | How data flows through the system: web requests, search, product lifecycle, orders, returns, local dev |

## Diagrams (draw.io)

Open these in [diagrams.net](https://app.diagrams.net/) or the VS Code Draw.io extension:

| # | Diagram | Description |
|---|---------|-------------|
| 01 | [Platform Architecture](01-platform-architecture.drawio) | Full platform architecture: CDN, frontends, BFF, backend services, data layer, internal tools |
| 02 | [SaaS Integration Map](02-saas-integration-map.drawio) | Visual map of all third-party SaaS integrations organized by category |
| 03 | [Product Lifecycle](03-product-lifecycle.drawio) | Product data flow from ERP creation through PIM enrichment to website |

Additional architecture diagrams are in [../draw.io/](../draw.io/).

---

## Quick Reference

**Three Frontends:**
- **React PWA** (75%) - Modern pages: PLP, PDP, cart, checkout, search, account
- **SiteGenesis** (25%) - Legacy: culinary classes, home page
- **BFF** (NestJS) - API gateway with 47+ integration modules

**Key SaaS Services:**
- **Salesforce Commerce Cloud** - E-commerce engine (migrating off in 18-24 months)
- **Bloomreach** - Search, suggest, recommendations ($60k-$130k+/year)
- **Oracle OMS** - Order management (migrating to Kibo)
- **Narvar** - Returns processing
- **Emarsys** - Email marketing (migrating to Braze)

**Repositories:**
- `slt-sfcc` - SiteGenesis (SFCC backend)
- `slt-sfcc-react` - React PWA storefront
- `slt-sfcc-bff` - BFF API gateway (NestJS)
- `slt-sfcc-configs` - SFCC configuration

---

*Sources: Team huddle (Feb 24, 2026), codebase analysis, existing platform documentation.*
