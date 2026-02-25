# Sur La Table E-Commerce Platform - Codebase Overview

This is a monorepo containing 4 interconnected projects that form a headless Salesforce Commerce Cloud implementation with a modern decoupled architecture.

---

## 📦 Project Structure

The monorepo consists of:

1. **slt-sfcc** - Salesforce Commerce Cloud Backend
2. **slt-sfcc-react** - React PWA Storefront
3. **slt-sfcc-bff** - Backend-For-Frontend (NestJS API Gateway)
4. **slt-sfcc-configs** - Configuration Management

---

## 🏗️ Project Details

### 1. slt-sfcc - SFCC Backend

**Location:** `/slt-sfcc/`

Traditional Salesforce Commerce Cloud backend using the SiteGenesis framework with a modular cartridge architecture.

#### Cartridge Organization

| Type | Count | Purpose | Examples |
|------|-------|---------|----------|
| **App Cartridges** | 5 | Core application logic | `app_surlatable_core`, `app_surlatable_controllers`, `app_giftregistry` |
| **Integration Cartridges** | 26+ | Third-party service integrations | Payment (Afterpay, Cybersource, PayPal), Marketing (Emarsys, Bloomreach), OMS (int_oroms), Tax/Shipping (Avatax, UPS) |
| **Business Manager** | 6 | Backend administrative tools | Tools, settings, reports, integrations |
| **Plugins** | 1 | SFCC platform plugins | `plugin_slas` (SLAS authentication) |
| **Core Libraries** | - | Shared utilities | `bc_library`, `bc_integrationframework` |

#### Key Directories

- **`bin/`** - Build and build tools scripts
- **`app_surlatable_core/cartridge/`** - Core app files:
  - `controllers/` - Controller + Action methods
  - `templates/` - ISML server-side templates
  - `js/` - Client-side JavaScript (CommonJS modules)
  - `scss/` - SCSS stylesheets
  - `scripts/` - Server-side utility scripts
  - `forms/` - Form definitions (multiple languages)
- **`test/`** - Unit and E2E test automation
  - `unit/` - Mocha/Chai unit tests
  - `application/web*` - WebdriverIO E2E tests
- **`doc/`** - Documentation and styleguide

#### Technology Stack

- **Language:** JavaScript (server), ISML (templates), SCSS
- **Module System:** CommonJS, Browserify
- **Build Tools:** Gulp, Grunt, npm scripts, node-sass
- **Testing:** Mocha + Chai (unit), WebdriverIO + Selenium (E2E)
- **Key Libraries:** jQuery, lodash, Handlebars
- **Deployment:** WebDAV upload to SFCC instances

---

### 2. slt-sfcc-react - React PWA Storefront

**Location:** `/slt-sfcc-react/`

Modern React 18 + TypeScript Progressive Web App (~310K lines of code) providing the customer-facing storefront.

#### Key Directories

- **`app/pages/`** - Feature pages with co-located components/hooks
- **`app/components/`** - Shared components across features
- **`app/ui-components/`** - Design system components (Chakra UI based)
- **`app/hooks/`** - Global custom React hooks
- **`app/utils/`** - Global utility functions
- **`app/types/`** - TypeScript type definitions
- **`app/theme/`** - Design tokens and Chakra customizations
- **`app/bff-api/`** - Client for BFF backend
- **`app/commerce-api/`** - Salesforce Commerce API integration
- **`app/contexts/`** - React context providers
- **`app/constants/`** - Application constants
- **`app/translations/`** - i18n translation files
- **`tests/`** - Unit, integration, and Lighthouse tests
- **`stories/`** - Storybook component documentation
- **`.storybook/`** - Storybook configuration

#### Technology Stack

- **Framework:** React 18, TypeScript 4.9
- **Build Tool:** Webpack 5 with Babel
- **UI Framework:** Chakra UI 2.8, Tailwind CSS 3.4
- **Styling:** Emotion (CSS-in-JS), SCSS modules
- **State Management:** React Query (TanStack), React Context
- **Forms:** React Hook Form
- **Routing:** React Router 5.3
- **Internationalization:** react-intl
- **Testing:** Jest 29, React Testing Library, Lighthouse performance tests
- **Documentation:** Storybook 9
- **Deployment:** Salesforce Managed Runtime (MRT)
- **Requirements:** Node.js ^22.0.0, npm ^10.0.0

#### Key Features

- Server-side rendering (SSR) for performance
- Progressive Web App capabilities (offline support, installable)
- Multi-language internationalization
- Accessible, mobile-first design
- Comprehensive component library with Storybook

---

### 3. slt-sfcc-bff - Backend-For-Frontend (NestJS)

**Location:** `/slt-sfcc-bff/`

NestJS-based API gateway that acts as a unified backend layer between the React frontend and multiple services (SFCC, third-party APIs).

#### Module Structure

**Core Domains** (`src/core/`):
- `baskets/` - Shopping cart operations
- `customers/` - Customer management
- `products/` - Product catalog
- `stores/` - Store locator
- `categories/` - Product categories
- `content/` - Content management
- `orders/` - Order management

**Integration Modules** (47+ NestJS services):
- `bloomreach/` - Search functionality
- `emarsys/` - Email marketing
- `rio-seo/` - SEO services
- `turn-to/` - Ratings and reviews
- `radar/` - Location services
- `optiversal/` - Content optimization
- `bopis/` - Buy Online Pickup In Store
- `go-local/` - Local delivery
- `delivery/` - Delivery services
- `gift-card/` - Gift card management
- `registry/` - Gift registry
- `recaptcha/`, `svs/`, `wsdl/`, `webhooks/` - Additional integrations

**Infrastructure Modules:**
- `commerce/` - Salesforce Commerce Cloud SDK integration
- `app-config/` - Configuration management (AWS SSM)
- `utils/` - Shared utilities
- `bridge/` - Server configuration bridge
- `admin/` - Admin endpoints
- `health/` - Health check endpoints

#### Key Files

- **`src/main.ts`** - Standard Node.js Express server entry point
- **`src/lambda.ts`** - AWS Lambda handler
- **`src/app.module.ts`** - Root NestJS module
- **`.env.develop`, `.env.staging`, `.env.production`** - Environment-specific variables
- **`jest.config.js`, `jest-integration.json`, `jest-e2e.json`** - Test configurations

#### Technology Stack

- **Framework:** NestJS 11 (TypeScript-first)
- **Runtime:** Node.js >=22.18.0 (AWS Lambda compatible)
- **HTTP Server:** Express
- **API Docs:** Swagger/OpenAPI
- **Validation:** class-validator, class-transformer
- **Caching:** cache-manager with Redis or in-memory
- **Logging:** Pino with structured logging
- **Testing:** Jest with auto-mocking support
- **SDKs:** Salesforce commerce-sdk, commerce-sdk-isomorphic
- **HTTP Clients:** Axios with retry logic
- **External Integrations:** 25+ third-party APIs

#### Key Features

- Single unified API for React frontend
- Integration orchestration between SFCC and third-party services
- Caching and performance optimization layer
- Data transformation and enrichment
- Custom business logic (gift registries, BOPIS, local delivery)
- Deployable as Node.js service or AWS Lambda

---

### 4. slt-sfcc-configs - Configuration Management

**Location:** `/slt-sfcc-configs/`

Git-based configuration repository for managing Salesforce B2C Commerce settings across environments.

#### Environment Directories

- **`development/`** - Development environment configurations
- **`staging/`** - Staging environment configurations
- **`production/`** - Production environment configurations
- **`staging_test/`** - Staging test environment
- **`production_test/`** - Production test environment

#### Configuration Files (per environment)

- **`system_objects.xml`** - Custom attributes, preferences metadata
- **`custom_objects.xml`** - Custom object schema definitions
- **`ocapi-shop-global.json`** - SFCC Shop API settings
- **`ocapi-data-global.json`** - SFCC Data API settings
- **`pwa_mrtrules.json`** - CDN/MRT routing rules for Managed Runtime

#### Key Features

- Environment-specific configuration management
- OCAPI permissions per environment
- Custom object definitions
- CDN routing configuration
- Git-based version control for audit trail

---

## 🔗 Architecture Overview

```
┌─────────────────────────────────────────────────────────┐
│               slt-sfcc-react (React PWA)                │
│          Salesforce Managed Runtime Hosting             │
└─────────────────┬───────────────────────────────────────┘
                  │ HTTP API Calls
                  ▼
┌─────────────────────────────────────────────────────────┐
│     slt-sfcc-bff (NestJS API Gateway & Orchestrator)    │
│   - Caching Layer                                       │
│   - Integration Orchestration (47+ services)            │
│   - Data Transformation                                 │
│   Deployable as: Node.js Service or AWS Lambda         │
└──────┬──────────────────────────────────────────┬───────┘
       │                                          │
       │ Commerce SDK & REST APIs               │ Third-Party APIs
       ▼                                          ▼
┌──────────────────────────────┐  ┌──────────────────────────────┐
│  slt-sfcc (SFCC Backend)     │  │ External Services (26+):     │
│  - SiteGenesis Framework      │  │ - Bloomreach (Search)        │
│  - 37+ Cartridges             │  │ - Emarsys (Marketing)        │
│  - Product Catalog            │  │ - Afterpay (Payments)        │
│  - Shopping Cart              │  │ - UPS (Shipping)             │
│  - Order Management           │  │ - Avatax (Tax)               │
│  - Integrations               │  │ - RADAR (Location)           │
│  - Business Manager Tools     │  │ - And many more...           │
└──────────────────────────────┘  └──────────────────────────────┘

        ┌──────────────────────────────────────┐
        │  slt-sfcc-configs (Configuration)    │
        │  - System Objects                    │
        │  - Custom Objects                    │
        │  - OCAPI Rules                       │
        │  - CDN Routing                       │
        └──────────────────────────────────────┘
```

---

## 🛠️ Technology Stack Summary

| Layer | Technologies |
|-------|--------------|
| **Frontend** | React 18, TypeScript, Chakra UI, Tailwind CSS, Emotion |
| **PWA Framework** | Salesforce PWA Kit |
| **Backend API** | NestJS 11, Express, TypeScript |
| **Commerce Backend** | Salesforce Commerce Cloud (SiteGenesis) |
| **Caching** | Redis / cache-manager |
| **Logging** | Pino (structured logging) |
| **Testing** | Jest, React Testing Library, Mocha/Chai, WebdriverIO, Lighthouse |
| **Build Tools** | Webpack 5, Babel, Gulp, Grunt, node-sass |
| **Code Quality** | ESLint, Prettier, TypeScript strict mode |
| **Documentation** | Storybook 9, JSDoc, Swagger/OpenAPI |
| **Deployment** | Salesforce Managed Runtime, AWS Lambda, CloudFlare CDN |
| **Infrastructure** | AWS (Parameter Store, CloudWatch, Lambda) |

---

## 📋 Quick Reference

### Environment Setup

Each project has its own requirements:
- **slt-sfcc:** Java (for Maven), Node.js
- **slt-sfcc-react:** Node.js ^22.0.0, npm ^10.0.0
- **slt-sfcc-bff:** Node.js >=22.18.0, npm >=10.0.0
- **slt-sfcc-configs:** No runtime requirements (configuration files)

### Key Integration Points

1. **React → BFF:** React app calls BFF API endpoints at `app/bff-api/`
2. **BFF → SFCC:** BFF uses Salesforce Commerce SDK to communicate with SFCC
3. **BFF → Third Parties:** 47+ integration modules handle external service calls
4. **Configuration:** SFCC configs loaded from slt-sfcc-configs, overridable via environment variables

### Testing Strategy

- **Unit Tests:** Jest (React, NestJS), Mocha/Chai (SFCC)
- **Integration Tests:** Jest integration suites
- **E2E Tests:** WebdriverIO (SFCC), Jest E2E (React)
- **Performance:** Lighthouse tests for React PWA
- **API Documentation:** Swagger/OpenAPI (NestJS BFF)

---

## 📚 Documentation

- **slt-sfcc:** `/doc/` directory
- **slt-sfcc-react:** README.md, Storybook at `/.storybook/`
- **slt-sfcc-bff:** Swagger/OpenAPI endpoints at `/api/docs`
- **slt-sfcc-configs:** Git history and comments in XML/JSON files

---

## 🚀 Deployment

- **slt-sfcc-react:** Salesforce Managed Runtime (MRT)
- **slt-sfcc-bff:** Node.js service or AWS Lambda
- **slt-sfcc:** WebDAV deployment to SFCC instances
- **slt-sfcc-configs:** Git-based, applied through SFCC import/export tools

---

## 📊 Project Statistics

- **slt-sfcc-react:** ~310K lines of TypeScript/React code
- **slt-sfcc-bff:** 47+ NestJS integration modules
- **slt-sfcc:** 37+ cartridges with 50+ controller classes
- **Third-party integrations:** 26+ external services

---

Generated with Claude Code
