# Sur La Table - SaaS & Third-Party Services Inventory

An inventory of all external SaaS products, third-party services, and vendor platforms used by the Sur La Table e-commerce platform.

---

## Commerce Platform

| Service | Purpose | Cost | Notes |
|---------|---------|------|-------|
| **Salesforce Commerce Cloud (SFCC)** | Core e-commerce platform (catalog, cart, checkout, orders) | Hundreds of thousands $/year | Contract renewed for 18-24 months. Planning to migrate off. Originally built by Demandware (acquired by Salesforce). |
| **SFCC Business Manager** | Admin UI for merchandising, orders, jobs, site configuration | Included with SFCC | Merchant Tools (business-facing) + Admin (DevOps-facing) |
| **SFCC Managed Runtime (MRT)** | Hosting for the React PWA storefront | Included with SFCC | Deployed via "Commerce Cloud" deployment. SSR with Express. |
| **SFCC CDN (CloudFlare)** | CDN managed by Salesforce | Included with SFCC | Routes traffic between PWA and SiteGenesis. Configured by SLT. |

---

## Search & Recommendations

| Service | Purpose | Cost | Notes |
|---------|---------|------|-------|
| **Bloomreach** | Product search, auto-suggest, recommendations | ~$60k-$130k+/year | Expensive but effective. Quota-based pricing (~40M requests/year). Caching issues with PLP pages are causing overage. Running into API call quotas. Could potentially be replaced with AI-based solution. |

**Bloomreach Staging Endpoints:**
- Search: `https://staging-core.dxpapi.com/api/v1/core/`
- Suggest: `https://staging-suggest.dxpapi.com/api/v2/suggest/`
- Recommendations: `https://staging-recommend.dxpapi.com/api/v2/widgets/`

---

## Order Management & POS

| Service | Purpose | Cost | Notes |
|---------|---------|------|-------|
| **Oracle OMS** | Order management system | - | Only using OMS portion (not POS or commerce). Kibo migration in progress. |
| **Oracle POS** | Point of sale (in-store) | - | Used for physical retail stores. |
| **Kibo OMS** | Replacement OMS | - | Migration target for Oracle OMS. Has both OMS and commerce platform, but SLT only using OMS. |

---

## Returns & Shipping Protection

| Service | Purpose | Cost | Notes |
|---------|---------|------|-------|
| **Narvar** | Returns processing, order tracking, notifications | ~$6/RMA (estimated) | Expensive for what it provides. Processes return requests via webhook. Has a customer-facing portal. SLT built custom Lambda integrations around it. |
| **Saphonia** | Shipping protection (claims, exchanges, refunds) | - | Handles damage/loss claims. Creates replacement orders. Shares the Narvar integration Lambda monorepo (`narvar-integration`). |

---

## Payments

| Service | Purpose | Cost | Notes |
|---------|---------|------|-------|
| **Cybersource** | Primary payment processing | - | Integrated via SFCC cartridge (`int_cybersource`) |
| **PayPal** | Alternative payment method | - | Integrated via SFCC cartridge |
| **Afterpay** | Buy-now-pay-later | - | Integrated via SFCC cartridge |
| **Apple Pay** | Mobile/web payment | - | Implemented in PWA |
| **SVS** | Gift card processing | - | Integrated via BFF |

---

## Reviews & Ratings

| Service | Purpose | Cost | Notes |
|---------|---------|------|-------|
| **TurnTo** | Customer reviews, ratings, Q&A | - | Out-of-box UI was not good enough; SLT had to build custom React components. Could potentially be replaced by Bazaarvoice (used at One Kings Lane), which offers review syndication across sites and is cheaper. |

---

## Email & Marketing

| Service | Purpose | Cost | Notes |
|---------|---------|------|-------|
| **Emarsys (Marxist)** | Email marketing, transactional emails, newsletters | - | Currently in use. Being replaced by Braze. |
| **Braze** | Email marketing (replacement) | - | Future replacement for Emarsys. |
| **Google Tag Manager (GTM)** | Pixel management, third-party tracking, analytics | - | Two containers: client-side (prod + staging/dev) and server-side (prod, non-prod). Diego is the primary maintainer. Near capacity, requires cleanup when adding new tags. |

---

## Analytics & User Behavior

| Service | Purpose | Cost | Notes |
|---------|---------|------|-------|
| **Heap** | Product analytics, funnel analysis | - | Similar to Amplitude/Mixpanel. Business and product teams review regularly. |
| **Content Square** | Session recordings, user behavior analysis | - | Acquired or partnered with Heap. Useful for debugging specific user issues. |

---

## SEO & Content Optimization

| Service | Purpose | Cost | Notes |
|---------|---------|------|-------|
| **Rio SEO** | SEO services, URL management | - | Integrated via BFF |
| **Optiversal** | Content optimization | - | Integrated via BFF |

---

## Delivery & Logistics

| Service | Purpose | Cost | Notes |
|---------|---------|------|-------|
| **UPS** | Address validation, estimated delivery dates, shipping | - | Integrated via BFF |
| **GoLocal** | Same-day delivery | - | Has its own portal. Integrated via BFF. |
| **FedEx** | Shipping carrier | - | Additional carrier option |
| **Channel Advisor** | Marketplace listing (Amazon) | - | Ships product data to Amazon marketplace |

---

## Product Information & ERP

| Service | Purpose | Cost | Notes |
|---------|---------|------|-------|
| **Island Pacific** | ERP - product creation, inventory, pricing | - | Extremely old system (iSeries/DB2). Looks like a "Jurassic Park terminal." Start-of-life for all SKUs. New Balance also used this. Future migration needed. |
| **CSC PIM** | Product information management, enrichment | - | Replaced Stibo (10+ year old PIM). Products are enriched here before going to the website. Only sellable SKUs (not recipes). |

---

## Warranties

| Service | Purpose | Cost | Notes |
|---------|---------|------|-------|
| **Extend** | Extended warranty program | - | Has its own portal. Integrated via SFCC cartridge. |

---

## Geolocation

| Service | Purpose | Cost | Notes |
|---------|---------|------|-------|
| **Radar** | Geolocation services for store locator, BOPIS | - | Integrated via BFF |

---

## AB Testing

| Service | Purpose | Cost | Notes |
|---------|---------|------|-------|
| **AB Testing Tool** | Experiment management | - | Specific vendor not confirmed in transcript. Bloomreach also runs AI-enabled experiments (which conflict with caching strategy). |

---

## Internal Tools (SLT-Built)

| Tool | Stack | Purpose | Status |
|------|-------|---------|--------|
| **eCom Tools V1** | Django Admin + MongoDB | CRUD for recipes, stores, vendors. First internal operational tool. | Deprecated (should be retired) |
| **eCom Tools V2** | React (UI) + Python (API) + MongoDB | Recipes, stores, price overrides, Narvar returns, Saphonia claims, analytics, gift registry ops | Active. Not yet rolled out to all business users. |

---

## Infrastructure (AWS)

| Service | Purpose | Notes |
|---------|---------|-------|
| **AWS ECS** | BFF hosting (Elastic Container Service) | Production deployment target for BFF |
| **AWS Lambda** | Serverless functions | Narvar/Saphonia integrations, BFF Lambda deployment option |
| **AWS SSM Parameter Store** | Secret/config management | Stores all environment-specific secrets. BFF loads config from here (unless `SKIP_SSM=true`). |
| **AWS CloudWatch** | Logging, monitoring | BFF telemetry logs go here |
| **AWS S3** | File storage | CSV uploads, data imports |
| **GitHub** | Source control | All repos hosted here |

---

## Environments

All SFCC environments follow the pattern: `{env}-na01-surlatable.demandware.net`

| Environment | URL | Notes |
|-------------|-----|-------|
| **Development** | `development-na01-surlatable.demandware.net` | Dev sandbox |
| **Staging** | `staging-na01-surlatable.demandware.net` | Primary data loading target. Code + data replicated to prod from here. |
| **Production** | `production-na01-surlatable.demandware.net` | Live site. Cannot push code directly; must promote from staging. Only inventory updates go directly here. |

---

## Cost Optimization Opportunities

Based on the team huddle discussion, these services are candidates for replacement or renegotiation:

1. **Salesforce Commerce Cloud** - Hundreds of thousands/year. Plan to migrate off within contract window.
2. **Bloomreach** - $60k-$130k+/year. Could potentially be built in-house with AI/ML.
3. **Narvar** - Expensive per-RMA pricing. Core functionality (returns UI, order enrichment) could be built in-house.
4. **TurnTo** - Could be replaced by Bazaarvoice (cheaper, offers review syndication).
5. **Emarsys** - Already being replaced by Braze.

---

*Sources: Team huddle transcript (Feb 24, 2026), codebase analysis of slt-sfcc-bff integration modules, SFCC cartridge inventory.*
