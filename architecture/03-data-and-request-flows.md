# Sur La Table - Data & Request Flows

How data flows through the platform for key operations.

---

## 1. Customer Web Request Flow

How a page request travels through the system:

```
Customer Browser
       │
       ▼
Salesforce CDN (CloudFlare)
       │
       ├── URL matches PWA route? ──► React PWA (MRT, SSR with Express)
       │                                    │
       │                                    ├── Client renders page
       │                                    │
       │                                    └── API calls (client-side) ──► BFF (NestJS)
       │                                                                        │
       │                                                                        ├──► SFCC SCAPI
       │                                                                        ├──► Bloomreach
       │                                                                        ├──► TurnTo
       │                                                                        └──► Other services
       │
       └── URL matches SiteGenesis? ──► SiteGenesis (SFCC hosted)
                                              │
                                              └── Script API (direct to SFCC data)
```

**PWA Routes** (handled by React): PLP, PDP, cart, checkout, account, search, registry, stores
**SiteGenesis Routes** (legacy): culinary classes, home page (CMS), some legacy pages

---

## 2. Product Search Flow

When a customer searches for a product:

```
Customer types "knife" in search bar
       │
       ▼
React PWA (useSearch hook)
       │
       ▼
BFF: GET /bloomreach/product-search
     GET /bloomreach/search (auto-suggest)
       │
       ├──► Bloomreach Search API (hardgood results)
       ├──► Bloomreach Search API (culinary results)
       ├──► Bloomreach Search API (recipe results)
       └──► Bloomreach Suggest API (auto-complete)
       │
       ▼
BFF aggregates results, returns to React
       │
       ▼
React renders search results page
```

**Key env vars for local search:**
- `SLT_BR_PARAM_VALUE_AUTH_KEY` - Bloomreach API key
- `SLT_BR_PARAM_VALUE_ACCOUNT_ID` - Bloomreach account
- `SKIP_SSM=true` - Use local .env instead of AWS Parameter Store

---

## 3. Product Lifecycle (SKU Creation to Website)

How a product goes from creation to appearing on the website:

```
1. Island Pacific (ERP)
   └── SKU created with basic info (name, price, category)
         │
         ▼
2. Aggregator Microservice
   └── Pulls product data into MongoDB (central product store)
         │
         ├──► 3a. Oracle OMS (order management needs product data)
         ├──► 3b. CSC PIM (product enrichment)
         ├──► 3c. Salesforce Commerce Cloud (loaded to staging)
         ├──► 3d. Bloomreach (search index)
         └──► 3e. Channel Advisor (Amazon marketplace)
                │
                ▼
4. CSC PIM
   └── Business team enriches product:
       - Images, descriptions, dimensions
       - Web attributes, categories
       - "Enable for web" flag
         │
         ▼
5. Product appears on website
   └── SFCC staging → replicated to production
       Bloomreach index updated
       Search results include new product
```

**Important:** Products do NOT appear on the website until enriched and enabled in PIM. The PIM replaced an older system called Stibo that SLT used for 10+ years.

---

## 4. Order Flow

```
Customer places order on website (React PWA)
       │
       ▼
SFCC processes order
  ├── Payment authorized (Cybersource / PayPal / Afterpay / Apple Pay)
  ├── Tax calculated (Avatax)
  ├── Order created in SFCC
  └── Hooks fire (before/after, same DB transaction)
       │
       ▼
Order exported to Oracle OMS
  ├── Fulfillment routing (ship from store, warehouse, drop ship)
  ├── BOPIS / Same-day delivery (GoLocal) if applicable
  └── Shipping via UPS / FedEx
       │
       ▼
Post-purchase
  ├── Confirmation email (Emarsys → Braze)
  ├── Tracking notification (Narvar)
  ├── Warranty registration (Extend) if applicable
  └── Analytics events (Heap, GTM pixels)
```

---

## 5. Return Flow

```
Customer initiates return
       │
       ▼
Narvar customer portal
  └── Customer selects items, reason
       │
       ▼
Narvar webhook → AWS Lambda (narvar-integration)
       │
       ├── Is it a culinary class? ──► Auto-process (digital item, no physical return)
       │                               BUT: blocked if within 48 hours of class date
       │
       └── Is it a hard good? ──► Standard return flow
              │
              ▼
       Lambda calls Oracle OMS
         └── Creates return authorization
              │
              ▼
       eCom Tools V2 (operational dashboard)
         └── Operations team can view/manage returns
```

**Saphonia (shipping protection)** follows a similar flow but for damage/loss claims instead of returns. Both Narvar and Saphonia integrations live in the same Lambda monorepo (`narvar-integration`).

---

## 6. SFCC Data Replication Flow

```
                    STAGING                              PRODUCTION
              ┌─────────────────┐                  ┌─────────────────┐
              │                 │    Replication    │                 │
  Code ──────►│  Code deployed  │─────────────────►│  Code promoted  │
              │  here first     │                  │  from staging   │
              │                 │                  │                 │
  Data ──────►│  Products       │─────────────────►│  Products       │
  (catalog,   │  Pricing        │  Replicated      │  Pricing        │
   pricing,   │  Store data     │                  │  Store data     │
   stores)    │  Categories     │                  │  Categories     │
              └─────────────────┘                  └─────────────────┘
                                                          ▲
                                                          │
                                              Inventory ──┘
                                              (direct to prod,
                                               real-time delta)
```

**Rules:**
- Code: Staging → Production only. Cannot push directly to prod.
- Catalog/pricing/stores: Loaded to staging, replicated to prod.
- Inventory: Direct to production (real-time updates, intraday deltas).
- Temporary store changes (e.g., weather closures): Must update in eCom Tools + SFCC Staging + SFCC Prod (in correct order to avoid replication overwrite).

---

## 7. Local Development Request Flow

```
Browser
  │
  ▼
localhost:8000 (Express Proxy - proxy/server.ts)
  │
  ├── Matches PWA_ROUTES? ──► localhost:3000 (React dev server)
  │                                 │
  │                                 └── Client API calls (relative URLs)
  │                                     go back through :8000 proxy
  │                                     └──► localhost:1337 (BFF)
  │                                               │
  │                                               ├──► Bloomreach staging API
  │                                               ├──► SFCC dev sandbox
  │                                               └──► Other staging APIs
  │
  └── Does NOT match? ──► SFCC dev sandbox
                          (development-na01-surlatable.demandware.net)
```

**Start commands:**
```bash
# Terminal 1: React PWA + Proxy
cd slt-sfcc-react && npm run dev    # proxy :8000, react :3000

# Terminal 2: BFF
cd slt-sfcc-bff && npm run start:dev  # BFF :1337 (sets SKIP_SSM=true)
```

---

## 8. GTM / Analytics Data Flow

```
Customer interacts with page
       │
       ▼
React fires dataLayer events
       │
       ▼
GTM Client-Side Container
  ├── Heap (product analytics, funnels)
  ├── Content Square (session recording)
  ├── Marketing pixels (various)
  └── Forwards to GTM Server-Side ──► Hosted on AWS
                                        ├── Yelp pixel
                                        └── Other server-side tags
```

**GTM containers:**
- Client-side: One container with prod + staging/dev (not ideal, but current setup)
- Server-side: Separate prod and non-prod environments
- Near capacity: must clean up old tags when adding new ones
- Diego is the primary maintainer

---

*Sources: Team huddle transcript (Feb 24, 2026), codebase analysis of slt-sfcc-bff and slt-sfcc-react.*
