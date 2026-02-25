# Sur La Table - Internal Workflow Documentation

Documentation of internal user workflows for people who manage the Sur La Table e-commerce platform. These are **not** shopper-facing flows; they cover the operational, merchandising, content, and engineering workflows performed by internal teams.

---

## Workflow Index

| # | Workflow | Audience | Description |
|---|----------|----------|-------------|
| 01 | [Product & Catalog Management](01-product-catalog-management.md) | Merchandisers, Search Managers | Product data entry, category management, dynamic categorization rules, Bloomreach search configuration, product feeds |
| 02 | [Order Management & Fulfillment](02-order-management-fulfillment.md) | Operations, Customer Service, Finance | Order lifecycle, OMS export (Oracle), payment processing, warranty (Extend), PayPal transaction review |
| 03 | [Content Management](03-content-management.md) | Content Managers, SEO, Marketing | Content assets, content slots, URL redirects, recipes, sitemaps, Optiversal optimization |
| 04 | [Configuration & Deployment](04-configuration-deployment.md) | Developers, DevOps, Release Mgrs | Git-based config management, environment promotion, deployment of React/BFF/SFCC, environment variables |
| 05 | [Customer Service & Admin](05-customer-service-admin.md) | Customer Service Reps, Account Admins | Customer lookup, order support, password resets, gift registries, gift cards, wish lists, cooking classes |
| 06 | [Marketing & Email](06-marketing-email.md) | Marketing Managers, Email Ops | Emarsys configuration, transactional emails, newsletter subscriptions, SmartInsight analytics, GTM tracking |
| 07 | [Inventory & Store Operations](07-inventory-store-operations.md) | Store Ops, Inventory, Fulfillment | BOPIS, same-day delivery (GoLocal), store management, cooking class rosters, delivery estimation |
| 08 | [Integration Monitoring & Jobs](08-integration-monitoring-jobs.md) | Platform Engineers, Operations | Scheduled jobs, 47+ integration health, caching strategy, BM admin tools, service configuration |

---

## Diagrams

Each workflow has an accompanying draw.io diagram that can be opened in [diagrams.net](https://app.diagrams.net/) or the VS Code Draw.io extension:

| Diagram | File |
|---------|------|
| Product Catalog Management Flow | [01-product-catalog-management.drawio](01-product-catalog-management.drawio) |
| Order Management Lifecycle | [02-order-management-fulfillment.drawio](02-order-management-fulfillment.drawio) |
| Content Management Flow | [03-content-management.drawio](03-content-management.drawio) |
| Configuration & Deployment Flow | [04-configuration-deployment.drawio](04-configuration-deployment.drawio) |
| Customer Service Flow | [05-customer-service-admin.drawio](05-customer-service-admin.drawio) |
| Marketing & Email Flow | [06-marketing-email.drawio](06-marketing-email.drawio) |
| Inventory & Store Operations Flow | [07-inventory-store-operations.drawio](07-inventory-store-operations.drawio) |
| Integration Monitoring & Jobs | [08-integration-monitoring-jobs.drawio](08-integration-monitoring-jobs.drawio) |

Additional architecture diagrams are available in [../draw.io/](../draw.io/).

---

## Internal Roles Summary

| Role | Primary Workflows |
|------|------------------|
| **Merchandiser** | Product data, categories, pricing, promotions (01) |
| **Search Manager** | Bloomreach search rules, synonyms, facets (01) |
| **Content Manager** | Content assets, slots, recipes, landing pages (03) |
| **SEO Specialist** | URL redirects, sitemaps, meta tags, canonical URLs (03) |
| **Marketing Manager** | Emarsys campaigns, newsletters, GTM analytics (06) |
| **Customer Service Rep** | Customer lookup, order support, registries, gift cards (05) |
| **Operations Manager** | Order export monitoring, job scheduling, failure alerts (02, 08) |
| **Store Operations Mgr** | Store config, BOPIS, SDD, cooking class management (07) |
| **Inventory Coordinator** | Stock levels, BOPIS/SDD availability, store allocation (07) |
| **Culinary Program Mgr** | Cooking class scheduling, rosters, fill rates (07) |
| **Developer** | Code changes, local dev, PR workflow, testing (04) |
| **DevOps / Platform Eng** | Deployment pipelines, infrastructure, integration health (04, 08) |
| **Release Manager** | Cross-component release coordination (04) |
| **Finance** | PayPal transaction review, gift card management (02) |

---

## Key Systems

| System | Type | Used In |
|--------|------|---------|
| **SFCC Business Manager** | Admin UI | All workflows |
| **BFF API (NestJS)** | API Gateway | All workflows |
| **React PWA** | Customer storefront | 01, 03, 05 |
| **Bloomreach** | Search engine | 01, 06 |
| **Emarsys** | Email marketing | 05, 06 |
| **Oracle OMS** | Order management | 02 |
| **Extend** | Warranties | 02 |
| **SVS** | Gift cards | 02, 05 |
| **Cybersource/PayPal/Afterpay** | Payments | 02 |
| **UPS / FedEx** | Shipping | 02, 07 |
| **GoLocal** | Same-day delivery | 07 |
| **Radar** | Geolocation | 07 |
| **TurnTo** | Reviews & Q&A | 01, 03 |
| **Optiversal** | Content optimization | 01, 03 |
| **Rio SEO** | SEO / URLs | 03 |
| **Google Tag Manager** | Analytics | 06 |
| **AWS (SSM, Lambda, CloudWatch)** | Infrastructure | 04, 08 |
| **GitHub** | Source control | 04 |

---

## How These Docs Were Created

These workflow documents were inferred from analyzing:
- The SFCC Business Manager cartridges (`bm_*`) and their controllers, menus, and forms
- The BFF NestJS modules, controllers, and service files (47+ integration modules)
- The SFCC integration cartridges (`int_*`) including jobs, services, and custom objects
- The React PWA routes, pages, and component structure
- The configuration repository structure and deployment scripts
- The existing architecture diagrams in `draw.io/`

They represent the best understanding of actual platform workflows as implemented in source code. Corrections and updates are welcome.
