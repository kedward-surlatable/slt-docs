# Order Management & Fulfillment Workflow

Internal workflow for operations staff, customer service, and fulfillment teams managing the order lifecycle from placement through export and delivery.

---

## Roles

| Role | Responsibilities |
|------|-----------------|
| **Operations Manager** | Order export monitoring, job health, reconciliation |
| **Customer Service Rep** | Order lookup, status updates, appeasements |
| **Fulfillment Team** | BOPIS pickup, same-day delivery, shipping |
| **Finance** | PayPal transaction review, gift card management |

---

## 1. Order Lifecycle

### State Flow

```
┌──────────┐     ┌───────────┐     ┌──────────────┐     ┌───────────┐
│  CREATED  │────▶│ CONFIRMED │────▶│ EXPORT_READY │────▶│ EXPORTED  │
└──────────┘     └───────────┘     └──────────────┘     └───────────┘
                                          │                    │
                                          ▼                    ▼
                                   ┌──────────────┐    ┌─────────────┐
                                   │EXPORT_FAILED │    │  COMPLETED  │
                                   └──────────────┘    └─────────────┘
                                          │
                                          ▼
                                   (Retry up to 3x,
                                    then mark FAILED)
```

### Order Types

Orders can spawn related sub-orders for different fulfillment methods:

```
Primary Order
  ├── c_deliveryOrderNo      → Standard shipping order
  ├── c_bopisOrderNo         → Buy Online Pick In Store order
  └── c_sameDayDeliveryOrderNo → Same-day delivery order
```

---

## 2. Order Placement

### Where It Happens

- **React PWA** checkout flow → **BFF Baskets API** → **SFCC Commerce SDK**

### Workflow

```
Customer completes checkout on React PWA
  │
  ▼
BFF /baskets/place-order endpoint
  │
  ├── Validate basket contents and inventory
  ├── Apply tax calculation
  ├── Process payment instruments:
  │     ├── Credit Card (Cybersource)
  │     ├── PayPal
  │     ├── Afterpay
  │     ├── Apple Pay
  │     └── Gift Cards (SVS)
  ├── Create order in SFCC
  └── Split into sub-orders if mixed fulfillment:
        ├── Standard shipping → delivery order
        ├── BOPIS → store pickup order
        └── SDD → same-day delivery order
  │
  ▼
Emarsys sends order confirmation email
  │
  ▼
Order marked EXPORT_READY for OMS
```

---

## 3. Order Export to Oracle OMS

### Where It Happens

- **SFCC scheduled job** (`int_oroms` cartridge)
- **SFCC Business Manager** > Operations > Job Schedules

### Workflow

```
Scheduled Job: Order Export (daily)
  │
  ├── Query orders with exportStatus = EXPORT_STATUS_READY
  ├── Batch processing (configurable, default 1000)
  │
  ▼
For each order batch:
  │
  ├── GenerateXML.js creates XML order representation
  │     (comprehensive ~96KB generator covering all order data)
  ├── Send to Oracle OMS via OROMS service
  │
  ├── On Success:
  │     └── Set exportStatus = EXPORT_STATUS_EXPORTED
  │
  └── On Failure:
        ├── Increment retry counter
        ├── If retries < 3: Keep as EXPORT_STATUS_READY
        ├── If retries >= 3: Set EXPORT_STATUS_FAILED
        └── Send email report of failures
  │
  ▼
Stale Order Cleanup Job (daily at 13:00 UTC)
  │
  └── Orders older than 30 days still READY → mark FAILED
```

### Configurable Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| Order Number | - | Export specific order (manual) |
| Date Range | - | Batch export by date |
| Batch Size | 1000 | Orders per batch |
| Max Retries | 3 | Failure retry limit |
| Email Recipients | - | Failure notification addresses |

---

## 4. Customer Service Order Operations

### Where It Happens

- **BFF Admin API** (`/admin/orders`)
- **BFF Customer Orders API** (`/customers/:id/orders`)
- **SFCC Business Manager** > Orders

### Lookup Workflow

```
CS Rep receives customer inquiry
  │
  ├── Look up by order number:
  │     BFF Admin: GET /admin/orders/:orderNo
  │     Returns order + all related sub-orders
  │
  ├── Look up by customer:
  │     BFF: GET /customers/:customerId/orders
  │     Returns paginated order history
  │
  └── Guest order tracking:
        BFF: POST /customers/validate-track-order
        Validates order number + email/ZIP
```

### Order Update Workflow

```
CS Rep needs to modify order
  │
  ├── BFF: PATCH /orders/:orderNo (with customer auth)
  │     Validates customer ownership
  │     Updates order attributes
  │
  └── SFCC BM: Direct order editing
        For appeasements, cancellations, etc.
```

---

## 5. PayPal Transaction Management

### Where It Happens

- **SFCC Business Manager** > bm_paypal > PayPal Transactions

### Workflow

```
Finance/CS needs to review PayPal transactions
  │
  ▼
BM > Orders > PayPal Transactions
  │
  ├── Search by PayPal transaction ID
  ├── Search by order number
  ├── View PayPalNewTransactions custom objects
  ├── Paginated transaction list
  └── Link to associated SFCC orders
```

---

## 6. Warranty (Extend) Order Processing

### Where It Happens

- **SFCC scheduled jobs** (`int_extend` cartridge)

### Workflow

```
Order contains warrantable product
  │
  ▼
Scheduled Job: Extend Contracts Creation
  │
  ├── Read ExtendContractsQueue custom objects
  ├── Create warranty contracts in Extend system
  ├── Track: order number, product, plan, customer, address
  └── Data retained for 90 days
  │
  ▼
Scheduled Job: Extend Orders Creation
  │
  └── Create corresponding orders in Extend system
  │
  ▼
Scheduled Job: Extend Create Refunds from SFCC
  │
  └── Process warranty refunds when orders are returned
```

---

## 7. Delivery & Shipping

### Where It Happens

- **BFF Delivery module** (`/delivery/`)
- **UPS / FedEx** integrations

### Workflow

```
Operations monitors delivery estimates
  │
  ├── GET /delivery/estimated-delivery-date?zip=...
  │     Returns delivery date by ZIP via FedEx
  │
  └── POST /delivery/validate-address
        UPS/FedEx address validation
  │
  ▼
Emarsys sends shipping confirmation email
  │  (configured in BM > bm_emarsys > Shipping Confirmation)
```

---

## Related Diagrams

- [Order Management Lifecycle](02-order-management-fulfillment.drawio)
- [State Diagram - Cart/Order](../../draw.io/05-state-diagram-cart-order.drawio) (existing)
- [Sequence - API Request Flow](../../draw.io/04-sequence-api-request-flow.drawio) (existing)

---

## Systems Involved

| System | Role |
|--------|------|
| **SFCC Business Manager** | Order management, job scheduling, PayPal review |
| **BFF API** | Order lookup, updates, delivery estimates |
| **Oracle OMS** | Order fulfillment backend (via int_oroms) |
| **Extend** | Warranty contract creation and refunds |
| **Cybersource** | Credit card payment processing |
| **PayPal** | PayPal payment processing |
| **Afterpay** | BNPL payment processing |
| **SVS** | Gift card balance and redemption |
| **Emarsys** | Order/shipping confirmation emails |
| **UPS / FedEx** | Shipping and delivery estimates |
