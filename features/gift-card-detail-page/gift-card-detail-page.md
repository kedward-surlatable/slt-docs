# Gift Card Detail Page (GDP)

The Gift Card Detail Page is the product detail page for Sur La Table gift cards. It handles both physical ("Traditional") and electronic ("E-Gift") card types through a single page component with type-specific form fields.

---

## Routes

| Route | Card Type | Example URL |
|-------|-----------|-------------|
| `/product/traditional-gift-card/:sku` | Physical | `/product/traditional-gift-card/876953?amount=50` |
| `/product/electronic-gift-card/:sku` | Electronic | `/product/electronic-gift-card/876954?amount=100` |

Both routes are defined in `slt-sfcc-react/app/routes.jsx` and render the same `GiftCardDetailPage` component. The card type is determined by `product.c_gcType` (`'physical'` or `'electronic'`).

---

## Architecture

See `gift-card-detail-page.drawio` for visual diagrams.

### Component Tree

```
GiftCardDetailPage (PageFC)
  getProps() ---- ssrBffGet('/products/:sku') ---- BFF /products/:id
  |
  SSRProductProvider (product from SSR)
    PdpProvider (productId from URL params)
      GiftCardDetailPageLayout
        |- Layout
        |- Seo + Helmet (canonical URL)
        |- ProductSchema + FAQSchema + BreadcrumbSchema
        |- CheckBalanceModal
        |- GDPBreadcrumbsSkeleton > Breadcrumb
        |- GDPPhotoGridSkeleton > PhotoGrid
        |- GDPBuyBoxSkeleton > GiftCardBuyBox
        |    |- DesignSelector (variant carousel)
        |    |- GiftCardAmount (chips: $50/$100/$150/$250/$500 + custom input)
        |    |- Form fields (varies by card type)
        |    |- GiftCardMessageInput
        |    |- ApplePayButton
        |    |- AddToBagButton
        |    |- AfterpaySection
        |- GDPWhatCanYouGetSkeleton > WhatCanYouGetSection
        |- DiscoverCookingClass (feature-flagged)
        |- GDPFAQSkeleton > GiftCardsFAQSection
        |- ProductsAddedToBagDrawer
        |- ScrollToTop
```

### Key Files

| File | Purpose |
|------|---------|
| `app/pages/gift-cards/gift-card-details-page/GDP.tsx` | Page component with SSR `getProps` |
| `app/pages/gift-cards/gift-card-details-page/components/GiftCardBuyBox.tsx` | BuyBox form (amount, fields, add-to-bag) |
| `app/pages/gift-cards/gift-card-details-page/components/GiftCardAmount.tsx` | Amount selector with chips + custom input |
| `app/pages/gift-cards/gift-card-details-page/components/DesignSelector.tsx` | Variant design carousel (Swiper) |
| `app/pages/gift-cards/gift-card-details-page/components/WhatCanYouGetSection.tsx` | "What Can You Get" content cards |
| `app/pages/gift-cards/gift-card-details-page/components/DiscoverCookingClass.tsx` | Nearby cooking class carousel |
| `app/pages/gift-cards/gift-card-details-page/components/GDPSkeleton.tsx` | Loading skeletons for each section |
| `app/pages/gift-cards/gift-card-details-page/hooks/useGiftCardLocation.tsx` | URL `?amount` param sync + validation |
| `app/pages/gift-cards/gift-card-details-page/hooks/useGiftCardSuggestedAmounts.ts` | Default amounts: [50, 100, 150, 250, 500] |
| `app/components/gift-card-message-input/GiftCardMessageInput.tsx` | Multi-line message textarea with validation |
| `app/components/forms/useGiftCardFields.tsx` | React Hook Form field configs |
| `app/contexts/pdp/usePDP.tsx` | PDP context provider (product, loading, amount state) |
| `app/contexts/pdp/useSSRProduct.tsx` | SSR product context (passes server-fetched data) |
| `app/bff-api/products/useProduct.tsx` | Product query hook with SSR `initialData` support |

---

## Server-Side Rendering (SSR)

The GDP uses the same SSR pattern as the standard PDP (`PDPLazy.tsx`) to deliver product data on first render.

### Flow

1. **Server**: `getProps({ req, res, params })` runs on the server (gated by `shouldGetProps = isServer`)
2. **Server**: Validates SKU with `isValidSKU(params.sku)`
3. **Server**: Calls `ssrBffGet<Product>('/products/{sku}')` to fetch the product from the BFF with SSR authentication
4. **Server**: Checks if the URL matches `product.c_pwaUrl` — if not, issues an HTTP redirect (see Master vs Variant SKU section)
5. **Server**: Calls `configHtmlCache(res)` to set `Cache-Control: public, must-revalidate, max-age=3600` on the response
6. **Server**: Returns `{ product }` as props to the component
7. **Client**: `GiftCardDetailPage` passes `product` to `SSRProductProvider`
8. **Client**: `PdpProvider` calls `useProduct(sku)` which reads from `useSSRProduct()`
9. **Client**: If `sku === serverProduct.id`, the SSR product is used as React Query `initialData` — **no client-side fetch needed**, `isLoading` is `false` immediately

### Error Handling

If `ssrBffGet` fails (network error, 404, etc.), `getProps` catches the error and returns `{}`. The page still renders with client-side-only fetching as a fallback. Cache headers are still set in the catch block.

### Comparison with Standard PDP

| Aspect | Gift Card GDP | Standard PDP |
|--------|--------------|--------------|
| SSR product fetch | `ssrBffGet('/products/:sku')` | Same |
| SSR assets | Not needed (DesignSelector builds client-side) | `buildAssets()` for photo grid |
| Redirect logic | Master SKU → variant via `c_pwaUrl` | Same pattern + `pageURL` slug check |
| Preview mode | Not supported | Supported via `?preview=true` |
| Cache TTL | 1 hour (production) | 1 hour (production) |

---

## Master SKU vs Variant SKU (SLTBE-7888)

This section documents the root cause and fix for SLTBE-7888: "PWA-GDP-Indexed pages not loading up the BuyBox section."

### The Problem

Google indexes gift card pages using the **master product SKU** (e.g., `876953`) because the `<link rel="canonical">` tag uses `c_canonicalProductId` (the master). But when users navigate the site, the DesignSelector routes them to **variant SKUs** (e.g., `5940804` for a specific design).

```
Google indexes:  /product/traditional-gift-card/876953    (master SKU)
Site navigates:  /product/traditional-gift-card/5940804   (variant SKU)
```

Without SSR, the page depended entirely on the client-side SFCC session + BFF fetch chain. When a user landed directly on the master SKU URL from Google (no prior session), the client-side fetch could fail, leaving the BuyBox blank.

### The Fix

The GDP now mirrors the PDP's SSR pattern in `getProps`:

1. **Fetch**: `ssrBffGet('/products/876953')` resolves the master product server-side with SLAS authentication
2. **Redirect**: Compares `req.url` path with `product.c_pwaUrl`:
   - If master product (`product.type?.master`): **301 redirect** to variant URL
   - If URL slug mismatch: **302 redirect** to canonical URL
   - If URLs match: serve product data directly
3. **Serve**: For matching variant SKU URLs, returns `{ product }` for SSR hydration

### Redirect Logic Detail

```
User visits: /product/traditional-gift-card/876953 (master)
  Server: ssrBffGet('/products/876953') → product with c_pwaUrl=/product/traditional-gift-card/5940804
  Server: URL mismatch detected, product.type.master = true
  Server: res.redirect(301, '/product/traditional-gift-card/5940804')

User follows redirect: /product/traditional-gift-card/5940804 (variant)
  Server: ssrBffGet('/products/5940804') → product with c_pwaUrl=/product/traditional-gift-card/5940804
  Server: URLs match
  Server: return { product } → BuyBox renders immediately
```

### Why Variant SKUs Worked Without SSR

Variant SKUs could be fetched client-side because the user had typically navigated from another page first (establishing the SFCC session). The master SKU URL from Google was a cold page load with no prior session.

### Client-Side URL Normalization

The `GiftCardDetailPageLayout` component also has client-side URL normalization in a `useEffect` (lines 74-91). This handles SPA navigation cases where the URL slug might be wrong but the variant SKU is valid. It uses `history.replace()` (no HTTP status code) to correct the slug while preserving the variant SKU from the URL. This is separate from and complementary to the SSR redirect.

---

## BuyBox

### Amount Selection

The BuyBox supports gift card amounts from **$10 to $500**.

**Preset chips**: $50, $100, $150, $250, $500 (rendered by `GiftCardAmount` using `ChipList`)

**Custom input**: Free-text field allowing any integer amount within range. Capped at $500 on input, validated on blur.

**URL sync**: The selected amount is kept in sync with the `?amount=XX` query parameter via `useGiftCardLocation`:
- On page load: reads `?amount` from URL, validates, defaults to `$50` if missing/invalid
- On amount change: updates URL via `history.replace` (no history entry)
- Range: $10 minimum, $500 maximum

### Form Fields by Card Type

| Field | Physical | Electronic |
|-------|----------|------------|
| Design Selector | Yes | Yes |
| Amount | Yes | Yes |
| Sender Name | Yes | Yes |
| Recipient Name | No | Yes |
| Recipient Email | No | Yes |
| Send-to-Me Toggle | No | Yes |
| Gift Message | Yes | Yes |
| Quantity | Yes (max 99) | No |

### Gift Message

Handled by `GiftCardMessageInput`:
- Maximum 4 lines, 40 characters per line (160 total)
- Strips emojis and special characters (`~@^*()=+[]{}|"<>/`)
- Custom paste handling with sanitization
- Line count and character count display

### Add to Bag

The `AddToBagButton` calls form validation before adding. Gift card products:
- Skip inventory validation (unlimited inventory)
- Enforce minimum `c_gcAmount` of $10
- Use dummy product IDs for parallel cart additions
- Expand quantity > 1 into multiple single-quantity line items

---

## Design Selector

The `DesignSelector` component renders a Swiper carousel of gift card design variants:
- Derives assets from `product.c_variants` filtered to in-stock variants
- Each slide shows `variant.c_imageUrl` with the design name from `variationAttributes[0].values`
- Clicking a design navigates via URL: `navigateTo(buildPdpURL(product, designId))`
- Selected design has a highlighted border
- Navigation arrows appear when slides overflow the container

---

## Feature Flags

| Preference | Purpose | Default |
|------------|---------|---------|
| `GIFT_CARDS_DISCOVER_COOKING_CLASSES_NEAR_YOU` | Show "Discover Cooking Classes" section | `false` |
| `GIFT_CARDS_WHAT_CAN_YOU_GET_LINKS` | Override "What Can You Get" link URLs | Hardcoded paths |
| `PHYSICAL_GIFT_CARD_DELIVERY_NOTIFICATION` | Physical card delivery message | `''` |
| `ELECTRONIC_GIFT_CARD_DELIVERY_NOTIFICATION` | E-gift card delivery message | `''` |
