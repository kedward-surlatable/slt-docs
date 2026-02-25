# Integration Monitoring & Scheduled Jobs Workflow

Internal workflow for platform engineers and operations staff who monitor system integrations, manage scheduled jobs, and maintain the health of the 47+ service integrations powering the platform.

---

## Roles

| Role | Responsibilities |
|------|-----------------|
| **Platform Engineer** | Integration health, service configuration, troubleshooting |
| **Operations Manager** | Job scheduling, monitoring, failure response |
| **DevOps Engineer** | Infrastructure health, caching, logging |

---

## 1. Integration Workflow Management

### Where It Happens

- **SFCC Business Manager** > bm_integrationframework
- **SFCC BM** > Administration > Operations > Job Schedules

### Workflow Schedule Management

```
Platform Engineer configures integration workflows
  │
  ▼
BM > Operations > Workflow Schedules
  │  (bm_integrationframework)
  │
  ├── Define workflow schedule definitions
  │     ├── Workflow name and description
  │     ├── Execution frequency (cron-like)
  │     ├── Component relationships
  │     └── Enable/disable toggle
  │
  ├── Edit workflow component relationships
  │     Define which components run in which order
  │
  └── Download workflow log files
        For debugging and audit trails
```

### Workflow Monitoring

```
Operations monitors running workflows
  │
  ▼
BM > Operations > Workflow Plan
  │  (bm_integrationframework)
  │
  ├── View all running workflows
  ├── Check execution status
  ├── View execution plans
  └── Identify failed or stalled workflows
```

---

## 2. Scheduled Jobs Overview

### All Platform Jobs

| Job | Cartridge | Frequency | Purpose |
|-----|-----------|-----------|---------|
| **Order Export** | int_oroms | Daily | Export orders to Oracle OMS |
| **Stale Order Cleanup** | app_surlatable | Daily 13:00 UTC | Mark old stuck orders as failed |
| **Extend Contracts Creation** | int_extend | Recurring | Create warranty contracts |
| **Extend Product Export** | int_extend | Daily | Sync products to Extend |
| **Extend Refunds** | int_extend | Recurring | Process warranty refunds |
| **Extend Historical Orders** | int_extend | One-time | Backfill legacy orders |
| **Extend Orders Creation** | int_extend | Recurring | Create Extend orders |
| **PWA Sitemap Import** | int_pwa | Hourly | Import sitemaps for SEO |
| **PWA Cache Update** | int_pwa | Recurring | Refresh PWA cache categories |
| **Optiversal Page Optimization** | int_optiversalPLP | Recurring | Generate optimized PLP content |
| **Custom Feed Generation** | int_custom_feeds | Recurring | Generate product data feeds |
| **Dynamic Categorization** | int_custom_feeds | Recurring | Auto-assign product categories |
| **Import Logging** | int_custom_feeds | Recurring | Log data import results |

### Job Monitoring Workflow

```
Operations monitors scheduled jobs
  │
  ▼
BM > Administration > Operations > Job Schedules
  │
  ├── View all configured jobs
  ├── Check last run status (Success / Failed / Running)
  ├── View execution history
  ├── Manually trigger jobs (for testing or catch-up)
  └── Enable/disable jobs
  │
  ▼
BM > Administration > Operations > Job Monitor
  │
  ├── View currently running jobs
  ├── Check execution duration
  └── Cancel stuck jobs
```

---

## 3. BFF Health & Admin Monitoring

### Where It Happens

- **BFF Health endpoint** (`/health`)
- **BFF Admin endpoints** (`/admin/`)
- **Swagger/OpenAPI** documentation (`/api-doc`)

### Health Check Workflow

```
DevOps monitors BFF service health
  │
  ▼
GET /health
  │
  ├── Returns service status
  ├── Checks core dependencies
  └── Used by load balancers and monitoring
  │
  ▼
Automated alerts on failure
```

### API Documentation

```
Platform Engineer reviews available APIs
  │
  ▼
GET /api-doc (Swagger UI)
  │  (requires ACTIVATE_SWAGGER=true)
  │
  ├── Browse all BFF endpoints
  ├── View request/response schemas
  ├── Test endpoints interactively
  └── Review authentication requirements
```

---

## 4. Caching Strategy Monitoring

### Where It Happens

- **BFF caching layer** (Redis cluster or in-memory)
- **CDN** (CloudFlare)
- **BFF bridge configuration** (`/bridge/`)

### Cache Tiers

| Layer | TTL | Scope | Content |
|-------|-----|-------|---------|
| **CDN** | Varies | Global | Static assets, page responses |
| **Redis Cluster** | 1 min - 12 hrs | BFF | Products, categories, config |
| **In-Memory** | 500 keys max | BFF Instance | Frequently accessed data |
| **BOPIS Inventory** | 1 min | BFF | Store inventory levels |
| **Product Schemas** | 12 hrs | BFF | JSON-LD structured data |
| **Categories** | 12 hrs | BFF | Category tree hierarchy |
| **Bridge Config** | 12 hrs | BFF | Server configuration |

### Cache Bypass for Preview

```
Merchandiser previews scheduled content changes
  │
  ▼
React PWA sends request with header:
  x-slt-effective-date-time: 2025-03-15T00:00:00Z
  │
  ▼
BFF bypasses cache:
  ├── Categories served fresh (no cache)
  ├── Bloomreach search uses effective date
  └── Content slots reflect future state
  │
  ▼
Merchandiser sees future content as it will appear
```

---

## 5. Third-Party Service Integration Map

### Integration Categories

```
BFF Integration Modules (47+)
  │
  ├── COMMERCE PLATFORM
  │     └── Salesforce Commerce Cloud (Commerce SDK, SCAPI, Admin API)
  │
  ├── SEARCH & RECOMMENDATIONS
  │     ├── Bloomreach (search, suggest, recommendations, widgets)
  │     └── Optiversal (content optimization, related categories)
  │
  ├── PAYMENTS
  │     ├── Cybersource (credit card processing)
  │     ├── PayPal (express checkout)
  │     ├── Afterpay (BNPL)
  │     ├── Apple Pay (mobile payments)
  │     └── SVS (gift card balance/redemption)
  │
  ├── EMAIL & MARKETING
  │     ├── Emarsys (transactional + campaign emails)
  │     └── Google Tag Manager (analytics)
  │
  ├── REVIEWS & UGC
  │     └── TurnTo (reviews, Q&A, checkout comments)
  │
  ├── SHIPPING & DELIVERY
  │     ├── UPS (address validation, shipping)
  │     └── FedEx (delivery estimation)
  │
  ├── FULFILLMENT
  │     ├── BOPIS (buy online pick in store)
  │     ├── GoLocal (same-day delivery)
  │     └── Oracle OMS (order management system)
  │
  ├── LOCATION SERVICES
  │     └── Radar (geocoding, proximity search)
  │
  ├── SEO
  │     └── Rio SEO (URL mappings, sitemaps)
  │
  ├── SECURITY
  │     └── Google reCAPTCHA Enterprise (bot protection)
  │
  └── WARRANTY
        └── Extend (product warranties, contracts)
```

### Integration Failure Handling

```
Third-party service call fails
  │
  ├── BFF retry logic (Axios with retry):
  │     ├── Configurable retry count
  │     ├── Exponential backoff
  │     └── Circuit breaker patterns
  │
  ├── Graceful degradation:
  │     ├── Reviews unavailable → page renders without reviews
  │     ├── Search down → fallback behavior
  │     ├── Delivery estimate fails → hide estimate
  │     └── Offline products → admin API fallback
  │
  ├── Error classification:
  │     ├── SLTBadRequestException (400)
  │     ├── SLTNotFoundException (404)
  │     ├── SLTGoneException (410)
  │     └── Commerce SDK exception wrapping
  │
  └── Logging & observability:
        ├── Pino structured logging
        ├── CloudWatch Logs (search telemetry)
        └── Request tracking with user context
```

---

## 6. BM Admin Tools

### Where It Happens

- **SFCC Business Manager** > bm_tools (Tool Box)

### Available Tools

```
BM > Tool Box
  │
  ├── Total File Demander
  │     Browse local and remote file systems
  │     Upload/download files
  │     Useful for checking deployed code
  │
  ├── Browse Remote - HTTP
  │     Execute HTTP GET requests
  │     View raw responses
  │     Test external API connectivity
  │
  ├── BM Session Keep Alive
  │     Prevent session timeout during long operations
  │
  ├── Terminal
  │     Execute server-side JavaScript expressions
  │     Access to all SFCC APIs (dw.*)
  │     Useful for debugging and data inspection
  │
  └── Mapping Manager
        View/export/import key-value configuration maps
        File storage: /impex/src/toolbox/mappingmgr/
```

---

## 7. Service Configuration (SFCC)

### Service Definitions

```
Platform Engineer manages external service configs
  │
  ▼
BM > Administration > Operations > Services
  │
  ├── SVS (Gift Cards):
  │     SOAP service → webservices-cert.storedvalue.com
  │     Timeout: 5s, Circuit breaker: 5 calls/20s
  │
  ├── UPS (Shipping):
  │     REST service with OAuth
  │     Test + Production endpoints
  │
  ├── Extend (Warranties):
  │     HTTP service → demo API
  │     Timeout: 20s
  │
  ├── OROMS (Oracle OMS):
  │     Order export service
  │
  └── SLAS (Authentication):
        Imported via sfcc-slas-services.xml
```

---

## Related Diagrams

- [Integration Monitoring & Jobs Flow](08-integration-monitoring-jobs.drawio)
- [BFF Service Integration Categories](../../draw.io/07-bff-service-integration-categories.drawio) (existing)
- [Caching Strategy - Multi-layer](../../draw.io/10-caching-strategy-multi-layer.drawio) (existing)

---

## Systems Involved

| System | Role |
|--------|------|
| **SFCC Business Manager** | Job scheduling, workflow management, service config, tools |
| **BFF Health/Admin API** | Service health, API documentation |
| **Redis** | BFF caching layer |
| **CloudFlare CDN** | Edge caching |
| **AWS CloudWatch** | Logging and monitoring |
| **AWS Lambda** | Search telemetry processing |
| **47+ Third-Party Services** | Various integrations (see map above) |
