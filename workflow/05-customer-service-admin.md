# Customer Service & Admin Workflow

Internal workflow for customer service representatives and account administrators who support customers, manage accounts, and resolve issues.

---

## Roles

| Role | Responsibilities |
|------|-----------------|
| **Customer Service Rep (CSR)** | Customer inquiries, order issues, account support |
| **Account Admin** | Customer data management, password resets, account updates |
| **Registry Support** | Gift registry assistance |

---

## 1. Customer Lookup & Management

### Where It Happens

- **BFF Admin API** (`/admin/customers`) for programmatic access
- **SFCC Business Manager** > Merchant Tools > Customers

### Lookup Workflow

```
CSR receives customer inquiry
  │
  ├── Search by email:
  │     BFF Admin: GET /admin/customers?email=customer@example.com
  │     Searches DEFAULT_CUSTOMER_LIST_ID
  │
  ├── Search by customer ID:
  │     BFF Admin: GET /admin/customers/:customerId
  │
  └── SFCC BM: Customers > Customer Lists > Search
  │
  ▼
CSR views customer profile:
  ├── Name, email, phone
  ├── Addresses
  ├── Payment instruments
  ├── Order history
  ├── Wish lists
  ├── Gift registries
  └── Culinary class enrollments
```

### Account Update Workflow

```
CSR needs to update customer data
  │
  ├── BFF Admin: PATCH /admin/customers/:customerNo
  │     Update profile fields (name, email, phone, etc.)
  │
  └── SFCC BM: Direct customer record editing
```

---

## 2. Password & Authentication Support

### Where It Happens

- **BFF Customer API** (`/customers/`)
- **SFCC webhook** → **Emarsys email**

### Forgot Password Workflow

```
Customer requests password reset
  │
  ▼
React PWA: /account/forgot-password
  │
  ▼
BFF: POST /customers/forgot-password
  │  Sends request to SFCC
  │
  ▼
SFCC generates reset token
  │
  ▼
Webhook: POST /webhooks/reset-password-token
  │  (SLAS callback token validated)
  │
  ▼
Emarsys sends password reset email
  │
  ▼
Customer clicks link → /account/reset-password
  │
  ▼
BFF: POST /customers/reset-password
  │  Validates token and sets new password
```

### Password Change Workflow (Authenticated)

```
Customer changes password from account
  │
  ▼
BFF: POST /customers/:customerId/change-password
  │  Requires current password + new password
```

### Email Verification Workflow

```
CSR or system triggers email verification
  │
  ▼
BFF: POST /customers/:customerId/verify-email
  │  Sends verification request
  │
  ▼
Customer receives verification email
  │
  ▼
BFF: POST /customers/confirm-email
  │  Validates token and confirms email
```

---

## 3. Order Support

### Where It Happens

- **BFF Admin API** (`/admin/orders`)
- **BFF Customer Orders API** (`/customers/:id/orders`)
- **SFCC Business Manager** > Orders

### Order Inquiry Workflow

```
Customer contacts CS about order
  │
  ├── CSR looks up order:
  │     BFF Admin: GET /admin/orders/:orderNo
  │     Returns full order details
  │
  ├── View related sub-orders:
  │     BFF: GET /orders/:orderNo
  │     Returns primary + delivery + BOPIS + SDD orders
  │
  └── Guest order tracking:
        BFF: POST /customers/validate-track-order
        Validates order number + email or ZIP code
```

### Order Modification Workflow

```
CSR needs to update order
  │
  ├── BFF: PATCH /orders/:orderNo
  │     (Requires customer authorization)
  │
  └── SFCC BM: Direct order editing
        ├── Cancel items
        ├── Process appeasements
        ├── Update shipping
        └── Add notes
```

---

## 4. Gift Registry Support

### Where It Happens

- **BFF Registry API** (`/registry/`, `/customers/:id/gift-registries`)
- **React PWA** `/registry/` pages

### Registry Search (CSR or Customer)

```
CSR helps customer find a registry
  │
  ▼
BFF: GET /registry/search
  │  Search by:
  │  ├── Registrant name
  │  ├── Email address
  │  └── Event date range
  │
  ▼
Results returned (PII stripped for non-owners)
```

### Registry Management (CSR Assisting Customer)

```
CSR helps customer manage their registry
  │
  ├── View registries:
  │     GET /customers/:id/gift-registries
  │
  ├── View specific registry:
  │     GET /customers/:id/gift-registries/:registryId
  │
  ├── Update registry:
  │     PATCH /customers/:id/gift-registries/:registryId
  │
  ├── Add product to registry:
  │     POST /customers/:id/gift-registries/:registryId/items
  │
  ├── Update item quantity:
  │     PATCH /customers/:id/gift-registries/:registryId/items/:itemId
  │
  └── Remove product:
        DELETE /customers/:id/gift-registries/:registryId/items/:itemId
```

---

## 5. Gift Card Support

### Where It Happens

- **BFF Gift Card API** (`/gift-card/`, `/customers/:id/gift-cards`)
- **SVS** integration for balance checks

### Balance Check Workflow

```
Customer inquires about gift card balance
  │
  ▼
BFF: POST /gift-card/check-balance
  │  (reCAPTCHA Enterprise protected)
  │  Validates card number + PIN via SVS
  │
  ▼
Returns current balance
```

### Gift Card Wallet Management

```
CSR helps customer manage gift card wallet
  │
  ├── Add card:    POST /customers/:id/gift-cards
  ├── View cards:  GET /customers/:id/gift-cards
  ├── Remove card: DELETE /customers/:id/gift-cards/:paymentInstrumentId
  └── Refresh balance: POST /customers/:id/gift-cards/:id/refresh-balance
```

---

## 6. Wish List Support

### Where It Happens

- **BFF Wish Lists API** (`/customers/:id/wish-lists`)

### Workflow

```
CSR assists customer with wish list
  │
  ├── View default wish list:
  │     GET /customers/:id/wish-lists/default
  │
  ├── View all wish lists:
  │     GET /customers/:id/wish-lists
  │
  ├── Move items to cart:
  │     POST /customers/:id/wish-lists/:wlId/products/:prodId/add-to-basket/:basketId
  │     POST /customers/:id/wish-lists/:wlId/products/add-all-to-basket/:basketId
  │
  └── Share wish list (public link):
        GET /wish-lists/:wishlistId (no auth required)
```

---

## 7. Culinary Classes Support

### Where It Happens

- **BFF Customers API** (`/customers/:id/culinary-classes`)
- **SFCC BM** > bm_culinaryclassesreports

### CSR Workflow

```
Customer inquires about cooking classes
  │
  ├── View enrolled classes:
  │     BFF: GET /customers/:customerId/culinary-classes
  │     (Extracted from customer's order history)
  │
  └── View class roster (BM):
        BM > bm_culinaryclassesreports > Roster Search
        ├── Search by SKU
        ├── Print roster
        └── Export CSV
```

### Class Reporting Workflow

```
Operations reviews class fill rates
  │
  ▼
BM > bm_culinaryclassesreports > Fill Rate Search
  │
  ├── View class capacity vs. enrollment
  ├── Filter by date range, store, class type
  └── Generate fill rate reports
```

---

## Related Diagrams

- [Customer Service Flow](05-customer-service-admin.drawio)
- [Use Case Diagram](../../draw.io/02-use-case-diagram.drawio) (existing)

---

## Systems Involved

| System | Role |
|--------|------|
| **BFF Admin API** | Customer lookup, order lookup, account updates |
| **BFF Customer API** | Password management, email verification, lists |
| **SFCC Business Manager** | Direct customer/order editing, class reports |
| **SVS** | Gift card balance inquiries |
| **Emarsys** | Password reset emails, account verification emails |
| **React PWA** | Customer self-service (account pages) |
