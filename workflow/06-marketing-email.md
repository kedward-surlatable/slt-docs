# Marketing & Email Workflow

Internal workflow for marketing managers and email campaign operators who manage customer communications, newsletter subscriptions, and marketing automation through the Emarsys platform.

---

## Roles

| Role | Responsibilities |
|------|-----------------|
| **Marketing Manager** | Campaign strategy, newsletter content, promotions |
| **Email Operations** | Emarsys configuration, email templates, delivery monitoring |
| **Data Analyst** | SmartInsight analytics, predictive models, customer segments |

---

## 1. Emarsys Configuration (SFCC Business Manager)

### Where It Happens

- **SFCC Business Manager** > bm_emarsys (6 configuration screens)

### Configuration Areas

```
BM > bm_emarsys Menu
  │
  ├── Newsletter Subscriptions
  │     Configure subscription collection methods:
  │     ├── Footer subscription widget
  │     ├── Checkout opt-in
  │     └── Account creation opt-in
  │
  ├── Order Confirmation Configuration
  │     Map order data fields to Emarsys email template variables
  │
  ├── Shipping Confirmation Configuration
  │     Map shipping data fields to Emarsys email template variables
  │
  ├── SmartInsight Configuration
  │     Configure analytics field mapping for customer behavior tracking
  │
  ├── Predict Configuration
  │     Set up predictive recommendation engine parameters
  │
  └── Database Load Configuration
        Configure customer data synchronization to Emarsys
        (bulk data export for segmentation)
```

---

## 2. Newsletter Subscription Management

### Where It Happens

- **BFF Customers API** (`/customers/newsletter-subscription`)
- **SFCC BM** > bm_emarsys > Newsletter Subscriptions
- **Emarsys** platform (external)

### Subscription Workflow

```
Customer subscribes via storefront
  │
  ├── Footer email signup widget
  ├── Checkout newsletter opt-in checkbox
  └── Account registration newsletter opt-in
  │
  ▼
BFF: POST /customers/newsletter-subscription
  │
  ▼
Subscription sent to Emarsys
  │
  ▼
Customer added to Emarsys contact list
  │
  ▼
Marketing sends targeted campaigns via Emarsys
```

### Configuration Workflow

```
Marketing Manager configures subscription channels
  │
  ▼
BM > bm_emarsys > Newsletter Subscriptions
  │
  ├── Define subscription source identifiers
  ├── Map form fields to Emarsys contact attributes
  └── Configure opt-in/opt-out behavior per channel
```

---

## 3. Transactional Email Workflows

### Account Created Email

```
Customer creates new account
  │
  ▼
BFF: POST /emarsys/email/account-created
  │
  ▼
Emarsys sends welcome email
  │  (template configured in Emarsys dashboard)
```

### Create Account Invitation Email

```
System invites customer to create account
  │
  ▼
BFF: POST /emarsys/email/create-account
  │
  ▼
Emarsys sends account creation invitation email
```

### Password Reset Email

```
Customer requests password reset
  │
  ▼
SFCC generates reset token
  │
  ▼
Webhook: POST /webhooks/reset-password-token
  │  (SLAS callback token validated)
  │
  ▼
Emarsys sends password reset email with link
```

### Order Confirmation Email

```
Order placed successfully
  │
  ▼
SFCC triggers order confirmation
  │  (field mapping configured in BM > bm_emarsys > Order Confirmation)
  │
  ▼
Emarsys sends order confirmation email
  │  Includes: order details, items, totals, shipping info
```

### Shipping Confirmation Email

```
Order shipped (updated from OMS)
  │
  ▼
SFCC triggers shipping notification
  │  (field mapping configured in BM > bm_emarsys > Shipping Confirmation)
  │
  ▼
Emarsys sends shipping confirmation email
  │  Includes: tracking number, carrier, estimated delivery
```

---

## 4. Customer Data Synchronization

### Where It Happens

- **SFCC BM** > bm_emarsys > Database Load Configuration
- **Emarsys** platform

### Workflow

```
Marketing configures data export rules
  │  BM > bm_emarsys > Database Load Configuration
  │
  ├── Select customer attributes to sync
  ├── Configure sync frequency
  └── Map SFCC fields to Emarsys contact attributes
  │
  ▼
Scheduled data sync runs
  │
  ├── Extract customer data from SFCC
  ├── Transform to Emarsys format
  └── Bulk load to Emarsys contact database
  │
  ▼
Emarsys uses data for:
  ├── Customer segmentation
  ├── Personalized campaigns
  ├── Behavioral targeting
  └── SmartInsight analytics
```

---

## 5. SmartInsight & Predictive Analytics

### Where It Happens

- **SFCC BM** > bm_emarsys > SmartInsight Configuration
- **SFCC BM** > bm_emarsys > Predict Configuration
- **Emarsys** platform

### SmartInsight Workflow

```
Data Analyst configures SmartInsight fields
  │  BM > bm_emarsys > SmartInsight Configuration
  │
  ├── Map customer behavior data:
  │     ├── Purchase history
  │     ├── Browse behavior
  │     ├── Email engagement
  │     └── Lifecycle stage
  │
  ▼
Data flows to Emarsys SmartInsight
  │
  ▼
Emarsys generates:
  ├── Customer lifecycle segments
  ├── Churn predictions
  ├── Revenue attribution
  └── Campaign effectiveness reports
```

### Predict Configuration Workflow

```
Marketing configures recommendation engine
  │  BM > bm_emarsys > Predict Configuration
  │
  ├── Set recommendation parameters
  ├── Configure product data feed
  └── Define recommendation contexts (email, web)
  │
  ▼
Emarsys Predict generates:
  ├── Personalized product recommendations
  ├── "You may also like" suggestions
  └── Cross-sell / up-sell recommendations
```

---

## 6. Bloomreach Search Analytics

### Where It Happens

- **Bloomreach Dashboard** (external)
- **BFF Bloomreach module** telemetry

### Telemetry Workflow

```
Customer searches on storefront
  │
  ▼
BFF Bloomreach module processes search
  │
  ├── Log search query, results, clicks
  ├── Track zero-result searches
  └── Measure search-to-purchase conversion
  │
  ▼
Telemetry pipeline:
  CloudWatch Logs → Lambda → Aurora database
  │
  ▼
Marketing reviews search analytics:
  ├── Top search queries
  ├── Failed searches (no results)
  ├── Search conversion rates
  └── Popular product categories from search
```

---

## 7. Google Tag Manager (Analytics)

### Where It Happens

- **SFCC** (`int_googletagmanager` cartridge)
- **GTM Dashboard** (external)

### Workflow

```
GTM data layer events fire on user actions:
  │
  ├── Page views
  ├── Product impressions
  ├── Add to cart
  ├── Checkout steps
  ├── Purchases
  └── Custom events
  │
  ▼
GTM routes data to:
  ├── Google Analytics
  ├── Marketing pixels
  ├── Retargeting platforms
  └── Custom data destinations
```

---

## Related Diagrams

- [Marketing & Email Flow](06-marketing-email.drawio)

---

## Systems Involved

| System | Role |
|--------|------|
| **SFCC Business Manager** | Emarsys configuration (6 screens) |
| **BFF Emarsys API** | Transactional email triggers |
| **BFF Webhooks** | Password reset token handling |
| **Emarsys** | Email delivery, campaigns, SmartInsight, Predict |
| **Bloomreach** | Search analytics and telemetry |
| **Google Tag Manager** | Analytics data layer |
| **CloudWatch / Lambda / Aurora** | Search telemetry pipeline |
