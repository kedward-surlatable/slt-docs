# Sur La Table - High-Level Platform Architecture

## Overview

Sur La Table's e-commerce platform is a headless Salesforce Commerce Cloud (SFCC) implementation. The customer-facing website is split between two frontend systems (SiteGenesis ~25% and React PWA ~75%) with a Backend-for-Frontend (BFF) layer that orchestrates calls to SFCC and 26+ third-party services.

The platform is in active migration from the legacy SiteGenesis frontend to React PWA, with a longer-term plan to migrate entirely off Salesforce Commerce Cloud within 18-24 months.

---

## System Architecture

```
                          ┌──────────────────────────────┐
                          │         CUSTOMERS            │
                          │   (Browser / Mobile / App)   │
                          └──────────────┬───────────────┘
                                         │
                          ┌──────────────▼───────────────┐
                          │    Salesforce CDN (CloudFlare)│
                          │    Routing: PWA vs SiteGenesis│
                          └───────┬──────────────┬───────┘
                                  │              │
                    ┌─────────────▼──┐    ┌──────▼─────────────┐
                    │  React PWA     │    │  SiteGenesis        │
                    │  (75% of site) │    │  (25% - culinary,   │
                    │                │    │   legacy pages)     │
                    │  React 18      │    │  Rhinoscript        │
                    │  TypeScript    │    │  ISML Templates     │
                    │  Node 22       │    │  Node 6.14          │
                    │  Chakra UI     │    │  jQuery              │
                    │                │    │                     │
                    │  Hosted on MRT │    │  Hosted on SFCC     │
                    └───────┬────────┘    └──────┬──────────────┘
                            │                    │
                            │ REST/JSON           │ Script API (SDK)
                            │                    │
                    ┌───────▼────────────────────▼───────────────┐
                    │              BFF (NestJS)                   │
                    │         Backend-for-Frontend                │
                    │         AWS ECS / Lambda                    │
                    │         Port 1337 (local)                   │
                    │                                            │
                    │  47+ Integration Modules                   │
                    └──┬─────┬──────┬──────┬──────┬─────────────┘
                       │     │      │      │      │
           ┌───────────▼┐ ┌─▼────┐ ▼   ┌──▼──┐ ┌─▼──────────┐
           │ SFCC SCAPI  │ │Bloom-│ ... │ UPS │ │  26+ other  │
           │ (Commerce)  │ │reach │     │     │ │  services   │
           └─────────────┘ └──────┘     └─────┘ └────────────-┘
```

---

## The Three Frontends

The website is served by two distinct SFCC products, with the CDN routing between them:

### 1. React PWA (~75% of pages)
- **Repo:** `slt-sfcc-react`
- **Tech:** React 18, TypeScript, Chakra UI, Tailwind CSS
- **Hosts:** Product listing pages (PLP), product detail pages (PDP), cart, checkout, account, search, gift registry, store locator
- **Deployment:** Salesforce Managed Runtime (MRT) via "Commerce Cloud" deploy
- **Runtime:** Node 22, Express SSR

### 2. SiteGenesis (~25% of pages)
- **Repo:** `slt-sfcc`
- **Tech:** Rhinoscript (JS compiled to Java), ISML templates, jQuery
- **Hosts:** Culinary/cooking class pages, home page (CMS-driven), some legacy pages
- **Deployment:** WebDAV upload to SFCC Business Manager
- **Runtime:** Node 6.14
- **Cannot run locally** - must be deployed to a sandbox

### 3. BFF - Backend-for-Frontend
- **Repo:** `slt-sfcc-bff`
- **Tech:** NestJS 11, TypeScript, Express
- **Purpose:** Unified API layer between React PWA and all backend services
- **Deployment:** AWS ECS (containers) or AWS Lambda
- **Runtime:** Node 22
- **Why it exists:** PWA Kit's SSR environment was too limited. The BFF was created to handle integrations that couldn't be coded in the React SSR layer.

---

## Shared Header Problem

The navigation header is configured in one spot (SFCC categories) but **rendered twice** - once in SiteGenesis and once in PWA. Changes to navigation (like the registry nav) must be implemented in both systems. This will be resolved when the SiteGenesis migration completes.

---

## Data Flow: Product Lifecycle

```
Island Pacific (ERP)          ← SKU created here (bare-bones data)
       │
       ▼
  Microservice (Aggregator)   ← Brings data into MongoDB
       │
       ├──► Oracle OMS         ← Order management
       ├──► CSC PIM            ← Product enrichment (images, descriptions, web attributes)
       ├──► Salesforce (SFCC)  ← Commerce platform (after PIM enrichment)
       ├──► Bloomreach         ← Search index
       └──► Channel Advisor    ← Amazon marketplace listings
```

Products start in Island Pacific, flow through an aggregator microservice to MongoDB, get enriched in the CSC PIM, and then are distributed to SFCC, Bloomreach, and other systems. The website only shows products after they've been enriched in PIM and enabled.

---

## Data Replication Model

SFCC has a strict staging-to-production replication model:

- **Code:** Must be deployed to Staging first, then promoted to Production. Cannot push directly to Production.
- **Data** (products, pricing, stores, catalog): Loaded to Staging, then replicated to Production.
- **Exception:** Inventory updates go directly to Production (real-time delta/intraday updates).

This creates operational challenges. For example, temporarily disabling a store for BOPIS requires changes in three places: eCom Tools, SFCC Staging, and SFCC Production (in the right order to avoid replication overwriting changes).

---

## Integration Categories

### Via BFF (slt-sfcc-bff modules)

| Category | Services |
|----------|----------|
| **Search** | Bloomreach (search, suggest, recommendations) |
| **Commerce** | SFCC SCAPI (products, baskets, orders, customers, categories) |
| **Payments** | SVS (gift cards) |
| **Delivery** | UPS (address validation, EDD), GoLocal (same-day delivery) |
| **Reviews** | TurnTo (ratings, reviews, Q&A) |
| **Location** | Radar (geolocation, store finder) |
| **SEO** | Rio SEO, Optiversal (content optimization) |
| **Email** | Emarsys (marketing, transactional) |
| **Fulfillment** | BOPIS (buy online, pick up in store) |
| **Registry** | Gift registry operations |

### Via SFCC Cartridges (slt-sfcc)

| Category | Services |
|----------|----------|
| **Payments** | Cybersource, PayPal, Afterpay, Apple Pay |
| **Tax** | Avatax |
| **OMS** | Oracle OMS export |
| **Warranty** | Extend |
| **Marketing** | Bloomreach, Emarsys |
| **Auth** | SLAS (Salesforce Login & Access Service) |

### Via AWS Lambda

| Category | Services |
|----------|----------|
| **Returns** | Narvar (return processing, webhooks) |
| **Claims** | Saphonia (shipping protection claims) |
| **Auto-processing** | Culinary class returns (digital items, auto-approved) |

### Via Client-Side (GTM)

| Category | Services |
|----------|----------|
| **Analytics** | Heap, Content Square |
| **Tracking** | Various marketing pixels |
| **Experiments** | AB testing |

---

## Deployment Architecture

```
┌─────────────────────────────────────────────────────────┐
│                     AWS Cloud                            │
│                                                         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐ │
│  │  ECS (BFF)   │  │   Lambda     │  │  CloudWatch  │ │
│  │  Port 1337   │  │  (Narvar,    │  │  (Logs,      │ │
│  │              │  │   Saphonia)  │  │   Metrics)   │ │
│  └──────────────┘  └──────────────┘  └──────────────┘ │
│                                                         │
│  ┌──────────────┐  ┌──────────────┐                    │
│  │  SSM Param   │  │  S3          │                    │
│  │  Store       │  │  (Files,     │                    │
│  │  (Secrets)   │  │   Imports)   │                    │
│  └──────────────┘  └──────────────┘                    │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│              Salesforce Commerce Cloud                    │
│                                                         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐ │
│  │  MRT (PWA)   │  │  SFCC Host   │  │  CDN         │ │
│  │  React SSR   │  │  (SiteGen)   │  │  (CloudFlare)│ │
│  └──────────────┘  └──────────────┘  └──────────────┘ │
│                                                         │
│  ┌──────────────┐  ┌──────────────┐                    │
│  │  Business    │  │  Log Center  │                    │
│  │  Manager     │  │  (SFCC Logs) │                    │
│  └──────────────┘  └──────────────┘                    │
└─────────────────────────────────────────────────────────┘
```

---

## Local Development Architecture

```
Browser → localhost:8000 (Express Proxy)
              │
              ├──► localhost:3000 (React PWA dev server)
              │         │
              │         └──► localhost:1337 (BFF - NestJS)
              │                    │
              │                    ├──► Bloomreach Staging API
              │                    ├──► SFCC Dev Sandbox
              │                    └──► Other third-party staging APIs
              │
              └──► SFCC Dev Sandbox (SiteGenesis pages)
                   development-na01-surlatable.demandware.net
```

| Port | Service | Start Command | Directory |
|------|---------|---------------|-----------|
| 8000 | Express Proxy | `npm run dev` | `slt-sfcc-react` |
| 3000 | React PWA Dev Server | (started by `npm run dev`) | `slt-sfcc-react` |
| 1337 | BFF (NestJS) | `npm run start:dev` | `slt-sfcc-bff` |

The BFF requires `SKIP_SSM=true` locally to bypass AWS Parameter Store and use `.env.develop` for configuration.

---

## Repositories

| Repo | Purpose | Tech | Deployment |
|------|---------|------|------------|
| `slt-sfcc` | SiteGenesis legacy frontend + SFCC cartridges | Rhinoscript, ISML, Node 6 | WebDAV to SFCC |
| `slt-sfcc-react` | React PWA modern frontend | React 18, TypeScript, Node 22 | Salesforce MRT |
| `slt-sfcc-bff` | Backend-for-Frontend API gateway | NestJS 11, TypeScript, Node 22 | AWS ECS / Lambda |
| `slt-sfcc-configs` | SFCC configuration (system objects, OCAPI, CDN rules) | XML, JSON | SFCC import/export |
| `slt-ecom-tools-v2` | Internal operational tools (UI) | React | - |
| `ecom-tools-api` | Internal operational tools (API) | Python | - |
| `narvar-integration` | Narvar returns + Saphonia claims | Lambda (Node.js) | AWS Lambda |

---

## Migration Roadmap

### Short Term (Now)
- Finish migrating culinary pages from SiteGenesis to PWA
- Complete home page migration
- Goal: 100% of customer-facing pages on React PWA

### Medium Term (~1 year)
- Begin migrating off Salesforce Commerce Cloud
- Route all commerce API calls through BFF (abstraction layer)
- Evaluate replacement platforms (BigCommerce, custom, etc.)
- Replace API layer while preserving React UI components

### Long Term (18-24 months)
- Complete SFCC migration before contract renewal
- Evaluate replacing expensive SaaS (Bloomreach, Narvar) with in-house AI-powered solutions
- Migrate off Island Pacific ERP

---

*Sources: Team huddle transcript (Feb 24, 2026), codebase analysis, existing architecture diagrams.*
