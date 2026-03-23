# Technical Analysis: EU VTP (Vehicle Transaction Pages)

**Version:** 1.0  
**Status:** Draft  
**Date:** 2026-03-22  
**Source:** Code archaeology of `source-repos/audi-eu-vtp/` monorepo  

---

## 1. Architecture Overview

### Monorepo Structure

The EU VTP is an npm workspaces monorepo (`"workspaces": ["packages/*"]`) containing 8 packages that collectively deliver the vehicle shopping experience across European Audi markets.

```
audi-eu-vtp/
├── packages/
│   ├── plp/                    # Product Listing Page Feature App (v4.19.1)
│   ├── pdp/                    # Product Details Page Feature App (v4.30.1)
│   ├── configuration/          # Headless Configuration Feature App (v5.14.0)
│   ├── configuration-service/  # Feature Service library (v0.0.1)
│   ├── dealer-info/            # Dealer Info Feature App (v1.17.0)
│   ├── soldout/                # 404 Soldout Feature App (v1.1.0)
│   ├── shared/                 # Shared library — components, hooks, utils (v2.7.0)
│   ├── template-server/        # Local dev E2E template server
│   └── semantic-release-vtp-analyzer/  # Custom semantic-release analyzer
├── shared/                     # Top-level dev infra (webpack config, demo setup)
├── content-overrides/          # Chrome DevTools content override target (gitignored)
├── cypress/                    # E2E test suite
└── test/                       # Shared test setup
```

### Package Relationships

The packages form a layered dependency graph with clear separation between Feature Apps (deployed independently), Feature Services (shared state), and shared libraries:

```
┌──────────────────────────────┐
│  AEM Content Authors         │
│  (Content Fragments)         │
└─────────────┬────────────────┘
              │ content delivery via audi-content-service
              ▼
┌──────────────────────────────┐
│  fa-vtp-configuration        │  Headless FA — reads content, maps to VTPConfiguration
│  (renders nothing)           │
└─────────────┬────────────────┘
              │ setConfiguration() — pub/sub broadcast
              ▼
┌──────────────────────────────┐
│  vtp-configuration-service   │  Feature Service — reactive config store
└──┬───────────┬───────────┬───┘
   │           │           │ subscribeConfiguration()
   ▼           ▼           ▼
┌──────┐  ┌────────┐  ┌──────────┐  ┌──────────┐
│ PLP  │  │  PDP   │  │dealer-info│  │ soldout  │
│  FA  │  │  FA    │  │    FA     │  │   FA     │
└──┬───┘  └───┬────┘  └────┬─────┘  └──────────┘
   │          │             │
   └────┬─────┘─────────────┘
        │ imports (compile-time dependency)
        ▼
┌──────────────────────────────┐
│  vtp-shared                  │  Library — components, hooks, types, utils, contexts
└──────────────────────────────┘
```

### Microfrontend Composition Model

On a live AEM page, multiple Feature Apps coexist and communicate through the Feature Hub:

| Page | Feature Apps Present |
|------|---------------------|
| **Vehicle Listing Page** | `fa-vtp-configuration` + `fa-vtp-plp` |
| **Vehicle Detail Page** | `fa-vtp-configuration` + `fa-vtp-pdp` + `fa-vtp-dealer-info` |
| **Sold-Out Page** | `fa-vtp-soldout` (+ potentially other page-level FAs) |

Each Feature App is independently deployed to the **App Store** and included on AEM pages via the "Generic Feature App Include" component. The Feature Hub manages their lifecycle, dependency injection, and inter-app communication.

### Key Architectural Patterns

1. **Headless Configuration FA:** The configuration FA renders `null` — its sole job is to read AEM content and broadcast structured config to all other VTP FAs via the configuration service.
2. **Singleton React Contexts via `Symbol.for()`:** `ServicesContext` and `StockContext` use `Symbol.for('__VTP_SERVICES_CONTEXT__')` / `Symbol.for('__VTP_STOCK_CONTEXT__')` to ensure a single context instance across multiple Feature Apps that may share the same React runtime.
3. **Dual Design Systems:** Every visual FA builds four bundles — CSR classic, CSR alpha, SSR classic, SSR alpha — to support both the legacy ("unified") and new ("alpha") Audi design systems.
4. **Content-Driven Feature Toggling:** All feature flags are managed through AEM Content Fragments rather than a runtime feature flag service (no LaunchDarkly detected).

---

## 2. Technology Stack

| Layer | Technology | Version / Notes |
|-------|-----------|-----------------|
| **Language** | TypeScript | ^5.4.5 |
| **UI Framework** | React | ^18.2.0 (compatibility with 16/17) |
| **State Management** | Redux (`react-redux` ^8.0.2, `redux` ^4.1.2) | Vehicle store via `@oneaudi/stck-store` (3.4.2) |
| **Styling** | styled-components | ^5 |
| **Design System (Classic)** | `@oneaudi/unified-web-components` (^1.45.0), `@oneaudi/unified-web-common` (^1.13.0) | Legacy design |
| **Design System (Alpha)** | `@oneaudi/unified-web-alpha-components` (1.22.0), `@oneaudi/unified-web-alpha-common` (1.11.0) | New design |
| **Microfrontend Orchestration** | Feature Hub (`@feature-hub/react` ^3.6.0) | Manages FA lifecycle, DI, shared services |
| **CMS** | Adobe Experience Manager (AEM) Headless | Content Fragment Models for structured data |
| **GraphQL** | `@oneaudi/onegraph-client` (^4.9.2) | OneGraph for carline data; Omnigraph for wishlist |
| **API Client** | Custom `fetchAnythingScs()` with token auth | REST calls to SCS API |
| **Build** | webpack + `@oneaudi/oneaudi-os-build-scripts` (^7.1.2) | SWC for transpilation |
| **Bundler Transpiler** | SWC (`@swc/cli` ^0.6.0, `@swc/jest` ^0.2.37) | Fast transpilation |
| **Testing (Unit)** | Jest 29.7.0 + React Testing Library (^14.3.1) | `jest-environment-jsdom` |
| **Testing (E2E)** | Cypress with Cucumber preprocessor | Monorepo-level E2E suite |
| **Performance** | Lighthouse CI | Accessibility ≥ 0.3 (error), Performance ≥ 0.75 (warn) |
| **Package Manager** | npm workspaces (yarn configured in oneaudi-cli) | Monorepo workspace management |
| **Node Runtime** | v18.20.4 | |
| **CI/CD** | Semantic release with custom VTP analyzer | Conventional commits enforced |
| **Infrastructure** | AWS CDK (`@oneaudi/oneaudi-os-infrastructure` ^10.0.0) | Deployed to `arcade.apps.one.audi` |
| **Maps** | Google Maps API (`@googlemaps/react-wrapper` ^1.1.42) | Location filter, dealer maps |
| **HTML Sanitization** | DOMPurify (^3.2.4) | Sanitizing dealer comments, HTML content |
| **HTML Parsing** | `html-react-parser` (^3.0.1) | Rendering HTML content in React |
| **Auth** | `@oneaudi/audi-auth-service` (^6.1.2) | MyAudi authentication |

---

## 3. Data Flow

### Vehicle Data Flow

```
┌─────────────────────┐
│  SCS API             │  Stock Car Service — the single source of vehicle data
│  (REST)              │
└──────────┬──────────┘
           │ fetchAnythingScs() / fetchVehicleRaw()
           │ Token auth via apiKey header
           ▼
┌─────────────────────────────────────────┐
│  vtp-shared: FeatureAppInitialization   │
│  - SSR: fetch → serialize state         │
│  - CSR after SSR: deserialize state     │
│  - CSR only: fetch client-side          │
└──────────┬──────────────────────────────┘
           │
           ▼
┌─────────────────────┐     ┌─────────────────────────┐
│  StockContext        │     │  Redux Store (stck-store) │
│  (PDP — single car)  │     │  (PLP — vehicle list)     │
│  CompleteVehicleEntry │     │  vehicles, filters, sort, │
└──────────┬──────────┘     │  favorites, UI state       │
           │                 └──────────┬────────────────┘
           ▼                            ▼
    PDP Components              PLP Tile Components
```

**PLP Data Flow:**
1. On initialization, PLP fetches filters and vehicle results from SCS API via `entryFilters()`
2. Results populate the Redux store (`@oneaudi/stck-store`) as a vehicle map
3. Components read from Redux via selectors (`SELECTORS.UI.getLoadingState`, vehicle arrays, etc.)
4. Filter changes trigger new SCS API calls → Redux store updates → component re-renders
5. Pagination via "Load More" fetches 12 more vehicles, appending to the store

**PDP Data Flow:**
1. Vehicle ID extracted from URL (supports `sc_detail` param, SEO paths with `/id…/`, `?vehicleId=` param)
2. `initializeFeatureApp()` fetches complete vehicle data (`basic` + `detail`) from SCS API
3. Data provided via `StockContextProvider` to all PDP components
4. Vehicle 404 → redirect to `notFound` URL (301 status) or return 404

### Configuration Data Flow

```
┌──────────────────────┐
│  AEM Content Authors  │
│  Content Fragment     │
│  Editor / Universal   │
│  Editor               │
└──────────┬───────────┘
           │ structured content (flat field keys)
           ▼
┌──────────────────────────────────┐
│  audi-content-service            │  Feature Service — delivers CF data
└──────────┬───────────────────────┘
           │ getContent()
           ▼
┌──────────────────────────────────┐
│  fa-vtp-configuration            │
│  mapContent() transforms flat    │
│  AEM fields → nested             │
│  VTPConfiguration object         │
│  e.g., scopes_financeEnabled →   │
│       { scopes: {                │
│           financeEnabled: true } │
│       }                          │
└──────────┬───────────────────────┘
           │ configService.setConfiguration()
           ▼
┌──────────────────────────────────┐
│  vtp-configuration-service       │
│  Pub/Sub store — notifies all    │
│  subscribers immediately         │
│  SSR: serializes state for       │
│  hydration                       │
└──────────┬───────────────────────┘
           │ subscribeConfiguration(callback)
           ▼
    PLP / PDP / dealer-info
    read VTPConfiguration for:
    - CTA definitions, finance settings
    - Sort options, consumption display
    - Feature toggles (scopes)
    - Price configuration
    - External URLs
```

### Finance Data Flow

```
SCS API (vehicle.basic.financing) → FinanceProvider context
                                     ↓
CRS API (FSAG products + default calculation) → useDynamicFinancing hook
                                     ↓
                              Session storage persistence (per vehicle type NC/UC)
                                     ↓
              Finance components (rate display, change rate, breakdown)
```

---

## 4. Shared Infrastructure (`@oneaudi/vtp-shared`)

The shared package is the largest and most critical dependency. It provides the foundation consumed by all VTP Feature Apps.

### Components Layer

| Category | Key Components | Purpose |
|----------|---------------|---------|
| **CTA System** | `CTAButtons`, `GenericButton`, `CTA` (POST form) | Configurable action buttons with 18+ CTA types, dealer filtering, POST form submission |
| **Finance (Legacy)** | `Finance`, `PriceInformation`, `LeasingInformation`, `TaxationInformation` | Market-specific price templates (Japan, Spain, default) |
| **Finance (Rethink)** | `PriceInformationRethink`, `RateInformationRethink`, `FinanceLayerRethink`, `PriceBreakdownCta` | Redesigned finance components for PDP |
| **Consumption/Emission** | `CEE`, `ENVKV`, `ConsumptionTileElement`, `EfficiencyClassElement` | Vehicle environmental data display with German regulation compliance |
| **Vehicle Info** | `CarinfoWrapper`, `AvailableBadge`, `VehicleOrderStatus` | Vehicle headline, availability, delivery status |
| **Filter System (Rethink)** | `FilterOverlay`, `CheckboxFilter`, `RangeFilter`, `ModelFilter`, `LocationFilter`, `EquipmentFilter`, `FilterChips`, `DealersList` | Complete filter UI with ~40 component files |
| **Auth/Wishlist** | `MyAudiWishlistLoginLayer`, `AuthContextProvider`, `MyAudiWishlistContextProvider` | Authentication gating, cloud-synced favorites |
| **UI Primitives** | `LazyLoad`, `LazyImage`, `ImageSlider`, `ExpandableElement`, `FadeInAnimation`, `Slider` | Shared UI patterns |
| **Maps** | `MapWrapper`, `CookieConsentControl`, `RenderedMarker` | Google Maps with GDPR consent |

### Hooks Layer

| Category | Key Hooks | Purpose |
|----------|----------|---------|
| **API** | `useVehicleRaw`, `useChainDealers`, `useVehicle`, `useScsUrlParts` | Vehicle/dealer data fetching from SCS |
| **Formatting** | `useFormattedPrice`, `useFormattedDate`, `useConsumptionLabels`, `useEmissionLabels`, `useDynamicAltText` | Locale-aware formatting |
| **Finance** | `useDynamicFinancing`, `usePriceConfiguration` | Finance calculations, price template resolution |
| **DOM** | `useBodyScrollLock`, `useInView`, `useDesktopOrMobileView`, `useShareUrl` | Browser interaction utilities |
| **SSR** | `useClientServerUtils` | SSR/CSR detection |

### Context Layer

| Context | Pattern | Purpose |
|---------|---------|---------|
| `ServicesContext` | `Symbol.for()` singleton | Root service provider (config, env, store access) |
| `StockContext` | `Symbol.for()` singleton | Vehicle data for PDP |
| `AuthContext` | Standard React context | Authentication state management |
| `MyAudiWishlistContext` | Standard React context | Wishlist CRUD via Omnigraph GraphQL |
| `FilterContext` | Standard React context | Complete PLP filter state |
| `FinanceContext` | Standard React context | Dynamic finance state |

### Utilities Layer

| Utility | Purpose |
|---------|---------|
| `fetchAnythingScs()` | Core SCS API client with token auth |
| `formatUrl()` | URL template replacement with 20+ vehicle data placeholders |
| `getImageUrl()`, `vtpImageUrlAdaption()` | Image URL adaptation across vtpimages.audi.de/com and mediaservice domains |
| `getVehicleIdFromUrl()`, `generatePdpUrl()` | Vehicle ID extraction and SEO URL generation |
| `displayByDealerId()` | CTA visibility filtering by dealer include/exclude lists |
| `getConsumptionLabels()`, `getEmissionLabels()` | Consumption/emission label generation with PHEV/multi-fuel support |
| `getCashCheckout()`, `getLeanCheckoutPayload()` | E-commerce checkout payload assembly |
| `isCampaignActive()` | Campaign date range validation |
| `DOMPurify` sanitization | HTML content sanitization for dealer remarks |

### Feature App Initialization

`FeatureAppInitialization.tsx` provides shared initialization logic with three paths:

1. **SSR:** Schedule async rerender → fetch vehicle data → serialize state for hydration
2. **CSR after SSR:** Deserialize state from SSR → skip re-fetch (instant render)
3. **CSR only:** Fetch everything client-side

Error handling: Vehicle 404 → redirect to `notFound` URL (301) or return 404 status. Non-404 errors → silently hide Feature App.

---

## 5. Integration Architecture

### Page-Level Feature App Composition

```
┌─────────────────────────────────────────────────────────────┐
│  AEM Page (rendered by Renderman SSR proxy)                 │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  Feature Hub                                         │    │
│  │                                                      │    │
│  │  ┌───────────────────────┐                           │    │
│  │  │ fa-vtp-configuration  │ ← Headless, publishes     │    │
│  │  │ (renders null)        │   VTPConfiguration         │    │
│  │  └───────────┬───────────┘                           │    │
│  │              │ setConfiguration()                     │    │
│  │              ▼                                        │    │
│  │  ┌──────────────────────┐                            │    │
│  │  │ vtp-configuration-   │ ← Shared state bus         │    │
│  │  │ service (FeatureSvc) │                            │    │
│  │  └──┬────────┬──────┬───┘                            │    │
│  │     │        │      │                                │    │
│  │     ▼        ▼      ▼                                │    │
│  │  ┌──────┐ ┌─────┐ ┌───────────┐                     │    │
│  │  │ PLP  │ │ PDP │ │dealer-info│                     │    │
│  │  └──────┘ └─────┘ └───────────┘                     │    │
│  │                                                      │    │
│  │  PDP → loads partner-business-card FA via             │    │
│  │     FeatureAppLoader (nested FA)                      │    │
│  │                                                      │    │
│  │  dealer-info → loads partner-business-card FA via     │    │
│  │     FeatureAppLoader (nested FA)                      │    │
│  └──────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

### Inter-App Communication

| Mechanism | Used Between | Purpose |
|-----------|-------------|---------|
| **vtp-configuration-service** (pub/sub) | Configuration → PLP/PDP/dealer-info | Broadcast VTPConfiguration |
| **URL hash** | PLP → PDP (and back) | Filter state persistence (`#filter=…&preset=…`) |
| **Custom DOM events** | External → PDP | `pdp:open-warranties-tab` to programmatically open warranties tab |
| **localStorage** | PLP ↔ PDP | Selected vehicle for scroll-back, filter state, location search, myAudi wishlist pending add |
| **sessionStorage** | Within PDP | AOZ selected products, finance data |
| **Feature Services** (Feature Hub DI) | Feature Hub → all FAs | locale, i18n, auth, tracking, content, footnotes, notifications, layer management, e-commerce |
| **URL navigation** | PLP → PDP | PDP URL generated from configurable pattern with `SC_VEHICLE_ID` / `SC_VEHICLE` placeholders |

### Nested Feature App Loading

The `dealer-info` FA doesn't render dealer info directly — it delegates to the `fa-partner-business-card` Feature App, loaded dynamically via `FeatureAppLoader`. Configuration (partnerID, display toggles, dealer data) is passed as props. The PDP also spawns the Trade-In Teaser Feature App via `@oneaudi/falcon-tools`' `<Spawn>` component.

---

## 6. SSR Architecture

### Rendering Pipeline

```
Browser Request → AEM Page Skeleton
                       ↓
              Renderman (SSR Proxy)
                       ↓
              Feature Hub SSR Integrator
                       ↓
         ┌─────────────┼─────────────┐
         ▼             ▼             ▼
   FA SSR Bundle  FA SSR Bundle  FA SSR Bundle
   (app-ssr.js)   (app-ssr.js)   (app-ssr.js)
         │             │             │
         └─────────────┼─────────────┘
                       ↓
              Pre-rendered HTML markup
                       ↓
              Delivered to browser
                       ↓
              CSR hydration (app.js bundles)
```

### SSR Build Artifacts

Each visual Feature App produces four bundles:
- `fh/app.js` — CSR classic design
- `fh/app-ssr.js` — SSR classic design
- `fh/app-alpha.js` — CSR alpha design
- `fh/app-alpha-ssr.js` — SSR alpha design

### SSR Feature Services

| Service | ID | Role in SSR |
|---------|----|-------------|
| `s2:async-ssr-manager` | `@feature-hub/async-ssr-manager` | Schedules async rerenders during SSR |
| `s2:server-request` | `@feature-hub/server-request` | Provides HTTP request context (URL, headers) |
| `s2:serialized-state-manager` | `@feature-hub/serialized-state-manager` | Serializes state for CSR hydration |

### SSR Strategy by Package

| Package | SSR Approach |
|---------|-------------|
| **PLP** | SSR renders **skeleton placeholders** (not real vehicle data). `typeof window === 'undefined'` → skeleton. Vehicle data fetched client-side. |
| **PDP** | SSR fetches **real vehicle data**. Three-path initialization: SSR (fetch + serialize), CSR-after-SSR (deserialize), CSR-only (fetch). |
| **Configuration** | SSR serializes VTPConfiguration state for hydration. CSR-after-SSR deserializes without re-fetching content. |
| **Configuration Service** | Serializes/deserializes config state across SSR boundary. |
| **Dealer-info** | SSR support via optional SSR services. |
| **Soldout** | Sets 404 page info during SSR via `page-info-service`. |

### SSR Safety Patterns

- All `window`/`document` accesses guarded with `typeof window !== 'undefined'`
- Fallback dimensions for SSR (e.g., window width defaults to 400)
- Interactive elements gated by `isClient` check from `useClientServerUtils()`
- Redux loading state triggers skeleton display during hydration

---

## 7. Design System Architecture

### Dual Design System: Classic vs Alpha

The VTP supports two concurrent design systems to facilitate a gradual migration:

| Aspect | Classic ("Unified") | Alpha |
|--------|-------------------|-------|
| **Package** | `@oneaudi/unified-web-components` (^1.45.0) | `@oneaudi/unified-web-alpha-components` (1.22.0) |
| **Common** | `@oneaudi/unified-web-common` (^1.13.0) | `@oneaudi/unified-web-alpha-common` (1.11.0) |
| **Bundle Output** | `app.js` / `app-ssr.js` | `app-alpha.js` / `app-alpha-ssr.js` |
| **Selection** | AEM page configuration selects which bundle URL to load | |

### Implementation Approach

1. **Webpack Aliases:** Alpha builds remap `@oneaudi/unified-web-components` → `@oneaudi/unified-web-alpha-components` at build time via `alphaAliases` in webpack config
2. **`DynamicComponent` Pattern:** Components use `DynamicComponent` to render different implementations based on active design system
3. **`dynamicStylingHelper(legacyValue, alphaValue)`:** Utility function that returns the appropriate value based on active design system — used for CSS values, class names, component props
4. **`DynamicThemeProviderWrapper`:** Wraps the app in the appropriate theme provider

### Implications

- Every style change must be validated in **both** design variants
- Test matrices double (2 designs × 2 render modes × multiple breakpoints)
- Alpha component library uses fixed versions (1.22.0, 1.11.0) vs classic using ranges (^1.45.0)
- The dual-system adds build complexity and bundle size

---

## 8. External Service Dependencies

| Service | Type | Purpose | Used By |
|---------|------|---------|---------|
| **SCS (Stock Car Service)** | REST API | Vehicle data, search, filter, compare, matching, dealers. Base URL + market path (e.g., `de/nc`, `es/uc`). API key auth. | PLP, PDP (via shared) |
| **CRS (Car Rating Service)** | REST API | FSAG financial products, rate calculations, disclaimers. Built-in response caching. | Shared (finance hooks) |
| **OneGraph** | GraphQL | Carline names/groups for model filter resolution. Via `@oneaudi/onegraph-client`. | PLP (filter data) |
| **Omnigraph** | GraphQL (via auth proxy) | MyAudi wishlist CRUD. Accessed through `authService.fetch('/graphql')`. | Shared (wishlist context) |
| **Google Maps API** | JavaScript API | Location autocomplete, geocoding for "locate me", dealer map display. Requires two-click GDPR consent. | PLP (location filter) |
| **AVP (Audi Visualization Platform)** | Web Component (external script) | 3D vehicle webstreaming. Loaded from `avpRessourceUrl`. New cars only. | PDP (stage) |
| **Falcon Content API** | REST | Editorial content for Trade-In teaser Feature App. Origin: `oneaudi-falcon.prod.renderer.one.audi`. | PDP (trade-in) |
| **Partner Business Card FA** | Feature App (remote) | Dealer information rendering. Loaded via `FeatureAppLoader` at runtime. | Dealer-info, PDP |
| **myAudi Auth Service** | OAuth/Feature Service | Authentication (login/logout), token management. Auth proxy at `/qa-userinfo-emea/v2`. | PLP, PDP (favorites) |
| **Audi Tracking Service** | Feature Service | OneSight V2 analytics event dispatch. CSR-only. | PLP, PDP |
| **Footnote Services** | Feature Service | Legal footnote rendering and referencing. | PLP, PDP |
| **Notification Display Service** | Feature Service | Toast notifications (share success, wishlist add/remove). | PLP, PDP |
| **E-Commerce Service** | Feature Service (`@volkswagen-onehub/e-commerce-service`) | GLC lean checkout flow (reservation, cash, financing). | PDP (conversion bar) |
| **Navigation Service** | Feature Service | Page navigation for checkout flows. | PDP |
| **Page Info Service** | Feature Service | SEO metadata, HTTP response status (404 for soldout). | PDP, Soldout |
| **Env Config Service** | Feature Service | Environment-specific configuration (API URLs, keys). Template: `https://env-config.one.audi/config/live/{key}.json`. | All |
| **Instavid** | External script | Optional 360° spin integration. Script from `static.instavid360.com`. Usage status unclear. | PDP (hook exists) |

---

## 9. Content Model Architecture

### Content Fragment Hierarchy

```
AEM Content Fragments
├── VTP Configuration (shared, referenced by PLP + PDP)
│   ├── scopes_*        → Feature toggles (finance, ecom, phone, etc.)
│   ├── urls_*          → External links (start page, warranty logo, EEC image)
│   ├── assets_*        → Content reference assets
│   ├── cta[]           → CTA button definitions (type, label, URL, method, target)
│   │   └── options_*   → Per-CTA options (dealer filtering, reservation filter)
│   ├── scs_*           → SCS API path
│   ├── sortParams_*    → Sort options + default
│   ├── consumptionEmission_* → WLTP/NEDC display settings
│   ├── financeLayer_*  → Finance layer settings
│   ├── mainPriceConfiguration → Price type mappings per business model + availability
│   │   └── items[]     → Price fragment definitions (type, path, label)
│   └── priceConfiguration → Price breakdown configuration
│
├── Feature App - PLP (per PLP instance)
│   ├── appContext       → results | favorites
│   ├── isNewCarsPage    → NC/UC toggle
│   ├── detailsPageUrlPattern → PDP URL template
│   ├── quickFilters     → Up to 12 quick filter slots
│   ├── filterCategories → Up to 7 filter tab categories (25 groups each)
│   ├── carlinePhotos    → Override carline thumbnail images
│   ├── filterInfoLayers → Info button → URL mappings
│   ├── equipmentFilter_* → Equipment filter configuration
│   ├── locationFilterConfig_* → Google Maps auth, radius options
│   ├── campaigns        → Campaign configuration
│   └── vtpConfiguration → Reference to central VTP Configuration
│
├── Feature App - PDP (per PDP instance)
│   ├── searchLink       → Back navigation URL
│   ├── use3dWebstreaming → 3D view toggle
│   ├── avpRessourceUrl  → 3D script URL
│   ├── nbas             → Next Best Actions selection (favorite, share, audiCode)
│   ├── warrantyInfoLayer → Warranty type → info URL mappings
│   ├── scsTechdataInitial/Extended → Technical data field selection (65+ options)
│   ├── tradeInTeaser    → Trade-in editorial content
│   └── vtpConfiguration → Reference to central VTP Configuration
│
└── Feature App - Dealer Info
    ├── appVersion       → Partner Business Card FA version
    └── vtpDealerResultsURL → Dealer results link pattern
```

### Content Configuration Scope

Content authors control substantial application behavior without code changes:

- **Feature toggling:** Finance, ecom, phone CTAs, iframe forms, wishlist mode, 3D view, mandatory area search
- **CTA architecture:** Button type, label, URL, HTTP method, target window, display context, dealer filtering, reservation/payment filtering
- **Pricing:** Business model-specific price templates with availability variants
- **Filter system:** Which quick filters appear, filter tab categories, filter group ordering
- **Sort options:** Available sort keys and default selection
- **Consumption/emission:** Which test cycles to display, blacklisted items, regulation links
- **Technical data:** Which 65+ tech data fields to show, ordering, expandable sections
- **Market customization:** SCS market path, currency/date patterns, mileage/power units
- **URLs:** All external links (PDP pattern, start page, warranty, EEC images)

---

## 10. Technical Risks

### High Severity

| Risk | Impact | Evidence | Mitigation |
|------|--------|----------|------------|
| **Heavy coupling to `vtp-shared`** | Changes to shared affect all 4 consuming FAs simultaneously. Shared is a monolith (components, hooks, contexts, utils, types, services, rethink components). No independent versioning discipline (version 2.7.0 monolithically bumped). | `vtp-shared` exports 100+ modules consumed across PLP, PDP, dealer-info, and configuration | Consider splitting shared into smaller focused packages (e.g., `vtp-filters`, `vtp-finance`, `vtp-vehicle-types`) |
| **Dual design system complexity** | Every visual change requires validation in 2 designs × 2 render modes = 4 code paths. Alpha uses pinned versions while classic uses ranges — version drift risk. | Webpack alias swapping, `DynamicComponent`, `dynamicStylingHelper` throughout codebase | Plan migration timeline to deprecate classic design and remove dual-build |
| **Low Lighthouse accessibility threshold** | Accessibility score threshold is 0.3 (30%) — far below WCAG 2.0 AA compliance requirements. This means significant accessibility regressions could pass CI. | `.lighthouserc.json` in both PLP and PDP packages | Raise accessibility threshold to at least 0.7, add axe-core automated testing |
| **Content Fragment as feature flag system** | AEM Content Fragments serve as the sole feature toggling mechanism. No runtime feature flags (LaunchDarkly). Changes require content author publication, which may not support instant rollback or percentage rollouts. | 15+ toggleable scopes managed via `scopes_*` fields | Evaluate adding a proper feature flag service for high-risk feature rollouts |

### Medium Severity

| Risk | Impact | Evidence | Mitigation |
|------|--------|----------|------------|
| **Global state via `Symbol.for()`** | Singleton contexts across multiple FAs assume they share the same React instance. If Feature Hub loads FAs in isolated contexts, the singleton pattern fails silently. | `ServicesContext.tsx`, `StockContext.tsx` use `Symbol.for()` | Document this assumption; add runtime validation |
| **Legacy + Rethink component duplication** | Many components exist in both legacy and rethink versions (finance, CTA, filters). Dual maintenance burden and risk of behavior divergence. | `src/components/finance/` vs `src/rethink/pdp/components/finance/`, legacy CTA vs rethink CTA | Track which markets still use legacy; schedule deprecation |
| **Market-specific hardcoded branches** | Germany (ENVKV), France (EEC labels), Spain (price template, sort exception), Japan (price template) have hardcoded market checks. Adding new market-specific logic requires code changes. | `countryCode === 'de'`, `country === 'fr'`, market-specific templates in shared | Abstract market-specific logic into strategy patterns or content-driven configuration |
| **90+ filter types** | Filter model supports 90+ filter types with multiple layout/rendering variations. High combinatorial complexity for testing. | `models/filterModel.ts` enumerates all filter types | Focus E2E tests on most commonly configured filters |
| **SSR skeleton-only for PLP** | PLP SSR renders only skeleton placeholders, not real content. This limits SEO value for vehicle listing pages and increases time-to-content. | `FeatureApp.tsx` — `typeof window === 'undefined'` renders skeletons | Investigate full SSR for PLP with data fetching (like PDP does) |
| **React 16/17/18 compatibility requirement** | Supporting three major React versions constrains API usage (can't use newer hooks or features). | `package.json` peer dependency ranges | Define minimum supported React version; plan migration |

### Low Severity

| Risk | Impact | Evidence | Mitigation |
|------|--------|----------|------------|
| **Hardcoded page size** | Pagination fixed at 12 vehicles per page. Not content-configurable. | `requestParam.set('size', 12)` in `CountAndSort.tsx` | Make configurable if markets need different page sizes |
| **Partner Business Card version pinning** | Dealer-info FA defaults to `v3.2.0-rc.4` of partner-business-card — a release candidate, not a stable version. | `DealerInfo.tsx` fallback version | Update to stable release version |
| **Skipped tests** | Dealer-info has its main test file marked `test.skip`. Reduced confidence in regression detection. | `FeatureApp.test.tsx` uses `test.skip` | Re-enable skipped tests |
| **Boilerplate serverless functions** | Soldout package contains unused serverless function stubs (`hello-world_get.ts`). Technical debt. | `src/api/` in soldout package | Remove unused boilerplate |

---

## 11. Non-Functional Considerations

### Performance

| Aspect | Current State | Notes |
|--------|--------------|-------|
| **Lighthouse CI Threshold** | Performance ≥ 0.75 (warning only), Accessibility ≥ 0.3 (error) | Thresholds are low relative to industry standards |
| **SSR** | PDP does full SSR; PLP renders skeletons only | PLP SEO content depends on client-side rendering |
| **Image Optimization** | WebP format via mediaservice, responsive `srcSet` (400/600/900/1000/1440px), lazy loading with eager switch for adjacent slides | Good image optimization patterns |
| **Bundle Output** | 4 bundles per FA (classic CSR/SSR + alpha CSR/SSR) | Ensures only one design system loaded per page |
| **SWC Transpilation** | Uses SWC (`@swc/cli`) instead of Babel for faster builds | Modern build tooling |
| **State Serialization** | Configuration and vehicle state serialized during SSR for instant CSR hydration | Avoids duplicate API calls on hydration |
| **Pagination** | Progressive "Load More" (not infinite scroll), 12 items per batch | Controlled memory usage |
| **Finance API Caching** | CRS API responses cached via `getCachedValue()`/`updateCachedValue()` | Reduces duplicate finance API calls |
| **Scroll Position Restoration** | `setSelectedVehicle` / `scrollToPreviouslySelectedVehicle` for PLP → PDP → PLP flow | Good UX pattern for return navigation |

### Accessibility

| Aspect | Current State | Notes |
|--------|--------------|-------|
| **WCAG Compliance** | Target: WCAG 2.0 AA (per Audi platform requirements) | Lighthouse threshold at 0.3 undermines enforcement |
| **ARIA Labels** | `aria-label` on all interactive elements (buttons, links, images) | Consistently implemented |
| **Focus Management** | PLP: focus moves to first newly loaded tile after "Load More". PDP: focus-on-scroll to first visible focusable element | Good focus management patterns |
| **Keyboard Navigation** | Tab navigation uses `onKeyNavigate`. `role="tablist"` and `role="tab"` for tab panels | Keyboard-accessible tab navigation |
| **Semantic HTML** | Heading hierarchy: h2 for section headings, h3 for subsections. h3 for carline name in tiles | Proper heading semantics |
| **Link Security** | `rel="noopener"` on external `target="_blank"` links | Security best practice followed |
| **Layer Focus Trapping** | `primaryAriaLabel` and `secondaryAriaLabel` for gallery and warranty layers | Focus layer accessibility |

### SEO

| Aspect | Current State | Notes |
|--------|--------------|-------|
| **SEO-Friendly URLs** | PDP URLs generated from model year + description + ID. SEO filter resolver creates human-readable filter URLs. | Good URL architecture |
| **Page Info Service** | PDP sets page metadata via `page-info-service`. Soldout sets 404 status. | Server-side SEO signals |
| **SSR Limitation** | PLP renders skeletons during SSR — vehicle listing content not available for crawlers in initial HTML | Potential SEO gap for listing pages |
| **SEO Filter Resolution** | `createFilterSeoResolver` resolves SEO-friendly filter slugs before app initialization | Filter pages are crawlable |
| **Carline SEO Resolver** | `createCarlineSeoResolver` provides carline-specific SEO metadata | Per-model SEO data |

### Internationalization (i18n)

| Aspect | Current State | Notes |
|--------|--------------|-------|
| **i18n Service** | `@oneaudi/i18n-service` (^2.2.0) + `@oneaudi/i18n-context` (^5.1.0) | Full i18n infrastructure |
| **Translation Keys** | 40+ key patterns (`stockcars.*`, `nemo.ui.sc.*`) covering all user-facing text | Comprehensive coverage |
| **Market Configuration** | SCS market path, currency, currency pattern (10 options), date pattern (10 options), mileage unit (km/miles), power unit (PS/HP) | Rich market customization |
| **Number/Currency Formatting** | `@oneaudi/number-formatter-service` (^1.0.4) for locale-aware formatting | Service-based formatting |
| **Market-Specific Logic** | Germany (ENVKV), France (EEC label), Spain (price template, sort), Japan (price template) have code-level branches | Some markets require code changes |
| **Locale Service** | `@volkswagen-onehub/locale-service` provides language/country detection | Platform-standard locale resolution |
| **Hardcoded German fallback** | `CountAndSort.tsx` contains hardcoded German text as sorting explanation fallback | Should be moved to i18n |

### Security

| Aspect | Current State | Notes |
|--------|--------------|-------|
| **HTML Sanitization** | DOMPurify (^3.2.4) used for sanitizing HTML content (dealer remarks, product deviations) | Industry-standard XSS protection |
| **GDPR / Cookie Consent** | Google Maps integration requires explicit two-click consent with persistent or per-session option | GDPR-compliant consent flow |
| **Auth Proxy** | Authentication routed through `/qa-userinfo-emea/v2` proxy — not direct to auth provider | Server-side auth proxy pattern |
| **API Key Auth** | SCS API uses token-based authentication via header | Standard API auth |
| **External Link Safety** | `rel="noopener"` on `target="_blank"` links | Prevents tab-napping |
| **Content Security** | Feature Apps rendered within AEM's security context | Platform-managed security headers |

---

## Appendix A: Package Summary Table

| Package | Type | App Store ID | Version | Visual | SSR | Design Variants |
|---------|------|-------------|---------|--------|-----|-----------------|
| `fa-vtp-plp` | Feature App | `1894ccb5-dbea-4a6a-9fd8-068d635f0d66` | 4.19.1 | Yes | Skeletons | Classic + Alpha |
| `fa-vtp-pdp` | Feature App | `9877d64f-c7e8-42b7-80eb-d597ba12b311` | 4.30.1 | Yes | Full SSR | Classic + Alpha |
| `fa-vtp-configuration` | Feature App (headless) | `b84607fc-e63a-4531-a0f5-92ee1ce147a1` | 5.14.0 | No | State serialization | N/A |
| `fa-vtp-dealer-info` | Feature App | `9fea9015-4853-4c93-851e-fe338f9c1c19` | 1.17.0 | Yes (delegates) | Yes | Classic + Alpha |
| `fa-vtp-soldout` | Feature App (headless) | `852f355b-09dc-4797-a432-a06e6a66bff1` | 1.1.0 | No | 404 status | N/A |
| `vtp-configuration-service` | Feature Service (library) | N/A | 0.0.1 | No | State serialization | N/A |
| `vtp-shared` | Library | N/A | 2.7.0 | N/A | SSR utilities | N/A |

## Appendix B: Feature Service Dependency Matrix

| Feature Service | PLP | PDP | Config | Dealer-Info | Soldout |
|----------------|-----|-----|--------|-------------|---------|
| `gfa:locale-service` | ✓ | ✓ | ✓ | ✓ | |
| `vtp-configuration-service` | ✓ | ✓ | ✓ (provides) | ✓ | |
| `audi-content-service` | ✓ | ✓ | ✓ | ✓ | |
| `dbad:audi-i18n-service` | ✓ | ✓ | | ✓ | |
| `audi:envConfigService` | ✓ | ✓ | | ✓ | |
| `audi-tracking-service` | ✓ (opt) | ✓ (opt) | | ✓ (opt) | |
| `layer-manager` | ✓ | ✓ | | ✓ (opt) | |
| `audi-footnote-reference-service` | ✓ | ✓ | | | |
| `audi-footnote-service` | ✓ (opt) | ✓ (opt) | | | |
| `vw:authService` | ✓ | ✓ | | | |
| `audi-notification-display-service` | ✓ (opt) | ✓ (opt) | | | |
| `audi-number-formatter-service` | | ✓ | | | |
| `navigation-service` | | ✓ (opt) | | | |
| `e-commerce-service` | | ✓ (opt) | | | |
| `page-info-service` | | ✓ (opt) | | | ✓ (opt) |
| `gfa:service-config-provider` | | ✓ | | | |
| `s2:logger` | ✓ (opt) | ✓ (opt) | | ✓ (opt) | ✓ |
| `s2:async-ssr-manager` | ✓ (opt) | ✓ (opt) | | | |
| `s2:server-request` | ✓ (opt) | ✓ (opt) | | | |
| `s2:serialized-state-manager` | ✓ (opt) | ✓ (opt) | | | ✓ (opt) |
| `audi-stockcars-store-service` | ✓ | | | ✓ | |
| `onegraph-service` | ✓ | | | | |
