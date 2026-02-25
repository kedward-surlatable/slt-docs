# Inventory & Store Operations Workflow

Internal workflow for store operations managers, inventory coordinators, and fulfillment teams managing in-store pickup (BOPIS), same-day delivery, store inventory, and location-based services.

---

## Roles

| Role | Responsibilities |
|------|-----------------|
| **Store Operations Manager** | Store configuration, hours, capacity |
| **Inventory Coordinator** | Stock levels, availability, allocation |
| **Fulfillment Associate** | BOPIS pickup, same-day delivery dispatch |
| **Culinary Program Manager** | Cooking class scheduling, roster management |

---

## 1. BOPIS (Buy Online, Pick Up In Store)

### Where It Happens

- **BFF BOPIS module** (`/bopis/`)
- **BFF Baskets API** (inventory checks during checkout)
- **BFF Stores API** (`/stores/:storeId/inventory/bopis`)
- **SFCC** bridge preferences (feature toggle)

### BOPIS Availability Check Workflow

```
Customer selects BOPIS during shopping
  │
  ▼
BFF checks BOPIS eligibility
  │  (bopis.isEnabled() → reads bridge preferences)
  │
  ▼
BFF inventory check:
  GET /stores/:storeId/inventory/bopis?productIds=...&quantity=...
  │
  ├── Query SFCC inventory for specific store
  ├── Compare requested quantity vs. available
  └── Cache results (1-minute TTL)
  │
  ▼
Results returned to storefront:
  ├── Available → Show "Pick Up Today" option
  ├── Limited   → Show "Limited Stock" warning
  └── Unavailable → Hide BOPIS option for this store
```

### Bulk Inventory Check (Checkout)

```
Customer proceeds to checkout with BOPIS items
  │
  ▼
BFF: GET /baskets/:basketId/inventory
  │  Filters by BOPIS stores
  │
  ├── Check all basket items at selected store
  ├── Validate quantities available
  └── Return aggregated availability
  │
  ▼
Checkout flow:
  ├── All available → proceed to payment
  ├── Partial → prompt customer to adjust
  └── Unavailable → suggest alternatives
```

### BOPIS Order Fulfillment

```
BOPIS order placed
  │
  ▼
BOPIS sub-order created (c_bopisOrderNo)
  │
  ▼
Store receives pickup notification
  │
  ├── Pick items from store inventory
  ├── Stage for customer pickup
  └── Update order status
  │
  ▼
Customer picks up at store
  │
  ▼
Order marked as fulfilled
```

---

## 2. Same-Day Delivery (SDD)

### Where It Happens

- **BFF GoLocal module** (`/go-local/`)
- **BFF Stores API** (`/stores/inventory/same-day-delivery`)
- **GoLocal** integration for delivery logistics

### SDD Availability Workflow

```
Customer enters delivery address
  │
  ▼
BFF checks SDD eligibility by location:
  GET /stores/inventory/same-day-delivery?lat=...&lng=...&productIds=...
  │
  ├── Find eligible stores near customer
  ├── Check product inventory at nearby stores
  └── Validate delivery zone coverage
  │
  ▼
Results returned:
  ├── Available → Show "Same-Day Delivery" option
  └── Unavailable → Fall back to standard shipping
```

### SDD Order Fulfillment

```
SDD order placed
  │
  ▼
SDD sub-order created (c_sameDayDeliveryOrderNo)
  │
  ▼
Fulfilling store dispatches delivery
  │  (GoLocal delivery logistics)
  │
  ▼
Customer receives same-day delivery
```

---

## 3. Store Inventory Management

### Where It Happens

- **BFF Stores API** (`/stores/`)
- **SFCC Business Manager** > Merchant Tools > Products > Inventory

### Store Search Workflow

```
Operations needs to check store inventory
  │
  ├── Search by location:
  │     BFF: GET /stores/search/nearest?lat=...&lng=...
  │
  ├── Search with inventory filter:
  │     BFF: POST /stores/search
  │     Body: { productIds, quantities, location }
  │
  └── Get specific store:
        BFF: GET /stores/:storeId
        Returns: store details, hours, location
```

### Product Availability Check

```
BFF: GET /baskets/:basketId/availability
  │
  ├── Check standard inventory
  ├── Check BOPIS availability at relevant stores
  ├── Check SDD availability at nearby stores
  └── Return aggregated availability per item
```

---

## 4. Store Location & Configuration

### Where It Happens

- **SFCC Business Manager** > Merchant Tools > Stores
- **Radar** integration for geolocation
- **BFF Stores API**

### Store Data Management

```
Store Operations configures store details in SFCC BM
  │
  ├── Store name, address, phone
  ├── Operating hours (regular + holiday)
  ├── Store capabilities:
  │     ├── BOPIS enabled/disabled
  │     ├── SDD enabled/disabled
  │     ├── Cooking classes offered
  │     └── Store events
  ├── Geographic coordinates (lat/lng)
  └── Inventory allocation settings
  │
  ▼
BFF serves store data to React PWA
  │
  ├── Store locator page
  ├── Store detail pages
  ├── Checkout store selection
  └── Cooking class listings
```

### Location Services (Radar)

```
Customer uses store locator or delivery check
  │
  ▼
BFF uses Radar integration:
  │
  ├── Geocode customer address
  ├── Calculate distances to stores
  ├── Determine delivery zones
  └── Return sorted store results
```

---

## 5. Cooking Class Operations

### Where It Happens

- **SFCC Business Manager** > bm_culinaryclassesreports
- **BFF Stores API** (`/stores/:storeId/cooking-classes`)
- **BFF Customers API** (`/customers/:id/culinary-classes`)

### Class Roster Management

```
Store Operations manages class enrollment
  │
  ▼
BM > bm_culinaryclassesreports > Roster Search
  │
  ├── Search by class SKU
  ├── View enrolled customer list:
  │     ├── Customer name
  │     ├── Contact info
  │     ├── Enrollment date
  │     └── Payment status
  ├── Print roster for class day
  └── Export roster to CSV
```

### Fill Rate Reporting

```
Culinary Program Manager reviews class performance
  │
  ▼
BM > bm_culinaryclassesreports > Fill Rate Search
  │
  ├── View capacity utilization per class
  ├── Filter by:
  │     ├── Date range
  │     ├── Store location
  │     └── Class type
  ├── Identify under/over-performing classes
  └── Generate fill rate reports for planning
```

### Class Schedule Viewing

```
BFF: GET /stores/:storeId/cooking-classes
  │
  ├── Returns upcoming classes for store
  ├── Includes: date, time, title, capacity, availability
  └── Displayed on store detail page and class browsing
```

---

## 6. Delivery Estimation & Address Validation

### Where It Happens

- **BFF Delivery module** (`/delivery/`)
- **UPS / FedEx** integrations

### Delivery Date Estimation

```
Customer enters shipping ZIP code
  │
  ▼
BFF: GET /delivery/estimated-delivery-date?zip=12345
  │
  ├── Query FedEx for delivery estimate
  └── Return estimated delivery date
  │
  ▼
Displayed on product pages and checkout
```

### Address Validation

```
Customer enters shipping address at checkout
  │
  ▼
BFF: POST /delivery/validate-address
  │
  ├── Validate via UPS address API
  ├── Return validated/corrected address
  └── Flag any issues (incomplete, invalid)
  │
  ▼
Customer confirms or corrects address
```

---

## Related Diagrams

- [Inventory & Store Operations Flow](07-inventory-store-operations.drawio)

---

## Systems Involved

| System | Role |
|--------|------|
| **SFCC Business Manager** | Store config, inventory, class management |
| **BFF BOPIS Module** | BOPIS inventory checks and eligibility |
| **BFF GoLocal Module** | Same-day delivery logistics |
| **BFF Stores API** | Store search, details, cooking classes |
| **BFF Delivery API** | Delivery estimation, address validation |
| **Radar** | Geolocation, distance calculations |
| **UPS** | Address validation, shipping estimates |
| **FedEx** | Delivery date estimation |
| **GoLocal** | Same-day delivery logistics |
