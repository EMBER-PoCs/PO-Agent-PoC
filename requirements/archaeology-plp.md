# Code Archaeology Report: PLP (Product Listing Page)

**Package:** `@oneaudi/fa-vtp-plp`  
**Version:** 4.19.1  
**App Store ID:** `1894ccb5-dbea-4a6a-9fd8-068d635f0d66`  
**Team:** Mariokart  
**Supplier:** Accenture Song  
**Date:** 2026-03-22  

---

## Overview

- **Purpose:** The PLP (Product Listing Page) is a React-based Feature App (microfrontend) that provides the core vehicle listing/browsing experience. Users can browse, filter, sort, and favorite vehicles — both new and used cars. It renders within AEM pages via the Feature Hub.
- **Entry point:** `src/app/FeatureHubAppDefinition.tsx` (Feature Hub integration), `src/app/FeatureApp.tsx` (main React component), `src/bootstrap.tsx` (standalone dev)
- **Design variants:** Two design systems: `classic` (legacy) and `alpha` (newer design). Both CSR and SSR bundles are built for each.
- **Key dependencies (internal):**
  - `@oneaudi/vtp-shared` — shared utilities, filter context, filter components, vehicle types, tracking helpers, auth context, wishlist context, image utilities
  - `@oneaudi/vtp-configuration-service` — centralized configuration service
  - `@oneaudi/stockcars-store-service` — Redux-based vehicle store
  - `@oneaudi/stck-store` — Redux store actions/selectors for vehicles, filters, sorting, favorites, UI state
  - `@oneaudi/onegraph-service` / `@oneaudi/onegraph-client` — GraphQL client for Audi OneGraph API
  - `@oneaudi/content-service` — AEM Content Fragment delivery
  - `@oneaudi/i18n-service` / `@oneaudi/i18n-context` — internationalization
  - `@oneaudi/footnote-reference-service` / `@oneaudi/footnote-service` — footnote rendering
  - `@oneaudi/audi-tracking-service` — analytics tracking
  - `@oneaudi/audi-auth-service` — authentication (myAudi)
  - `@oneaudi/audi-notification-display-service` — toast notifications
  - `@oneaudi/unified-web-components` / `@oneaudi/unified-web-common` — Audi design system (classic)
  - `@oneaudi/unified-web-alpha-components` / `@oneaudi/unified-web-alpha-common` — Audi design system (alpha)
  - `@volkswagen-onehub/layer-manager` / `@volkswagen-onehub/gfa-layer-manager` — overlay/layer management
  - `@volkswagen-onehub/locale-service` — locale (language/country)
- **Key dependencies (external):** React 16/17/18, Redux (react-redux), styled-components

---

## Features Found

### 1. Vehicle Tile Grid Display
- **What it does:** Displays a responsive grid of vehicle tiles, each showing the vehicle carline name, model code, image gallery, pricing/finance info, key features, dealer info, availability badge, action buttons (CTA + favorites), and consumption/emission data.
- **Key files:**
  - `src/app/components/Tiles/Tiles.tsx` — Grid container, scrolls to previously selected vehicle on return
  - `src/app/components/Tiles/Tile.tsx` — Individual vehicle tile
  - `src/app/components/Tiles/Results.tsx` — Results context wrapper, renders tiles from Redux store vehicle map
  - `src/app/components/Tiles/Tiles.styles.tsx` — Responsive CSS grid (1 col → 2 col at MD → 2–3 col at LG → 3–4 col at 2XL, varies by context)
  - `src/app/components/Tiles/Tile.styles.tsx` — Tile styling with alpha/legacy design variants
- **UI elements:** Vehicle image gallery, carline headline (h3), model code text, finance info, key features, availability badge, dealer info, CTA buttons, favorites icon button, consumption and emission data
- **User interactions:** Click tile image/headline → navigate to details page. Click CTA buttons → contact dealer / go to ecom / go to details. Click favorites → add/remove from favorites. Swipe gallery images.

### 2. Vehicle Image Gallery
- **What it does:** Shows one or more images per vehicle tile with swipe/arrow navigation and pagination indicator. Prioritizes render images, then dealer images (for used cars unless nationWideSelling), with a fallback image.
- **Key files:**
  - `src/app/components/Tiles/Gallery/Gallery.tsx` — Gallery container with ImageSlider and Pagination
  - `src/app/components/Tiles/Gallery/GalleryImage.tsx` — Individual image with responsive srcSet, lazy loading, and WebP support
- **UI elements:** Image slider, pagination arrows (Previous/Next), pagination counter (e.g., "1/5")
- **User interactions:** Swipe to change image, click arrows to navigate, click image to navigate to details page
- **Business rules:**
  - Used car images (`type === 'U'`) may show dealer images; render images hidden if `hideRenderImages` is true
  - `nationWideSelling` vehicles never show dealer images
  - Images support VTP image URLs, Media Service URLs (with WebP optimization), and fallback images
  - Lazy loading with eager loading triggered when scrolling near current position

### 3. Sorting
- **What it does:** Provides a sort dropdown allowing users to reorder vehicles by various criteria. Sort options are content-configurable.
- **Key files:**
  - `src/app/components/Tiles/CountAndSort.tsx` — Contains sort SelectField, result count display, sorting info popover
  - `src/app/components/Tiles/ResultsBar.tsx` — Assembles count + sort + share components
- **UI elements:** Sort dropdown (`<SelectField>`), sorting info button (Popover with explanation), vehicle count headline, share button
- **User interactions:** Select sort option → fetches re-sorted results from SCS API
- **Sort options documented in code (configurable per market):**
  - `relevance:asc`, `price:asc`, `price:desc`, `byDistance:asc`, `actionVehicles`, and more (configured via `sortParams_options` in content)
  - Default option configurable via `sortParams_defaultOption`
  - Distance sorting available only when location is set
  - Campaign sorting (`actionVehicles`) shown only if campaign results exist
- **Business rules:**
  - Sort disabled when only 0 or 1 result
  - Distance sorting (`byDistance:asc`) auto-applied when location filter is set (except ES market)
  - Sort state persisted in session (`saveSortPLPSession` / `getPLPSession`)
  - A configurable filter (`overWriteSorting_filterToTriggerSorting`) can force override to a specific sort key (`overWriteSorting_overrideDefaultSortingKey`)
  - Some markets maintain separate i18n keys for each sort option instead of composite keys

### 4. Load More (Pagination)
- **What it does:** Progressive loading — shows a "Load More" button to fetch the next batch of vehicles. Not infinite scroll.
- **Key files:**
  - `src/app/components/Tiles/LoadMoreButton.tsx` — Load more button with progress indicator
- **UI elements:** "Load More" secondary button, progress bar indicator during loading
- **User interactions:** Click "Load More" → fetches next 12 vehicles, appends to list
- **Business rules:**
  - Load increment is fixed at **12 vehicles** per load
  - Initial page size is also 12 (`requestParam.set('size', 12)` in CountAndSort)
  - Button hidden when all vehicles are loaded (`vehiclesTotalCount <= vehiclesFetched.length`)
  - After loading, focus moves to the first newly loaded tile (accessibility)
  - Button hidden when total count is null/undefined

### 5. Quick Filters (Sidebar / Mobile Fullscreen)
- **What it does:** Provides a sidebar panel (desktop) or fullscreen overlay (mobile) with configurable quick-access filters. Supports checkbox filters, color filters, car-type icon filters, model checkboxes, range filters, and location filters.
- **Key files:**
  - `src/app/components/quickFilters/subcomponents/QuickFilters.tsx` — Main quick filters container
  - `src/app/components/quickFilters/subcomponents/QuickModelFilter.tsx` — Model/carline group checkboxes with show more/less
  - `src/app/components/quickFilters/subcomponents/QuickModelCheckbox.tsx` — Individual carline group checkbox
  - `src/app/components/quickFilters/subcomponents/QuickLocationFilter.tsx` — Location search with Google Maps autocomplete, radius selector, locate-me, two-click consent
  - `src/app/components/quickFilters/subcomponents/SwitchNCUC.tsx` — New Cars / Used Cars toggle buttons
  - `src/app/components/quickFilters/styles/` — Styled components for quick filters
- **UI elements:**
  - Headline ("All Stock Cars" / "Basic filters")
  - Filter chips showing active selections
  - Checkbox filters, color-tile filters, car-type icon filters
  - Range filters (sliders)
  - Model group checkboxes with "Show more options" / "Hide options" toggle
  - Location search input with Google Maps autocomplete
  - Radius dropdown selector (configurable options, default 10)
  - "Locate me" geolocation button
  - "Advanced" button → opens full filter overlay (advanced filters)
  - Results count button (mobile) → closes overlay and shows results
  - New Cars / Used Cars toggle switch
  - Close button (mobile overlay)
- **User interactions:** Select/deselect filters → results update dynamically, enter location → filter by distance, click "Advanced" → opens full filter overlay, switch NC/UC → navigates to other page
- **Business rules:**
  - Quick filters configuration is content-managed (up to 12 quick filters via CFM)
  - Filter type determined by suffix: `.checkbox`, `.color-checkbox`, `.car-type-icon-checkbox`, `.model-checkbox`, `.range`, `.location`
  - Model checkboxes shown with responsive breakpoint logic: 3 on XS/S/L/XL, 6 on M, 3 or 6 on XXL (depends on legacy vs alpha)
  - Desktop shows quick filters as sidebar; mobile shows as fullscreen layer
  - Quick filters only visible in `results` context (not `favorites`) and only on desktop by default
  - Filter overlay state managed via `isFilterOverlayOpened` flag in context
  - Mandatory area search modal can be triggered on overlay close

### 6. Location Filter with Google Maps Integration
- **What it does:** Allows users to search by location with Google Maps autocomplete, set distance radius, use browser geolocation.
- **Key files:**
  - `src/app/components/quickFilters/subcomponents/QuickLocationFilter.tsx`
- **UI elements:** Search input with autocomplete, radius dropdown, locate-me icon button, Google Maps two-click consent modal, consent toggle switch
- **User interactions:** Type address → autocomplete suggestions, select location → filter by radius, click locate me → use browser geolocation, grant/revoke cookie consent for Google Maps
- **Business rules:**
  - Google Maps requires two-click consent (GDPR compliance)
  - Consent can be persistent or per-session
  - Radius options configurable (default: `[10, 20, 50, 100, 200]`)
  - Default radius configurable (default: 10)
  - Location search state persisted in localStorage with 30-day expiry
  - Mandatory area search provider can force location input
  - Location determines dealer filter inclusion/exclusion

### 7. Favorites System
- **What it does:** Users can mark vehicles as favorites, view a dedicated favorites page, and manage their favorites list. Supports both local (localStorage) favorites and myAudi cloud-synced wishlist.
- **Key files:**
  - `src/app/components/Tiles/Favorites.tsx` — Favorites page view
  - `src/app/components/Tiles/ActionButtons.tsx` — Add/remove favorites button, myAudi wishlist integration
  - `src/app/components/Tiles/ZeroResultsPage.tsx` — Empty state for favorites
- **UI elements:** Favorite icon button (filled/unfilled heart), favorites count badge in toolbar, "Remove from favorites" confirmation overlay (in favorites view), "Zero favorites" empty state with headline and "New search" link
- **User interactions:** Click favorite → add/remove, confirmation dialog on remove in favorites view, navigate to favorites page via toolbar
- **Business rules:**
  - Two modes: local favorites (Redux store `ACTIONS.FAVORITE_VEHICLES`) or myAudi Wishlist (cloud sync)
  - `enableMyAudiWishlist` toggles between local and cloud mode
  - When myAudi wishlist is enabled and user is not authenticated, clicking favorite opens login layer
  - After login, pending wishlist add is retrieved from localStorage (`MYAUDI_WISHLIST_LOCAL_STORAGE_KEY`)
  - Vehicle type (`N` = new, `U` = used) is passed to wishlist API
  - In favorites view, removing a favorite shows confirmation overlay; in results view, removal is instant
  - Favorites data fetched from SCS API filtered by vehicle IDs

### 8. Toolbar
- **What it does:** Top bar with filter button (mobile), favorites link, and back button (favorites context).
- **Key files:**
  - `src/app/components/toolbar/Toolbar.tsx`
  - `src/app/components/toolbar/styles/Toolbar.styles.tsx`
- **UI elements:** Filter button with active filter count badge, favorites link with count, back button (favorites page only)
- **User interactions:** Click filter → open quick filters overlay (mobile), click favorites → navigate to favorites page, click back → `window.history.back()` or redirect to `/`
- **Business rules:**
  - Filter button only shown in `results` context
  - Favorites link hidden when `enableMyAudiWishlist` is true
  - Back button only shown in `favorites` context
  - Mobile filter button opens quick filters as fullscreen layer
  - Active filter count shown as badge/parenthetical

### 9. Consumption & Emission Display
- **What it does:** Shows vehicle consumption and emission data on tiles, with a detailed layer available.
- **Key files:**
  - `src/app/components/Tiles/ConsumptionAndEmission.tsx` — Tile-level consumption/emission summary
  - `src/app/components/Tiles/ConsumptionAndEmissionLayer/ConsumptionEmissionLayerPlp.tsx` — Full detail layer
  - `src/app/components/Tiles/ConsumptionAndEmissionLayer/ConsumptionAndEmissionDisplay.tsx` — Consumption and emission values list
  - `src/app/components/Tiles/ConsumptionAndEmissionLayer/EfficiencyDisplay.tsx` — Efficiency class image display
- **UI elements:** Inline consumption/emission values, "Details" link opening layer, WLTP/NEDC data sections, efficiency class image (SVG), footnote references
- **Business rules:**
  - **DE market:** Shows special `EnvKConsumptionAndEmissions` component (German EnVKV regulation)
  - **Non-DE markets:** Shows standard consumption/emission component with link to detail layer
  - WLTP data prioritized over NEDC; both can be shown if configured
  - Efficiency class image URL varies by country; French market uses `eecLabel` from vehicle data
  - Consumption emission blacklist (configurable) can hide certain emission types
  - Layer supports footnote references

### 10. Share Results
- **What it does:** Allows users to copy a shareable URL with current filter selections.
- **Key files:**
  - `src/app/components/Tiles/ShareButton.tsx`
- **UI elements:** Share icon link, toast notification on successful copy
- **User interactions:** Click share → URL with filter hash copied to clipboard, notification shown
- **Business rules:**
  - URL includes filter state from localStorage hash
  - Existing URL hashes are stripped before appending filter hash
  - Uses notification display service for success toast

### 11. CTA (Call-to-Action) Buttons
- **What it does:** Configurable action buttons on each tile — "Go to Details", "Contact Dealer", "Buy Online", phone call, etc.
- **Key files:**
  - `src/app/components/Tiles/ActionButtons.tsx`
- **UI elements:** Up to 2 CTA buttons per tile (primary + secondary), plus favorite icon button
- **Business rules:**
  - CTA configuration loaded from `vtp-configuration-service` via `useButtons` hook
  - Maximum 2 CTAs shown; first is primary, second is secondary
  - If no CTAs configured, fallback "Go to Details" button shown
  - CTA types: `details`, `contact`, `nws`, `bevAgency`, `ecom`, `phone`, `central-customer-hotline`
  - `details` CTA navigates to PDP; others use `GenericButtonRethink` shared component
  - Phone CTA can show phone number directly (`phoneWithNumber` flag)

### 12. SSR (Server-Side Rendering) Support
- **What it does:** Renders skeleton placeholders during SSR, hydrates client-side.
- **Key files:**
  - `src/app/FeatureApp.tsx` — SSR check: `typeof window === 'undefined'` renders skeletons
  - `src/app/components/Tiles/Skeleton/ToolBarSkeleton.tsx`
  - `src/app/components/Tiles/Skeleton/TilesSkeleton.tsx`
  - `src/app/components/Tiles/Skeleton/TileSkeleton.tsx`
  - `src/app/components/Tiles/Skeleton/QuickFilterSkeleton.tsx`
  - `src/app/components/Tiles/Skeleton/QuickFilterComponentsSkeleton.tsx`
  - `src/app/components/Tiles/TilesWrapper.tsx` — Shows skeleton during loading state or SSR
- **Business rules:**
  - SSR renders skeleton placeholders (not real vehicle data)
  - `isClient` check from `useClientServerUtils()` used to gate interactive elements
  - Loading state from Redux store (`SELECTORS.UI.getLoadingState`) also triggers skeleton display
  - Optional SSR services: `s2:async-ssr-manager`, `s2:server-request`, `s2:serialized-state-manager`

### 13. SEO Filter Resolution
- **What it does:** Resolves SEO-friendly filter values from URL/page context before initial render.
- **Key files:**
  - `src/app/FeatureHubAppDefinition.tsx` — calls `createFilterSeoResolver` during initialization
  - `src/app/FeatureApp.tsx` — receives `seoFilters` prop, passes to `entryFilters`
- **Business rules:**
  - SEO filters resolved before app initialization via `initializeFeatureApp`
  - Filter data fetched from SCS API on initial load and on URL hash changes
  - `cleanupFilterURLHash()` called after initial filter fetch

### 14. OneGraph Integration (Demo/Prototype)
- **What it does:** Demo component querying the Audi OneGraph GraphQL API for configured car data.
- **Key files:**
  - `src/app/components/one-graph/OneGraph.tsx` — Demo component querying `configuredCarByCarline`
  - `src/app/components/one-graph/configured-car-by-carline.graphql` — GraphQL query
- **Note:** This appears to be a demo/prototype component, not integrated into the main tile flow. The OneGraph client is initialized in `FeatureHubAppDefinition.tsx` via `OneGraphProvider`, so the infrastructure is production-ready.

---

## Data Models

### `OneCMSContent` (Content Fragment root type)
**File:** `src/app/types/oneCMS.ts`
| Field | Type | Description |
|-------|------|-------------|
| `quickFilters` | `QuickFilters` (optional) | Up to 12 configurable quick filter slots |
| `filterStartPagePathname` | `string` | Path to filter start page |
| `favouritesPagePathname` | `string` | Path to favorites page |
| `newSearchUrl` | `string` | URL for "new search" on zero favorites |
| `isNewCarsPage` | `boolean` | Whether this is a new cars page (vs used cars) |
| `plpOtherCarsPagePathname` | `string` | Link to the opposite car type page (NC↔UC) |
| `powerUnit` | `'ps' \| 'hp'` | Market-specific power unit |
| `mileageUnit` | `'km' \| 'miles'` | Market-specific mileage unit |
| `filterCategories` | `FilterCategoryFields[]` (optional) | Advanced filter categories (max 7), each with up to 25 filter groups |
| `carlinePhotos` | `CarlinePhotosFields[]` (optional) | Override carline images |
| `filterInfoLayers` | `InfoLayerFields[]` (optional) | Info buttons that open info layers for specific filters |
| `equipmentFilter_introCategory` | `EquipmentFilterOptions` (optional) | Equipment filter intro category |
| `equipmentFilter_subCategories` | `EquipmentFilterFields[]` (optional) | Equipment filter sub-categories (max 4) |
| `locationFilterConfig_mapsAuthQueryParams` | `string` (optional) | Google Maps auth params |
| `locationFilterConfig_defaultRadius` | `number` (optional) | Default search radius |
| `locationFilterConfig_radiusOptions` | `number[]` (optional) | Available radius options |
| `filterHandlerOnly` | `boolean` | If true, PLP renders nothing (filter handler only mode) |
| `detailsPageUrlPattern` | `string` (optional) | URL pattern for PDP links (with `SC_VEHICLE_ID` / `SC_VEHICLE` placeholders) |
| `appContext` | `'results' \| 'favorites'` | Which context this instance operates in |

### `oneCMSVtpConfiguration` (Shared VTP Configuration)
**File:** `src/app/hooks/tileHook.tsx`
| Field | Type | Description |
|-------|------|-------------|
| `scopes_iframeForms` | `boolean` | Enable iframe forms |
| `scopes_financeEnabled` | `boolean` | Enable finance display |
| `scopes_financeOption` | `FinanceOptionType` | Finance option variant |
| `scopes_interpretDisclaimersTextStyle` | `boolean` | Interpret disclaimers text |
| `scopes_hideCalculationDisclaimer` | `boolean` | Hide calculation disclaimers |
| `scopes_hideRateChangeCTA` | `boolean` | Hide rate change CTA |
| `scopes_hideFinanceForEcom` | `boolean` | Hide finance for ecom vehicles |
| `scopes_forcePhoneAsPrimary` | `boolean` | Force phone CTA as primary |
| `scopes_phoneWithNumber` | `boolean` | Show phone number on CTA |
| `cta` | `any[]` | CTA button configurations |
| `urls_scStartPageLink` | `string` | Stock car start page URL |
| `urls_eecImageUrl` | `string` | Energy efficiency class image base URL |
| `urls_warrantyPlusLogoURL` | `string` | Warranty plus logo URL |
| `scs_scsMarketPath` | `string` | SCS API market path (e.g., `de/nc`, `es/uc`) |
| `sortParams_options` | `SortOption[]` | Available sort options |
| `sortParams_defaultOption` | `string` | Default sort option |
| `currency` | `string` (optional) | Currency symbol |
| `currencyPattern` | `PatternType` | Currency formatting pattern |
| `datePattern` | `DatePatternType` (optional) | Date formatting pattern |
| `overWriteSorting_filterToTriggerSorting` | `string` | Filter name that triggers sort override |
| `overWriteSorting_overrideDefaultSortingKey` | `string` | Override sort key when triggered |
| `enableMyAudiWishlist` | `boolean` | Enable cloud-synced myAudi wishlist |

### `VehicleBasic` (from `@oneaudi/vtp-shared`)
**Referenced throughout tiles.** Key fields used in PLP:
- `id`, `type` ('N'=new, 'U'=used), `symbolicCarline.description`, `modelCode.description`
- `pictures`, `tilesPictures`, `fallbackPictures` — image arrays
- `nationWideSelling`, `hideRenderImages` — image display rules
- `dealer.name` — dealer info
- `entryUrl` — fallback details page URL
- `modelYear`, `model.description` — for SEO URL generation
- `io.hasWltp`, `io.hasNedc`, `vlsEnergyProvision.hasWltp/hasNedc` — emission data availability
- `efficiencyClass`, `vlsEfficiencyClass.code`, `envkvIOData.efficiencyClass` — efficiency rating
- `eecLabel` — French market efficiency label
- `noNedc` — COC6 emission representation flag

### Carline type
**File:** `src/app/types/oneCMS.ts`
Supported carline values: `a1`, `a2`, `a3`, `a4`, `a5`, `a6`, `a7`, `a8`, `q2`, `q3`, `q4`, `q5`, `q6`, `q7`, `q8`, `q8etron`, `tt`, `r8`, `q6etron`, `etrongt`

---

## API Dependencies

### SCS (Stock Car Service) API
- **Base URL:** Configured via `envConfig.scs.baseUrl` and `scs_scsMarketPath`
- **API Key:** `envConfig.scs.apiKey`
- **Endpoints used:**
  - `GET /search/filter/{marketPath}` — Fetches filtered vehicle results with sort/filter/size params
  - `GET /range-scope` (via `getRangeScope`) — Fetches available filter ranges
  - `GET /campaigns` (via `fetchCampaignsForPLP`) — Fetches campaign data for PLP
  - Generic vehicle fetch for favorites (via `fetchAnythingScs`)
- **Version mapping:** API version configurable per endpoint type (`filter`, `match`, etc.)
- **Authentication:** API key passed as header/param

### OneGraph GraphQL API
- **Client:** Apollo Client via `@oneaudi/onegraph-service`
- **Query:** `configuredCarByCarline` — Fetches car configuration by carline (demo only currently)
- **Production use:** OneGraphProvider wraps the entire app, available for vehicle data queries

### Google Maps API
- **Purpose:** Location autocomplete, geocoding for "locate me"
- **Auth:** Configurable via `locationFilterConfig_mapsAuthQueryParams` (API key or client/channel)
- **Consent:** Two-click consent required (GDPR)

---

## Business Rules

### Filter System
- **File:** `src/app/FeatureApp.tsx`, `src/app/components/quickFilters/`
- Filters fetched from SCS API on initial load via `entryFilters()`
- Filter data re-fetched on URL hash changes (`hashchange` event listener)
- Active filters persisted in URL hash and localStorage
- Filter URL hash cleaned up after initial fetch
- `filterHandlerOnly` mode: If true, PLP renders nothing (used when only the filter handler is needed, not the listing UI)

### Filter Types (90+ options)
- **File:** `models/filterModel.ts`
- **Checkbox filters:** carline, body-type, fuel, gear-type, drive, color-type, equipment subcategories, campaigns, dealers, warranty types, brakes, efficiency, seats, light, sound, steering wheel, etc.
- **Range filters:** price (retail, rate, all-in), power (kW, PS), mileage, displacement, acceleration, CO2 emission, electric range, model year, luggage capacity, initial registration year, previous owners, doors, finance amount, production year
- **Special filter types:** color-checkbox (color tiles), car-type-icon-checkbox (body type icons), model-checkbox (carline groups), equipment-block (grouped equipment), location (geo filter)
- **Layout:** Each filter can be configured with layout width: 50%, 100%, or 50%/100% (responsive)

### Vehicle Type Differentiation (New vs Used)
- **Files:** `src/app/components/quickFilters/subcomponents/SwitchNCUC.tsx`, various
- NC/UC toggle buttons allow switching between new and used car pages
- `isNewCarsPage` boolean determines current context
- Market path contains `uc` suffix for used cars (e.g., `de/uc` vs `de`)
- Image display logic differs: used cars may show dealer photos
- Filter options differ (e.g., `usedCarMileage`, `usedCarInitialRegistrationYear`)

### Sorting Override Mechanism
- **File:** `src/app/components/Tiles/CountAndSort.tsx`
- Configurable via `overWriteSorting_filterToTriggerSorting` and `overWriteSorting_overrideDefaultSortingKey`
- When a specific filter is activated (e.g., a geo filter), sorting auto-changes to a configured sort key
- Prevents re-triggering if the same filter value is already active

### PDP (Details Page) URL Generation
- **File:** `src/app/hooks/useVehicleDetailsPageUrl.ts`
- URL pattern configurable: `detailsPageUrlPattern` with placeholders `SC_VEHICLE_ID` and `SC_VEHICLE`
- `SC_VEHICLE` replaced with SEO-friendly URL generated from `modelYear`, `model.description`/`symbolicCarline.description`, and `id`
- Fallback: uses `vehicle.entryUrl` if no pattern configured
- All tile interactions that navigate to PDP set `selectedVehicle` for scroll-back tracking

### Return-to-Position Scroll
- **File:** `src/app/components/Tiles/Tiles.tsx`, `src/app/hooks/tileHook.tsx`
- When user clicks a vehicle tile and later returns, page scrolls to previously selected vehicle
- Uses `setSelectedVehicle(vehicle.id)` on click and `scrollToPreviouslySelectedVehicle()` on mount

---

## Market / Locale Variations

| Variation | Markets | Evidence |
|-----------|---------|----------|
| DE market: EnVKV-compliant consumption display | Germany (`de`) | `ConsumptionAndEmission.tsx` — checks `localeService.countryCode === 'de'`, renders `EnvKConsumptionAndEmissions` |
| DE market: Fallback sorting explanation text (German) | Germany (`de`) | `CountAndSort.tsx` — hardcoded German sorting explanation fallback if i18n key empty |
| FR market: Custom efficiency label image | France (`fr`) | `EfficiencyDisplay.tsx` — checks `country === 'fr'`, uses `vehicle.eecLabel` instead of computed SVG path |
| ES market: Distance sorting exception | Spain (`es`) | `CountAndSort.tsx` — when location is set, ES market does NOT auto-switch to distance sorting |
| Power units: PS vs HP | Market-configurable | `oneCMS.ts` — `powerUnit: 'ps' \| 'hp'` |
| Mileage units: km vs miles | Market-configurable | `oneCMS.ts` — `mileageUnit: 'km' \| 'miles'` |
| Currency & formatting | Market-configurable | `tileHook.tsx` — `currency`, `currencyPattern`, `datePattern` |
| SCS market path | Per-market | `scs_scsMarketPath` — e.g., `de/nc`, `es/uc`, `no/nc` |
| Used cars market flag | Market-configurable | `marketPath.includes('uc')` determines used car context |
| Sort option translation override | Some markets (CSR-1048) | `CountAndSort.tsx` — direct i18n key mapping for sort options per market |

---

## Feature Flags

| Flag / Toggle | What It Controls | Source |
|---------------|-----------------|--------|
| `enableMyAudiWishlist` | Cloud-synced myAudi wishlist vs local favorites | `vtpConfiguration.fields.enableMyAudiWishlist` (content-configured) |
| `filterHandlerOnly` | Hides entire PLP UI, only initializes filter handler | `contentService.getContent().fields.filterHandlerOnly` or `featureAppConfig.filterHandlerOnly` |
| `isNewCarsPage` | New vs used cars context | Content-configured boolean |
| `appContext` (`results` / `favorites`) | Switches between results listing and favorites view | Content-configured enumeration |
| `enableMandatoryAreaSearch` | Forces location input before showing results | `MandatoryAreaSearchProvider` context |
| `useEfficiencyImage` | Show/hide efficiency class image in consumption layer | `vtpConfiguration.fields.useEfficiencyImage` |
| `scopes_financeEnabled` | Enable finance/pricing display | `vtpConfiguration.fields` |
| `scopes_hideFinanceForEcom` | Hide finance for ecom vehicles | `vtpConfiguration.fields` |
| `scopes_forcePhoneAsPrimary` | Force phone CTA as primary button | `vtpConfiguration.fields` |
| `scopes_iframeForms` | Enable iframe-based forms | `vtpConfiguration.fields` |
| Google Maps cookie consent | Gate Google Maps features behind GDPR consent | `useGoogleMaps()` hook, `needConsent`/`effectiveConsent` |

**Note:** No LaunchDarkly feature flags were found in the PLP package itself. Feature toggling is done via content configuration (AEM Content Fragments) rather than a dedicated feature flag service.

---

## Edge Cases & Error Handling

### Zero Results State
- **File:** `src/app/components/Tiles/ZeroResultsPage.tsx`
- Two variants: zero results (filters too restrictive) and zero favorites (no favorites saved)
- Results context: Shows "No results found" text + "Reset filters" button that clears all filters and re-fetches
- Favorites context: Shows headline, copy text, and "New search" link to configured URL

### SSR Rendering (No Window)
- **File:** `src/app/FeatureApp.tsx`
- `typeof window === 'undefined'` → renders skeleton placeholders only
- Multiple `typeof document !== 'undefined'` guards throughout
- SSR services are optional dependencies

### Loading State
- **File:** `src/app/components/Tiles/TilesWrapper.tsx`
- Redux loading state (`SELECTORS.UI.getLoadingState`) triggers skeleton display
- `isClient` check gates interactive content

### Missing Vehicle Data
- **File:** `src/app/components/Tiles/Tile.tsx`
- `if (!vehicle) return null` — gracefully handles undefined vehicle
- Model code rendered conditionally: `vehicle.modelCode?.description && ...`

### Missing Configuration
- **File:** `src/app/FeatureApp.tsx`
- `if (!vtpConfiguration || !envConfig) return null` — renders nothing without configuration
- `if (!filterData || filterHandlerOnly || filterHandlerOnlyConfig) return null` — renders nothing without filter data

### Fallback Images
- **File:** `src/app/components/Tiles/Gallery/Gallery.tsx`
- If no render/dealer images available, falls back to `render_4x3` fallback image
- Image lazy loading with eager switch for adjacent slide positions

### Navigation History Edge Case
- **File:** `src/app/components/toolbar/Toolbar.tsx`
- Back button: if `window.history.length > 1`, goes back; otherwise redirects to `/`
- SSR guard: logs info message instead of navigating

### Favorites Layer - Remove Confirmation
- **File:** `src/app/components/Tiles/ActionButtons.tsx`
- In favorites context (not results), removing a favorite shows Yes/No confirmation overlay
- In results context, removal is immediate

### Geolocation Error
- **File:** `src/app/components/quickFilters/subcomponents/QuickLocationFilter.tsx`
- Geolocation errors logged to console, no user-facing error message

---

## Content Author Configuration

All configurable via AEM Content Fragment Models.

### Root Model (`Feature App - PLP`)
| Field | Type | What Authors Control |
|-------|------|---------------------|
| `appContext` | Enum: results/favorites | Whether this PLP instance shows search results or favorites |
| `detailsPageUrlPattern` | Text | URL pattern for vehicle detail page links |
| `filterStartPagePathname` | Text (required) | Path to filter start page |
| `favouritesPagePathname` | Text (required) | Path to favorites page |
| `newSearchUrl` | Text (required) | URL for "new search" link on empty favorites |
| `isNewCarsPage` | Boolean | Whether this is a new cars page |
| `plpOtherCarsPagePathname` | Text | URL to opposite car type page (NC↔UC switch) |
| `powerUnit` | Enum: PS/HP | Power unit display |
| `mileageUnit` | Enum: km/miles | Mileage unit display |
| `filterCategories` | Content Fragment references (max 7) | Advanced filter tab categories |
| `quickFilters` | Content Fragment reference | Quick filter configuration |
| `carlinePhotos` | Content Fragment references | Override carline thumbnail images |
| `filterInfoLayers` | Content Fragment references | Info buttons with layer content URLs |
| `equipmentFilter_introCategory` | Enum | Intro filter for equipment tab |
| `equipmentFilter_subCategories` | Content Fragment references (max 4) | Equipment sub-category filter sets |
| `locationFilterConfig_*` | Various | Location filter: Maps auth, dark theme ID, default radius, radius options |
| `filterHandlerOnly` | Boolean | Hide PLP UI (filter handler only mode) |
| `campaigns` | Content Fragment reference | Campaign configuration |
| `vtpConfiguration` | Content Fragment reference | Central VTP configuration (shared across PLP/PDP) |

### Child Models
| Model | What It Configures |
|-------|-------------------|
| `Feature App - PLP: Category and Filters` | A filter tab with label + up to 25 filter groups with layout widths |
| `Feature App - PLP: Quick Filters` | Up to 12 quick filter selections from all available filter types |
| `Feature App - PLP: Carline Photos` | Carline group name + override photo asset |
| `Feature App - PLP: Info Layers` | Filter group ID + layer content URL for info buttons |
| `Feature App - PLP: Equipment Filter` | Up to 5 equipment sub-category filters |

---

## Tracking (Analytics)

### Tracked Events
**Files:** `src/app/tracking/tilesTracking.ts`, `src/app/tracking/quickFiltersTracking.ts`

| Event Name | Action | Trigger |
|-----------|--------|---------|
| `vtp product list page` | `feature_app_ready` | App initialization complete |
| `vtp results - change gallery image` | `navigation` | Swipe or arrow click on image gallery |
| `vtp results - sort` | `content` | User changes sort option |
| `vtp results - go to detail page` | `internal_link` | Click on vehicle link/image/button to PDP |
| `vtp results - add to favorites` | `favorite` | Add vehicle to favorites |
| `vtp results - remove from favorites` | `favorite` | Remove vehicle from favorites |
| `vtp results - list view` / `vtp results - grid view` | `content` | Toggle view type |
| `vtp product list page - show more cars` | `content` | Click "Load More" button |
| `vtp product list page - tab click` | `content` | Tab navigation |
| `vtp product list page - click on contact/ecom/call` | Various | CTA button clicks |
| `vtp results - open modal layer` | `internal_link` | Open consumption/emission layer |
| `vtp results - close modal layer` | `content` | Close consumption/emission layer |
| `vtp quick filter - go to new/used cars platform` | `internal_link` | NC/UC switch click |
| `vtp quick filter - go to advanced filters` | `internal_link` | Click advanced filter button |
| `vtp filter - filter layer` | `view_change` | Filter overlay opened |
| `vtp quick filter - open quick filters` | `content` | Mobile: open quick filters |
| `vtp quick filter - click to show results` | `content` | Mobile: close quick filters |
| `vtp share` | `content` | Share results link click |

### Component Update Data (sent with `feature_app_ready`)
- `implementer: 2`
- `availableCategories` — sorted list of configured filter tab names
- `sortingOption` — current sort selection
- `search.name` — `'vtp search - new'` or `'vtp search - used'`
- `search.results` — total result count
- `search.filter` — active filter array
- `viewType: 'grid'`

---

## Test Coverage Insights

### Tests Found
| Test File | What It Verifies |
|-----------|-----------------|
| `src/test/feature-app-setup.test.tsx` | FeatureApp has a default export named 'FeatureApp' |
| `src/app/components/Tiles/Tiles.test.tsx` | Tiles container renders children within `TilesContainer` |
| `src/app/components/Tiles/Tile.test.tsx` | Tile renders carline description and model code; returns empty for undefined vehicle |
| `src/app/components/Tiles/TilesWrapper.test.tsx` | Renders Results component for `appContext='results'`, Favorites for `appContext='favorites'` |
| `src/app/components/Tiles/LoadMoreButton.test.tsx` | Button renders when more vehicles exist; hides when all loaded; calls `loadFilteredResults` on click |
| `src/app/components/Tiles/ZeroResultsPage.test.tsx` | Zero results shows "No results found" + reset button; zero favorites shows headline + "New search" link |
| `src/app/components/Tiles/ActionButtons.test.tsx` | Add favorite dispatches `addFavoriteVehicleId`; remove in favorites view shows confirmation overlay; overlay dismiss doesn't remove |
| `src/app/components/Tiles/ResultsBar.test.tsx` | Passes formatted count and label to CountAndSort component |
| `src/app/components/Tiles/ConsumptionAndEmission.test.tsx` | Renders consumption element for vehicles with WLTP data |
| `src/app/components/Tiles/Favorites.test.tsx` | Shows zero results page when no favorites exist |
| `src/app/components/Tiles/Gallery/Gallery.test.tsx` | Renders fallback picture; shows dealer images for used cars with pagination |
| `src/app/components/quickFilters/subcomponents/QuickLocationFilter.test.tsx` | Location filter renders; tests Google Maps consent flow |
| `src/app/components/quickFilters/subcomponents/QuickModelCheckbox.test.tsx` | Model checkbox renders with count; handles selection toggle |
| `src/app/hooks/useVehicleDetailsPageUrl.test.ts` | (exists, not fully read) |
| `src/app/hooks/tileHook.test.tsx` | (exists, not fully read) |

---

## Infrastructure & Build

### Build Configuration
- **File:** `oneaudi-cli.json`
- Template: `cfa-template` (Custom Feature App)
- Features enabled: App Store registration, semantic release, OneGraph, OneSight tracking, RocketChat, Dependabot, SWC (Speedy Web Compiler)
- Package manager: yarn
- Node version: v18.20.4
- AWS domain: `arcade.apps.one.audi`

### Lighthouse Configuration
- **File:** `.lighthouserc.json`
- Accessibility minimum score: **0.3** (error threshold)
- Performance minimum score: **0.75** (warning threshold)

### Design Variants
- **Classic (legacy):** `fh/app.js` (CSR), `fh/app-ssr.js` (SSR)
- **Alpha (new design):** `fh/app-alpha.js` (CSR), `fh/app-alpha-ssr.js` (SSR)
- Dynamic styling: `dynamicStylingHelper('legacyValue', 'alphaValue')` and `DynamicComponent` render different UIs

---

## Integration with Other Packages

| Package | Integration Point | How |
|---------|--------------------|-----|
| **PDP** | Vehicle details page navigation | PLP generates PDP URLs via `detailsPageUrlPattern` with `SC_VEHICLE_ID`/`SC_VEHICLE` placeholders |
| **Configuration** (`fa-vtp-configuration`) | Central VTP settings | PLP reads `vtpConfiguration` content fragment for shared settings (finance, CTAs, URLs, sorting, consumption) |
| **Shared** (`@oneaudi/vtp-shared`) | Core business logic | Filter context, filter components, vehicle types, image utilities, tracking helpers, auth/wishlist context, SEO resolver, price/finance components |
| **Configuration Service** (`vtp-configuration-service`) | Service definition | PLP defines its own instance of the configuration service; reads CTA configs, sort options, scopes |
| **Stockcars Store** (`stck-store`) | State management | Redux store for vehicles, filters, sorting, favorites, UI state |
| **Content Service** (`audi-content-service`) | Content delivery | AEM Content Fragment data consumed at runtime |
| **Dealer Info** | Dealer data display | `DealerInfoRethink` shared component rendered in each tile |

---

## What Was NOT Fully Analyzed (Remaining Areas)

1. **`@oneaudi/vtp-shared` internals** — The shared package contains the bulk of filter logic (`FilterContextProvider`, `CheckboxFilter`, `RangeFilter`, `FilterOverlay`, `entryFilters`, `fetchResults`, `generateFilterRequestParam`), price/finance components (`FinanceRethink`), image utilities, and tracking helpers. A separate archaeology of this package would reveal deeper filter mechanics, price formatting rules, and shared business logic.
2. **Content overrides system** — `content-overrides/` directory was not analyzed. This contains market-specific content configuration overrides.
3. **Webpack configuration** — `webpack.config.js` was not read. May contain environment-specific build flags, bundle splitting configuration.
4. **CDK infrastructure** — `infrastructure/` and `cdk.json` were not analyzed. Contains AWS CDK deployment configuration.
5. **Protected/changelog** — `protected/` directory was not analyzed.
6. **Full i18n coverage** — Over 40 i18n keys referenced but the full translation scope was not cataloged.
7. **Cypress E2E tests** — Separate `cypress/` directory exists at monorepo root level but PLP-specific test specs were not analyzed.
8. **`docs/` directory** — Documentation build configuration was not analyzed.
