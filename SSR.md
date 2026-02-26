# Server-Side Rendering (SSR)

How the React PWA renders pages on the server before sending them to the browser.

## Overview

The Sur La Table React app uses **Salesforce PWA Kit** for server-side rendering. When a user requests a page, an Express server running on Node.js renders the React component tree into HTML, sends it to the browser, and then React "hydrates" the static HTML into a fully interactive application.

This gives users a fast initial page load (no blank screen while JavaScript downloads) while still providing the rich interactivity of a single-page app once hydration completes.

## Architecture

```
Browser (localhost:8000 or production URL)
    │
    ▼
┌──────────────────────────────────┐
│  Proxy Server (port 8000)        │
│  proxy/server.ts                 │
│  Routes PWA paths → SSR server   │
│  Routes other paths → SFCC       │
└──────────┬───────────────────────┘
           │
    ┌──────┴──────┐
    ▼             ▼
┌────────┐   ┌────────────┐
│ SSR    │   │ SFCC       │
│ :3000  │   │ SiteGenesis│
└────────┘   └────────────┘
```

### Request Flow

1. Browser requests `localhost:8000/search?q=cookware`
2. Proxy matches `/search` as a PWA route and forwards to `localhost:3000`
3. SSR server matches the route to the `SearchResults` component
4. `SearchResults.getProps()` runs server-side — fetches data from Commerce Cloud APIs
5. React renders the component tree to an HTML string
6. HTML is wrapped in the `_document` template with `<head>` tags, scripts, and styles
7. Response sent to browser — user sees a fully rendered page
8. Browser downloads JavaScript bundles and React hydrates the HTML
9. Page becomes interactive — React takes over DOM management

## Key Files

| File | Purpose |
|------|---------|
| `app/ssr.js` | Express server entry point — configures middleware and starts the SSR runtime on port 3000 |
| `app/main.jsx` | Client entry point — calls `start()` to hydrate SSR HTML and registers the service worker |
| `app/routes.jsx` | Route definitions mapping URL paths to React components with lazy loading |
| `app/components/_document/index.jsx` | HTML document shell wrapping SSR output (`<html>`, `<head>`, `<body>`) |
| `app/request-processor.js` | Pre-routing filter that strips marketing query params (UTM, gclid, fbclid, etc.) |
| `proxy/server.ts` | Proxy server that routes between the SSR app (port 3000) and SFCC |
| `webpack.config.js` | Builds separate bundles for client (browser), SSR (Node), and renderer targets |

## SSR Server (`app/ssr.js`)

The SSR server is an Express application built on PWA Kit's runtime:

```javascript
const { getRuntime } = require('@salesforce/pwa-kit-runtime/ssr/server/express');

const options = {
    buildDir: path.resolve(process.cwd(), 'build'),
    defaultCacheTimeSeconds: 600,
    mobify: getConfig(),
    port: 3000,
    protocol: process.env.PROTOCOL || 'http'
};

const runtime = getRuntime();
const app = runtime.createHandler(options, (app) => {
    // Register middleware...
});
```

### Middleware Pipeline

Middleware is registered in order on the Express app:

| Order | Middleware | Purpose |
|-------|-----------|---------|
| 1 | `plusForSpaceInTheUrl` | Decodes `%2B` to `+` in URLs |
| 2 | `addSSRUserAgent` | Tracks request user agent globally |
| 3 | `defaultPwaKitSecurityHeaders` | PWA Kit security defaults |
| 4 | `redirectByRules` | Applies redirect rules from `utils/redirect-rules.js` |
| 5 | `/commerce-api` proxy | Proxies API calls to Salesforce Commerce Cloud |
| 6 | `blockIp` | IP allowlist enforcement |
| 7 | `cookieParser` | Parses incoming cookies |
| 8 | `helmet` | Sets security headers (CSP, HSTS, X-Frame-Options) |

### Commerce API Proxy

The SSR server proxies `/commerce-api/*` requests to Commerce Cloud:

- Target: `https://{shortCode}.api.commercecloud.salesforce.com`
- Supports SLAS private client authentication via `X-SLT-RewriteAuth` header
- Path rewrites for `/mobify/slas/private` endpoints

## Data Fetching with `getProps`

Pages implement a static `getProps` method that runs on the server during SSR:

```typescript
interface IGetProps {
    api: ICommerceAPI;              // Commerce API client
    req: Partial<Request>;          // Express request
    res: Partial<Response>;         // Express response
    params: Record<string, string>; // URL route params
    location: Location;             // Current URL location
}

const SearchResults: PageFC = (props) => {
    return <div>{/* render with pre-fetched props */}</div>;
};

SearchResults.getProps = async ({ api, req, res, params }) => {
    configHtmlCache(res);  // Set Cache-Control headers
    const results = await api.shopperSearch.productSearch({ /* ... */ });
    return { results };
};
```

**How it works:**
- PWA Kit calls `getProps` on the matched route component during SSR
- The returned object is passed as props to the component for rendering
- The same data is serialized into the HTML so the client can hydrate without re-fetching
- `shouldGetProps` can optionally control whether `getProps` runs

## Client-Side Hydration (`app/main.jsx`)

After the browser receives the server-rendered HTML:

```javascript
import { registerServiceWorker, start } from '@salesforce/pwa-kit-react-sdk/ssr/browser/main';

const main = () =>
    Promise.all([start(), registerServiceWorker('/worker.js')]);
```

1. `start()` — React hydrates the SSR HTML inside `<div class="react-target">`, attaching event listeners and making the page interactive
2. `registerServiceWorker('/worker.js')` — Registers the PWA service worker for offline support and asset caching
3. After hydration, client-side routing takes over — subsequent navigations are handled by React Router without full page reloads

## HTML Document Template (`app/components/_document/index.jsx`)

The `_document` component wraps all SSR output:

```jsx
<html {...htmlAttributes}>
    <head>
        {/* Meta tags, stylesheets from react-helmet */}
        {head.map(child => child)}
    </head>
    <body {...bodyAttributes}>
        {afterBodyStart.map(child => child)}
        <div className="react-target"
             dangerouslySetInnerHTML={{ __html: html }} />
        {beforeBodyEnd.map(child => child)}
    </body>
</html>
```

- `html` — The stringified React component tree
- `head` — Elements collected by `react-helmet` (meta tags, title, stylesheets)
- `afterBodyStart` / `beforeBodyEnd` — Injection points for scripts and tracking snippets

## Route Configuration (`app/routes.jsx`)

Routes use `@loadable/component` for code splitting so only the JavaScript for the current page is loaded:

```javascript
const SearchResults = loadable(() => import('./pages/search-results'), {
    fallback: <SearchResultsSkeleton />
});

const routes = [
    { path: '/search', component: SearchResults, exact: true },
    { path: '/product/:id/:sku', component: PDPLazy, exact: true },
    { path: '/checkout', component: Checkout, exact: true },
    // ...40+ routes
    { path: '*', component: DynamicCategory }  // Catch-all
];
```

Each route specifies a `fallback` skeleton component shown while the page bundle loads during client-side navigation.

## Request Processor (`app/request-processor.js`)

Before routing, the `processRequest` function strips marketing and tracking query parameters from URLs:

- **UTM params**: `utm_source`, `utm_medium`, `utm_campaign`, etc.
- **Ad platform IDs**: `gclid`, `fbclid`, `msclkid`, `gbraid`, `ttclid`
- **70+ total params** stripped to improve CDN cache hit rates

Exception: `utm_medium=email` is preserved for email attribution tracking.

## Caching Strategy

| Layer | TTL | Details |
|-------|-----|---------|
| HTML responses (production) | 600s default | Set via `configHtmlCache()` in `getProps` |
| HTML responses (local dev) | 0s (no cache) | Override with `ENABLE_LOCAL_HTML_CACHE=true` |
| React Query data | 10 minutes | Stale time for client-side API responses |
| SSR pre-fetched data | Infinity | Prevents client re-fetch of server-fetched data |
| Static assets | 1 year | Cached by service worker |

## Proxy and Port Relationship

The proxy server (`proxy/server.ts`) decides which requests go to SSR vs SFCC:

**PWA routes** forwarded to SSR on port 3000:
- `/search/*`, `/product/*`, `/checkout/*`, `/shopping-cart`
- `/account/*`, `/stores/*`, `/recipes/*`, `/registry/*`
- `/gift-cards/*`, `/compare-products`, `/order-confirmation`
- `/callback`, `/worker.js`, `/pwa/*`
- Category pages: `/gifts`, `/brands`, `/holidays`, `/sale`, `/new`, etc.

**Everything else** forwarded to SFCC SiteGenesis (culinary classes, home page, etc.)

A safety net in `LoadFromServer.tsx` redirects any direct `localhost:3000` access to `localhost:8000` so requests always flow through the proxy.

## Running Locally

```bash
# Full stack (proxy + SSR) — recommended
npm run dev

# SSR only (no proxy, direct access on :3000)
npm run dev:noproxy

# Proxy only (requires SSR to already be running)
npm run proxy
```

`npm run dev` uses `concurrently` to start both the proxy (port 8000) and SSR server (port 3000). If only the proxy is running, requests to PWA routes will fail with `ECONNREFUSED` on port 3000.
