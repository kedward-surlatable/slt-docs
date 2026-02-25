# Content Management Workflow

Internal workflow for content managers and marketing teams maintaining site content, landing pages, SEO, and editorial experiences on the Sur La Table platform.

---

## Roles

| Role | Responsibilities |
|------|-----------------|
| **Content Manager** | Content assets, slots, landing pages |
| **SEO Specialist** | URL mappings, redirects, sitemaps, metadata |
| **Recipe Editor** | Recipe content, images, videos |
| **Marketing Manager** | Promotional banners, campaign landing pages |

---

## 1. Content Asset Management

### Where It Happens

- **SFCC Business Manager** > Merchant Tools > Content > Content Library
- **BFF Content API** (`/content/asset/:id`) for delivery

### Workflow

```
Content Manager creates/edits content in SFCC BM
  │
  ├── Create content asset (HTML body, metadata)
  ├── Set online/offline scheduling
  ├── Assign to content folders
  └── Configure multilingual versions
  │
  ▼
BFF Content module serves content
  │  GET /content/asset/:contentAssetId
  │
  ▼
React PWA renders content on storefront
```

---

## 2. Content Slot Management

Content slots are configurable regions on the storefront that display dynamic content based on context.

### Where It Happens

- **SFCC Business Manager** > Merchant Tools > Content > Slots
- **BFF Content API** for slot delivery

### Slot Types & Endpoints

| Slot Type | BFF Endpoint | Use Case |
|-----------|-------------|----------|
| **Global** | `GET /content/slot/:id/global` | Site-wide banners, headers, footers |
| **Category** | `GET /content/slot/:id/category/:catId` | Category-specific promotional content |
| **Product** | `GET /content/slot/:id/product/:prodId` | Product page custom content blocks |
| **Folder** | `GET /content/slot/:id/folder/:folderId` | Content folder-based slot display |

### Workflow

```
Marketing Manager configures content slots in SFCC BM
  │
  ├── Select slot location (global, category, product, folder)
  ├── Assign content assets or HTML to slot
  ├── Set scheduling rules (date range, A/B testing)
  ├── Configure slot rendering template
  └── Preview with effective date
  │
  ▼
BFF serves slot content based on context
  │
  ├── Dynamic configuration loading based on category template
  ├── Category online status validation
  └── Recipe category detection and special handling
  │
  ▼
React PWA renders slots in appropriate page regions
```

---

## 3. URL Management & Redirects

### Where It Happens

- **SFCC Business Manager** > Merchant Tools > Search > URL Redirects
- **Rio SEO** integration for URL mappings
- **BFF Content module** for URL resolution

### Workflow

```
SEO Specialist manages URL mappings
  │
  ├── SFCC BM: Configure URL redirects (301/302)
  ├── Rio SEO: Manage SEO URL mappings
  └── BFF resolves URLs at runtime:
        GET /content/resolve/*path
        GET /content/url-resolution/*path
  │
  ▼
BFF URL Resolution Logic:
  │
  ├── Check for category match → validate online status
  ├── Check for content asset match
  ├── Check for redirect rules (301/302)
  ├── Recipe category detection and routing
  └── Return appropriate response or redirect
```

---

## 4. Recipe Content Management

### Where It Happens

- **SFCC Business Manager** (recipe products)
- **BFF Recipes API** (`/recipes/`)

### Workflow

```
Recipe Editor creates recipe in SFCC
  │
  ├── Create recipe product with type=RECIPE
  ├── Add ingredients, instructions, images, videos
  ├── Set cooking times, difficulty, cuisine type
  └── Link to related products (cookware, tools)
  │
  ▼
BFF Recipes module enriches on delivery
  │  GET /recipes/:id
  │
  ├── Generate structured data schemas (JSON-LD)
  ├── Pull video metadata
  ├── Generate Q&A schemas from TurnTo
  ├── Fetch recipe recommendations
  └── Check related product inventory
  │
  ▼
React PWA renders recipe pages at /recipes/:id
```

---

## 5. SEO & Sitemap Management

### Where It Happens

- **SFCC Business Manager** > Search > Indexes
- **Rio SEO** integration (`int_seo`)
- **PWA scheduled jobs** (`int_pwa`)

### Sitemap Workflow

```
Scheduled Job: PWA Sitemap Import (hourly)
  │  (int_pwa > pwa-rio-jobs.xml)
  │
  ├── Fetch sitemap data from storefront URLs
  ├── Import into SFCC for search engine indexing
  └── Update PWA cache categories
  │
  ▼
Sitemaps served to search engines
```

### SEO Configuration

```
SEO Specialist configures metadata
  │
  ├── SFCC BM: Product/category meta titles, descriptions
  ├── Rio SEO: URL mappings and canonical URLs
  ├── Google Tag Manager: Analytics data layer
  │     (int_googletagmanager > GTM data layer scripts)
  └── Bloomreach: Search engine optimization settings
```

---

## 6. Content Optimization (Optiversal)

### Where It Happens

- **Optiversal** integration (`int_optiversalPLP`)
- **BFF** related categories endpoint

### Workflow

```
Optiversal scheduled job runs
  │  (int_optiversalPLP > optiversalPages.js)
  │
  ├── Generate optimized product listing content
  └── Create related category recommendations
  │
  ▼
BFF serves optimized content
  │  GET /products/:id/related-categories
  │
  ▼
React PWA displays related categories and optimized content
```

---

## 7. Culinary Classes Content

### Where It Happens

- **SFCC Business Manager** (class products)
- **BFF Stores API** for class schedules
- **BFF Customers API** for enrolled classes

### Workflow

```
Content Manager creates class in SFCC
  │
  ├── Create product with type=CULINARY
  ├── Set class details (date, time, store, instructor)
  ├── Configure enrollment SKU
  └── Assign to stores
  │
  ▼
BFF serves class data
  ├── GET /stores/:storeId/cooking-classes
  └── GET /customers/:customerId/culinary-classes
  │
  ▼
React PWA renders class pages at /cooking-classes/:id
```

---

## Related Diagrams

- [Content Management Flow](03-content-management.drawio)
- [React Feature Page Architecture](../../draw.io/08-react-feature-page-architecture.drawio) (existing)

---

## Systems Involved

| System | Role |
|--------|------|
| **SFCC Business Manager** | Content assets, slots, URLs, redirects, recipes |
| **BFF Content API** | Content delivery, URL resolution, recipe enrichment |
| **Rio SEO** | URL mappings, sitemaps, canonical URLs |
| **Optiversal** | Content optimization, related categories |
| **Google Tag Manager** | Analytics data layer for content tracking |
| **TurnTo** | Recipe Q&A and review content |
| **Emarsys** | Content-triggered email campaigns |
