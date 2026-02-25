# Product & Catalog Management Workflow

Internal workflow for merchandisers and product managers who maintain the Sur La Table product catalog, categories, and search experience.

---

## Roles

| Role | Responsibilities |
|------|-----------------|
| **Merchandiser** | Product data, pricing, categories, promotions |
| **Search Manager** | Bloomreach search configuration, synonyms, redirects |
| **Content Specialist** | Product descriptions, images, recipes |

---

## 1. Product Data Management

### Where It Happens

- **SFCC Business Manager** > Merchant Tools > Products and Catalogs
- **BFF Admin API** (`/admin/products`) for programmatic access

### Workflow

```
Merchandiser updates product in SFCC Business Manager
  │
  ├── Edit product attributes (name, description, price, images)
  ├── Set online/offline status (c_online flag)
  ├── Assign to categories
  ├── Configure variants (size, color, etc.)
  └── Set custom attributes (Apple Pay eligibility thresholds, etc.)
  │
  ▼
BFF enriches product data on read
  ├── Generates PWA URL based on product type:
  │     Standard  → /product/{id}
  │     Recipe    → /recipes/{id}
  │     Culinary  → /cooking-classes/{id}
  ├── Checks BOPIS / Same-Day Delivery eligibility
  ├── Pulls TurnTo reviews and Q&A schemas
  ├── Calculates Apple Pay eligibility
  └── Caches enriched data (12-hour TTL)
  │
  ▼
Product appears on React PWA storefront
```

### Key Operations

| Operation | Tool | Endpoint / Location |
|-----------|------|-------------------|
| View/edit product | SFCC BM | Products and Catalogs > Products |
| Bulk product lookup | BFF Admin API | `GET /admin/products?ids=...` |
| Search products | BFF Admin API | `GET /admin/products/search?query=...` |
| Check online status | BFF Admin API | Returns `c_online` flag |
| Set product offline | SFCC BM | Product > General > Online Status |

---

## 2. Category Management

### Where It Happens

- **SFCC Business Manager** > Merchant Tools > Products and Catalogs > Categories
- **SFCC BM** > SLT Settings > Dynamic Category Assignment

### Standard Category Workflow

```
Merchandiser manages category hierarchy in SFCC BM
  │
  ├── Create / edit / reorder categories
  ├── Assign products to categories
  ├── Set category online/offline status
  ├── Configure category templates
  └── Set effective date for scheduled changes
  │
  ▼
BFF categories endpoint serves hierarchy
  ├── Supports depth filtering (top-level vs. full tree)
  ├── 12-hour cache TTL
  └── Preview mode via x-slt-effective-date-time header (bypasses cache)
  │
  ▼
React PWA renders navigation and PLPs
```

### Dynamic Category Assignment Workflow

Automated product-to-category assignment based on configurable business rules.

```
Admin configures rules in SFCC BM
  │  (BM > SLT Settings > Dynamic Category Assignment)
  │
  ├── Brand Category Expansion (primary & secondary)
  │     Config: topLevelCategoryID, categoryIDToExclude, categoryDepthLimit
  │
  ├── Top-Rated Category
  │     Auto-assigns highly-rated products
  │
  ├── Sales & Clearance Category
  │     Auto-assigns products on sale
  │
  └── Culinary Classes Category
        Auto-assigns cooking class products
  │
  ▼
Rules stored in DynamicCategorizationRules custom objects
  │
  ▼
Scheduled job runs dynamic categorization
  │
  ▼
Products auto-assigned to matching categories
```

---

## 3. Search & Merchandising (Bloomreach)

### Where It Happens

- **Bloomreach Dashboard** (external)
- **BFF Bloomreach module** (`/bloomreach/`) for API access
- **SFCC BM** for search index configuration

### Workflow

```
Search Manager configures in Bloomreach Dashboard
  │
  ├── Configure search rules, synonyms, redirects
  ├── Set up product recommendations
  ├── Configure search widgets
  ├── Manage facets and filters:
  │     brand, color, size, material, cuisine type,
  │     category, price range, rating, savings
  └── Set product type filtering (RECIPE, CULINARY, TOOL, COOKWARE, etc.)
  │
  ▼
BFF Bloomreach module serves search results
  │  Endpoints:
  │  ├── GET /bloomreach/search          (full-text search + suggestions)
  │  ├── GET /bloomreach/product-search  (advanced product search)
  │  ├── GET /bloomreach/widgets         (recommendation widgets)
  │  └── GET /bloomreach/recommendations (product recommendations)
  │
  ├── Store-specific filtering (BOPIS / SDD eligible)
  ├── Effective date support for preview
  ├── Category tree retrieval
  └── Telemetry: CloudWatch Logs → Lambda → Aurora
  │
  ▼
React PWA renders search results, PLPs, recommendations
```

---

## 4. Product Feed Generation

### Where It Happens

- **SFCC scheduled jobs** (`int_custom_feeds`)
- **int_extend** for warranty product sync

### Workflow

```
Scheduled Job: Custom Feed Generation
  │
  ├── Extract product data from SFCC catalog
  ├── Generate feed files (XML/JSON)
  └── Push to external systems
  │
  ▼
Extend Product Export (daily job)
  │
  ├── Extract warrantable products
  ├── Send to Extend warranty platform
  └── Track sync status
```

---

## 5. Pricing & Promotions

### Where It Happens

- **SFCC Business Manager** > Online Marketing > Promotions
- **SFCC Business Manager** > Products and Catalogs > Price Books

### Workflow

```
Merchandiser creates/edits promotions
  │
  ├── Configure promotion rules and qualifiers
  ├── Set date ranges and scheduling
  ├── Assign coupon codes
  └── Test with preview mode (x-slt-effective-date-time)
  │
  ▼
BFF applies promotions via SFCC Commerce SDK
  │
  ├── Basket-level promotion calculation
  ├── Coupon validation (with reCAPTCHA protection)
  └── Returns promotion details to React frontend
```

---

## Related Diagrams

- [Product Catalog Management Flow](01-product-catalog-management.drawio)
- [BFF Service Integration Categories](../../draw.io/07-bff-service-integration-categories.drawio) (existing)

---

## Systems Involved

| System | Role |
|--------|------|
| **SFCC Business Manager** | Product data entry, categories, pricing, promotions |
| **BFF Admin API** | Programmatic product lookup and search |
| **Bloomreach** | Search engine, recommendations, merchandising rules |
| **Extend** | Warranty product sync |
| **TurnTo** | Reviews and Q&A enrichment |
| **Optiversal** | Related category recommendations |
