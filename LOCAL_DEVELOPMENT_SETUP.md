# Local Development Setup Guide

This guide covers how to set up and run the BFF (Backend-For-Frontend) and React PWA storefront for local development.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Quick Start](#quick-start)
- [BFF Setup](#bff-setup)
- [React App Setup](#react-app-setup)
- [Running Together](#running-together)
- [Environment Configuration](#environment-configuration)
- [Debugging](#debugging)
- [Testing](#testing)
- [Troubleshooting](#troubleshooting)

---

## Prerequisites

### Required Software

- **Node.js:**
  - BFF: >=22.18.0
  - React App: ^22.0.0
- **npm:** >=10.0.0
- **Git:** For cloning and version control
- **VSCode** (optional): Recommended for debugging

### Verify Installation

```bash
node --version    # Should be v22.x.x or higher
npm --version     # Should be v10.x.x or higher
```

### SFCC Configuration (Required for BFF)

Before running the BFF, your Salesforce Commerce Cloud sandbox must be configured:

1. **Import system preferences** into your sandbox
2. **Activate the PWA Plugin:**
   - Go to **Administration > Sites > Manage Sites > Sur La Table Webstore > Settings**
   - Add `plugin_slas` to the **Cartridges** field

3. **Configure SLAS Plugin:**
   - Navigate to **Merchant Tools > Site Preferences > Custom Preferences > Group: SLAS Plugin**
   - Set the following:
     - `redirectURI_SLAS`: `http://localhost:8000/callback`
     - `use_SLAS_session_bridge`: `Yes`
     - `ocapiSessionBridgeURI_SLAS`: `https://bcjl-[SANDBOX_ID].dx.commercecloud.salesforce.com/s/SLT/dw/shop/v21_3/sessions`

4. **Import SFCC configuration:**
   - Edit the `sfcc-slas-services.xml` file (follow comments in the file)
   - Import it into SFCC: **Administration > Operations > Import/Export**

---

## Quick Start

### One-Command Setup (First Time)

From the project root, run:

```bash
# Setup React app dependencies and translations
cd slt-sfcc-react
npm run dev:init

# In another terminal, setup BFF with utilities
cd slt-sfcc-bff
npm install
```

Then run both services:

```bash
# Terminal 1 - React App with Proxy
cd slt-sfcc-react
npm run dev

# Terminal 2 - BFF API Server
cd slt-sfcc-bff
npm run start:dev
```

Access the app at: **http://localhost:8000/pwa**

---

## BFF Setup

### 1. Navigate to the BFF Directory

```bash
cd slt-sfcc-bff
```

### 2. Install Dependencies

```bash
npm install
```

### 3. Configure Environment Variables

The BFF requires environment configuration. Two approaches:

#### Option A: Use Development Defaults (Skip SSM)

For local development without AWS Systems Manager Parameter Store:

```bash
npm run start:dev
```

This runs with `SKIP_SSM=true`, so the BFF uses the `.env.develop` file directly.

#### Option B: Use AWS SSM (Advanced)

If you have AWS credentials configured:

```bash
npm run start:dev:ssm
```

This loads configuration from AWS Parameter Store at path `/tf-test2/slt-sfcc-bff-dev`.

### 4. Configure .env File

Copy or update the `.env.develop` file with your settings. Key variables:

**SFCC Configuration:**
```env
SFCC_URL=https://dev.surlatable.com
SFCC_USER=storefront
SFCC_PASSWORD=<your-password>
```

**Commerce API:**
```env
COMMERCE_API_ENV_NAME=dev
COMMERCE_API_SITE_ID=SLT
COMMERCE_API_ORG_ID=f_ecom_bcjl_dev
COMMERCE_API_SHORT_CODE=xveiy6nj
COMMERCE_API_HOST=dev.surlatable.com
COMMERCE_API_CLIENT_ID=5a8cf24c-1d25-4736-ac00-fc8b1f08727e
```

**Bloomreach Search:**
```env
SLT_BR_PARAM_VALUE_AUTH_KEY=<your-key>
SLT_BR_PARAM_VALUE_ACCOUNT_ID=<your-account>
SLT_BR_URL_SEARCH=https://staging-core.dxpapi.com/api/v1/core/
SLT_BR_URL_SUGGEST=https://staging-suggest.dxpapi.com/api/v2/suggest/
SLT_BR_URL_RECOMMEND=https://staging-recommend.dxpapi.com/api/v2/widgets/
```

**Emarsys Marketing:**
```env
EMARSYS_USER=sur_la_table011
EMARSYS_PASSWORD=<your-password>
```

**Local Development Settings:**
```env
LOG_LEVEL=info
ACTIVATE_SWAGGER=true
SWAGGER_SERVER_NAME=local
SWAGGER_SERVER_URL=http://localhost:1337
```

### 5. Start the BFF Server

```bash
# Development with file watching and pretty logs
npm run start:dev

# With source maps and debugging
npm run start:debug

# Production mode
npm run start:prod
```

The BFF will start on **http://localhost:1337** (or the port specified in `.env`).

### 6. Verify BFF is Running

```bash
# Health check
curl http://localhost:1337/health

# API Documentation (if ACTIVATE_SWAGGER=true)
# Visit: http://localhost:1337/api-doc
```

---

## React App Setup

### 1. Navigate to the React Directory

```bash
cd slt-sfcc-react
```

### 2. Install Dependencies and Build Translations

First-time setup (includes translation compilation):

```bash
npm run dev:init
```

This runs:
- `npm install` - Install dependencies
- `npm run build-translations` - Extract and compile translations
- `npm run compile-translations:pseudo` - Create pseudo-locale for testing
- `node scripts/sync-agent-commands.mjs` - Sync AI assistant commands

Subsequent times, just install new packages:

```bash
npm install
```

### 3. Configure Environment Variables

The React app can use environment variables from `.env` or `.env.local`. Common variables:

```env
# BFF API Configuration
REACT_APP_BFF_URL=http://localhost:1337

# PWA Kit Configuration
MRT_ENVIRONMENT=development
EXTERNAL_DOMAIN_NAME=localhost:8000

# SFCC Commerce API
COMMERCE_API_URL=https://dev.surlatable.com
COMMERCE_API_CLIENT_ID=5a8cf24c-1d25-4736-ac00-fc8b1f08727e
```

### 4. Start Development Server

#### Option A: Full Development (Recommended)

Runs both the proxy server and SSR development server:

```bash
npm run dev
```

This starts:
- Proxy server on **http://localhost:3000** (compiling TypeScript)
- SSR server on **http://localhost:8000**

#### Option B: Without Proxy

Direct development without local proxy:

```bash
npm run dev:noproxy
```

Runs only the SSR server on **http://localhost:8000**.

#### Option C: With Babel Caching

For faster builds with Babel:

```bash
npm run dev:babel
```

Or on macOS with the shell script:

```bash
npm run dev:temp:macos
```

### 5. Access the App

Open your browser and navigate to:

```
http://localhost:8000/pwa
```

The app supports hot module replacement (HMR), so changes will auto-refresh in the browser.

### 6. Build for Production

When you're ready to build:

```bash
npm run build
```

Outputs to the `build/` directory.

---

## Running Together

### Setup Multiple Terminals

Running both services requires 2-3 terminal windows:

**Terminal 1: React App**
```bash
cd slt-sfcc-react
npm run dev
```

**Terminal 2: BFF API**
```bash
cd slt-sfcc-bff
npm run start:dev
```

**Terminal 3 (Optional): Testing/Additional Commands**
```bash
# Run tests, format code, lint, etc.
```

### Verify Both Are Running

```bash
# React App (check for 200 response)
curl http://localhost:8000/pwa

# BFF Health Check
curl http://localhost:1337/health

# BFF Swagger API Docs
curl http://localhost:1337/api-doc
```

### Access the Storefront

1. Open browser: **http://localhost:8000/pwa**
2. The React app communicates with BFF at **http://localhost:1337**
3. The BFF communicates with SFCC backend

---

## Environment Configuration

### Environment Files

#### React App
- **`.env`** - Base environment variables
- **`.env.local`** - Local overrides (git-ignored)
- **`.env.develop`** - Development configuration
- **`.env.production`** - Production configuration

#### BFF
- **`.env.develop`** - Development configuration (main file)
- **`.env.staging`** - Staging configuration
- **`.env.production`** - Production configuration
- **`.env.*-test`** - Test environments

### Important Variables

#### React App

| Variable | Purpose | Example |
|----------|---------|---------|
| `MRT_ENVIRONMENT` | PWA Kit environment | `development` |
| `EXTERNAL_DOMAIN_NAME` | Your domain | `localhost:8000` |
| `REACT_APP_BFF_URL` | BFF API URL | `http://localhost:1337` |

#### BFF

| Variable | Purpose | Example |
|----------|---------|---------|
| `PORT` | Server port | `1337` |
| `LOG_LEVEL` | Logging level | `info`, `debug` |
| `SKIP_SSM` | Skip AWS Parameter Store | `true` for local dev |
| `ACTIVATE_SWAGGER` | Enable Swagger docs | `true` |
| `USE_HTTPS` | Use HTTPS | `false` for local |

### Sensitive Configuration

For credentials and keys (not in git):

1. Create `.env.local` in each project
2. Add credentials there (git-ignored by default)
3. Never commit these files

```env
# .env.local example
SFCC_PASSWORD=your-actual-password
SLT_BR_PARAM_VALUE_AUTH_KEY=your-actual-key
EMARSYS_PASSWORD=your-actual-password
```

---

## Debugging

### BFF Debugging

#### VSCode Setup

Create `.vscode/launch.json`:

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Debug BFF",
      "runtimeExecutable": "npm",
      "runtimeArgs": ["run", "start:debug"],
      "console": "integratedTerminal",
      "internalConsoleOptions": "neverOpen"
    }
  ]
}
```

Then press `F5` or **Run > Start Debugging** in VSCode.

#### Command Line Debugging

```bash
npm run start:debug
```

The debugger will listen on `localhost:9229`.

### React App Debugging

#### Browser DevTools

1. Open the app: `http://localhost:8000/pwa`
2. Press `F12` to open Developer Tools
3. Use React DevTools browser extension for component inspection

#### VSCode Chrome Debugger

Install the "Debugger for Chrome" extension, then create `.vscode/launch.json`:

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "chrome",
      "request": "launch",
      "name": "Launch Chrome",
      "url": "http://localhost:8000/pwa",
      "webRoot": "${workspaceFolder}/slt-sfcc-react",
      "sourceMaps": true
    }
  ]
}
```

### Viewing Logs

#### BFF Logs (Pretty Format)

The `npm run start:dev` command uses `pino-pretty` for readable logs:

```bash
npm run start:dev

# Output example:
[1234] INFO  (NestFactory) Starting Nest application...
[1234] INFO  (InstanceLoader) AppModule dependencies initialized
```

#### React App Logs

Check browser console:
1. Open DevTools (`F12`)
2. Go to **Console** tab
3. Look for any errors or warnings

### Environment Variables in Development

#### React App

Access in code:
```ts
const bffUrl = process.env.REACT_APP_BFF_URL;
```

#### BFF

Access via `AppConfigService`:
```ts
constructor(appConfig: AppConfigService) {
  const { sfcc } = appConfig.getConfig();
  console.log(sfcc.url);
}
```

---

## Testing

### React App Testing

```bash
cd slt-sfcc-react

# Run all tests
npm run test

# Watch mode (reruns on file changes)
npm run test:watch

# Run tests for changed files only
npm run test:changed

# Generate coverage report
npm run test:ci

# Integration tests
npm run test:integration

# Performance tests
npm run test:profile

# Lighthouse performance testing
npm run test:lighthouse
```

### BFF Testing

```bash
cd slt-sfcc-bff

# Unit tests
npm run test

# Watch mode
npm run test:watch

# Integration tests
npm run test:it

# E2E tests
npm run test:e2e

# Coverage report
npm run test:cov

# Debug tests
npm run test:debug
```

### Linting and Formatting

#### React App

```bash
# Check for lint errors
npm run lint

# Fix lint errors
npm run lint:fix

# Check code format
npm run format

# Fix code format
npm run format:write
```

#### BFF

```bash
# Check for lint errors
npm run lint

# Fix lint errors
npm run lint:fix

# Check code format
npm run format

# Fix code format
npm run format:write
```

### Type Checking

#### React App

```bash
npm run compile
```

#### BFF

Type checking is done during the build, or you can use:

```bash
npm run build
```

---

## Troubleshooting

### Common Issues

#### 1. Node Version Mismatch

**Problem:** `npm ERR! The engines in this package.json will only execute in Node versions matching: >=22.18.0`

**Solution:**
```bash
# Check your Node version
node --version

# Update Node (use nvm or Homebrew)
nvm install 22
nvm use 22

# Or download from nodejs.org
```

#### 2. Port Already in Use

**Problem:** `Error: listen EADDRINUSE: address already in use :::8000`

**Solution:**
```bash
# Find and kill the process using the port
# macOS/Linux:
lsof -i :8000
kill -9 <PID>

# Or change the port in .env
PORT=8001
```

#### 3. BFF Connection Refused

**Problem:** React app can't connect to BFF at `http://localhost:1337`

**Solution:**
1. Verify BFF is running: `curl http://localhost:1337/health`
2. Check REACT_APP_BFF_URL in `.env`
3. Check firewall settings
4. Verify BFF PORT is 1337 in `.env`

#### 4. Environment Variables Not Loading

**Problem:** Changes to `.env` file not reflected

**Solution:**
1. Restart the dev server
2. Check `.env.local` isn't overriding with old values
3. Verify variable names match (case-sensitive)

#### 5. Translation Compilation Error

**Problem:** `Error: ENOENT: no such file or directory "compiled-translations"`

**Solution:**
```bash
cd slt-sfcc-react

# Rebuild translations
npm run build-translations
npm run compile-translations
npm run compile-translations:pseudo
```

#### 6. SFCC Authentication Failing

**Problem:** BFF returns 401 Unauthorized from SFCC

**Solution:**
1. Verify SFCC credentials in `.env`:
   - `SFCC_URL`
   - `SFCC_USER`
   - `SFCC_PASSWORD`
2. Check sandbox is accessible
3. Verify SLAS plugin is configured (see Prerequisites)
4. Check SLAS callback URL is correct

#### 7. Swagger UI Not Loading

**Problem:** `http://localhost:1337/api-doc` returns 404

**Solution:**
1. Verify `ACTIVATE_SWAGGER=true` in `.env`
2. Restart BFF server
3. Clear browser cache

#### 8. Hot Module Replacement (HMR) Not Working

**Problem:** Changes don't auto-refresh in browser

**Solution:**
1. Check proxy server is running: `http://localhost:3000`
2. Check browser console for errors
3. Try hard refresh (`Ctrl+Shift+R` or `Cmd+Shift+R`)
4. Restart development server

#### 9. Dependencies Conflicting

**Problem:** `npm ERR! peer dep missing`

**Solution:**
```bash
# Clear cache and reinstall
rm -rf node_modules package-lock.json
npm install

# Or use npm's force flag
npm install --force
```

#### 10. Storybook Not Starting

**Problem:** `npm run storybook` fails

**Solution:**
```bash
cd slt-sfcc-react

# Clear cache and rebuild
rm -rf node_modules/.cache
npm run build-storybook
npm run storybook
```

### Debug Checklist

Before filing an issue, try:

- [ ] Check Node.js version: `node --version`
- [ ] Check npm version: `npm --version`
- [ ] Clear node_modules: `rm -rf node_modules && npm install`
- [ ] Check `.env` files for typos
- [ ] Restart both servers
- [ ] Check firewall allowing localhost connections
- [ ] Review error logs in terminal
- [ ] Check browser console for errors
- [ ] Verify internet connection (for external API calls)

### Getting Help

If you still have issues:

1. Check the project's GitHub issues
2. Review the detailed documentation:
   - React App: `slt-sfcc-react/README.md`
   - BFF: `slt-sfcc-bff/README.md`
3. Check environment-specific documentation:
   - Salesforce PWA Kit: https://developer.salesforce.com/docs/commerce/pwa-kit-managed-runtime/overview
   - NestJS: https://docs.nestjs.com/

---

## Next Steps

After setup, you're ready to:

1. **Explore the codebase** - See `CODEBASE_OVERVIEW.md`
2. **Run tests** - Verify everything works
3. **Start developing** - Make changes and see them live
4. **Read component docs** - Run `npm run storybook` in React app
5. **Check API docs** - Visit `http://localhost:1337/api-doc` for BFF

Happy coding!
