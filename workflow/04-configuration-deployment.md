# Configuration & Deployment Workflow

Internal workflow for developers and DevOps engineers managing environment configuration, code deployments, and infrastructure across all four platform components.

---

## Roles

| Role | Responsibilities |
|------|-----------------|
| **Developer** | Code changes, local testing, PR creation |
| **DevOps / Platform Engineer** | Deployment pipelines, infrastructure, environment config |
| **Release Manager** | Coordinating releases across components |
| **QA Engineer** | Validating deployments in staging/test environments |

---

## 1. Configuration Management (slt-sfcc-configs)

### Git-Based Config Workflow

```
Developer creates feature branch
  │  Branch naming: SLTBE-[TICKET]/description
  │
  ├── Edit environment-specific config files:
  │     ├── development/    (system_objects.xml, custom_objects.xml, etc.)
  │     ├── staging/
  │     ├── staging_test/
  │     ├── production/
  │     └── production_test/
  │
  ├── Validate changes:
  │     ├── XML well-formedness
  │     ├── JSON validity
  │     ├── No credential leaks
  │     ├── Preserve attribute IDs (never rename existing)
  │     └── Follow least-privilege for OCAPI permissions
  │
  └── Create PR with Jira key prefix
       │
       ▼
  CODEOWNERS auto-review
       │
       ▼
  Merge to target branch
       │
       ▼
  Import to SFCC via Business Manager
       BM > Administration > Operations > Import/Export
```

### Configuration File Types

| File | Purpose | Import Method |
|------|---------|--------------|
| `system_objects.xml` | System object extensions, custom attributes | SFCC Import/Export |
| `custom_objects.xml` | Custom object type definitions | SFCC Import/Export |
| `ocapi-shop-global.json` | Shop API permissions per client | SFCC BM > OCAPI Settings |
| `ocapi-data-global.json` | Data API permissions | SFCC BM > OCAPI Settings |
| `pwa_mrtrules.json` | CDN routing rules for Managed Runtime | Commerce API `updateMrtRuleset` |

### Environment Promotion Flow

```
development/ ──▶ staging/ ──▶ staging_test/ ──▶ production_test/ ──▶ production/
     │               │              │                  │                 │
     ▼               ▼              ▼                  ▼                 ▼
  Dev sandbox    Staging env    Staging QA       Production QA      Live site
```

---

## 2. React PWA Deployment (slt-sfcc-react)

### Where It Deploys

- **Salesforce Managed Runtime (MRT)** with CloudFlare CDN

### Deployment Workflow

```
Developer works locally
  │
  ├── npm run dev          (local dev with proxy + SSR)
  ├── npm run test         (Jest unit tests)
  ├── npm run lint         (ESLint)
  ├── npm run format       (Prettier)
  └── npm run compile      (TypeScript type check)
  │
  ▼
Create PR → Code review → Merge
  │
  ▼
Build for production
  │  npm run build
  │  Outputs to build/ directory
  │
  ▼
Deploy to Managed Runtime
  │  npm run push
  │  (requires credentials via npm run save-credentials)
  │
  ▼
Validate deployment
  │  npm run lighthouse:staging    (Lighthouse performance audit)
  │  npm run lighthouse:prod       (production audit)
  │
  ▼
CDN routing via pwa_mrtrules.json
  Routes: /pwa, /product/, /checkout, /account, /recipes, etc.
```

### Key Build Scripts

| Script | Purpose |
|--------|---------|
| `npm run dev:init` | First-time setup (install + translations + sync) |
| `npm run build` | Production Webpack build |
| `npm run push` | Build + deploy to MRT |
| `npm run analyze-build` | Bundle size analysis |
| `npm run theme` | Regenerate Chakra design tokens |
| `npm run storybook` | Component documentation (port 6006) |
| `npm run build-translations` | Extract + compile i18n messages |

---

## 3. BFF Deployment (slt-sfcc-bff)

### Where It Deploys

- **Node.js service** (direct) or **AWS Lambda**

### Deployment Workflow

```
Developer works locally
  │
  ├── npm run start:dev         (watch mode, skip SSM)
  ├── npm run start:dev:ssm     (with AWS Parameter Store)
  ├── npm run test              (Jest unit tests)
  ├── npm run test:it           (integration tests)
  └── npm run lint              (ESLint)
  │
  ▼
Create PR → Code review → Merge
  │
  ▼
Build for deployment target:
  │
  ├── Node.js Service:
  │     npm run build
  │     npm run start:prod
  │
  └── AWS Lambda:
        npm run build-lambda    (Webpack bundle for Lambda)
        npm run package         (package.sh creates deployment artifact)
  │
  ▼
Environment configuration loaded from:
  ├── .env.develop / .env.staging / .env.production
  └── AWS Systems Manager Parameter Store
        Path: /tf-test2/slt-sfcc-bff-dev (dev example)
```

### Key Build Scripts

| Script | Purpose |
|--------|---------|
| `npm run start:dev` | Dev with file watching + SKIP_SSM |
| `npm run start:debug` | Debug mode (port 9229) |
| `npm run build` | Standard NestJS build |
| `npm run build-lambda` | Webpack Lambda build |
| `npm run package` | Create deployment package |
| `npm run svs:generate` | Generate SVS SOAP client from WSDL |

---

## 4. SFCC Backend Deployment (slt-sfcc)

### Where It Deploys

- **SFCC instances** via WebDAV upload

### Deployment Workflow

```
Developer works locally
  │
  ├── npm run build           (Gulp + JS/CSS compilation)
  ├── npm run build:watch     (watch mode for dev)
  ├── npm run test:unit       (Mocha/Chai unit tests)
  └── npm run lint            (ESLint)
  │
  ▼
Create PR → Code review → Merge
  │
  ▼
Build artifacts:
  │  npm run build
  │  ├── npm run css          (SCSS → CSS)
  │  ├── npm run js           (Browserify bundling)
  │  └── npm run libs         (third-party library bundle)
  │
  ▼
Deploy to SFCC via WebDAV
  │  Upload cartridges to sandbox/instance
  │
  ▼
Configure cartridge path in SFCC BM
  │  BM > Administration > Sites > Manage Sites > Cartridges
  │
  ▼
Replicate code across environments
     BM > Administration > Data Replication
```

---

## 5. Environment Variable Management

### BFF Environment Configuration

```
Local Development:
  │  .env.develop (checked in, defaults)
  │  .env.local   (git-ignored, credentials)
  │  SKIP_SSM=true
  │
  ▼
Staging / Production:
  │  AWS Systems Manager Parameter Store
  │  Path: /tf-{env}/slt-sfcc-bff-{env}
  │  Loaded via AppConfigService at startup
  │
  ▼
Key Configuration Groups:
  ├── SFCC (URL, credentials, Commerce API settings)
  ├── Bloomreach (auth key, account ID, API URLs)
  ├── Emarsys (user, password)
  ├── Server (port, log level, HTTPS, Swagger)
  └── Feature flags and service toggles
```

### React App Environment Configuration

```
Local:
  │  .env (base)
  │  .env.local (overrides, git-ignored)
  │
  ▼
Deployed (MRT):
  │  .env.develop / .env.production
  │  MRT environment variables
  │
  ▼
Key Variables:
  ├── REACT_APP_BFF_URL (BFF API endpoint)
  ├── MRT_ENVIRONMENT
  ├── EXTERNAL_DOMAIN_NAME
  └── COMMERCE_API_* (SFCC Commerce API settings)
```

---

## 6. SFCC Sandbox Setup

### Initial Configuration Steps

```
New sandbox setup:
  │
  ├── 1. Import system preferences into sandbox
  │
  ├── 2. Activate PWA Plugin:
  │       BM > Administration > Sites > Manage Sites
  │       Add plugin_slas to Cartridges field
  │
  ├── 3. Configure SLAS Plugin:
  │       BM > Merchant Tools > Site Preferences > SLAS Plugin
  │       ├── redirectURI_SLAS = http://localhost:8000/callback
  │       ├── use_SLAS_session_bridge = Yes
  │       └── ocapiSessionBridgeURI_SLAS = https://bcjl-[ID].dx.commercecloud.salesforce.com/...
  │
  ├── 4. Import SFCC services:
  │       Edit sfcc-slas-services.xml → Import via BM
  │
  └── 5. Import slt-sfcc-configs for environment:
          BM > Administration > Operations > Import/Export
```

---

## Related Diagrams

- [Configuration & Deployment Flow](04-configuration-deployment.drawio)
- [Deployment Diagram - Environments](../../draw.io/06-deployment-diagram-environments.drawio) (existing)

---

## Systems Involved

| System | Role |
|--------|------|
| **GitHub** | Source control, PRs, CODEOWNERS review |
| **SFCC Business Manager** | Config import, cartridge management, replication |
| **Salesforce Managed Runtime** | React PWA hosting and CDN |
| **AWS Lambda** | BFF serverless deployment |
| **AWS Systems Manager** | Parameter Store for BFF config |
| **CloudFlare CDN** | Edge caching and routing |
