# SLT Platform Documentation

Internal documentation for the Sur La Table e-commerce platform covering system architecture, operational workflows, and local development setup.

## Repository Structure

```
slt-docs/
├── architecture/          # System design, SaaS inventory, and data flows
│   ├── 01-saas-inventory.md
│   ├── 02-high-level-architecture.md
│   ├── 03-data-and-request-flows.md
│   └── *.drawio            # Architecture diagrams
├── workflow/              # Role-based operational workflows
│   ├── 01-product-catalog-management.md
│   ├── 02-order-management-fulfillment.md
│   ├── 03-content-management.md
│   ├── 04-configuration-deployment.md
│   ├── 05-customer-service-admin.md
│   ├── 06-marketing-email.md
│   ├── 07-inventory-store-operations.md
│   ├── 08-integration-monitoring-jobs.md
│   └── *.drawio            # Workflow diagrams
├── draw.io/               # Additional system and component diagrams
├── LOCAL_DEVELOPMENT_SETUP.md
└── README.md
```

## Platform Overview

The platform consists of three main applications serving the storefront:

| Application | Stack | Coverage | Repository |
|---|---|---|---|
| **React PWA** | React 18, TypeScript, Node 22 | ~75% of pages (PLP, PDP, cart, checkout, search, account) | `slt-sfcc-react` |
| **SiteGenesis** | SFCC ISML/Rhino | ~25% of pages (culinary classes, home) | `slt-sfcc` |
| **BFF** | NestJS 11, TypeScript, Node 22 | API gateway integrating 47+ services | `slt-sfcc-bff` |

Configuration is managed via the `slt-sfcc-configs` repository with environment promotion from development through production.

## Documentation Sections

### [Architecture](architecture/README.md)

System design, SaaS vendor inventory, and data/request flow documentation with accompanying draw.io diagrams.

### [Workflows](workflow/README.md)

Step-by-step operational workflows organized by role — merchandisers, developers, customer service, marketing, operations, and more.

### [Server-Side Rendering](SSR.md)

How the React PWA renders pages on the server — SSR architecture, data fetching with `getProps`, hydration, middleware pipeline, and the proxy/SSR port relationship.

### [Local Development Setup](LOCAL_DEVELOPMENT_SETUP.md)

Prerequisites, environment configuration, and debugging guides for running the React PWA and BFF locally.

## Diagrams

Diagrams are authored in [draw.io](https://app.diagrams.net/) format. Open `.drawio` files with the draw.io desktop app or VS Code extension.
