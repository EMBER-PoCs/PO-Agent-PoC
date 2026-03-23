---

# Code Archaeology Report: VTP Supporting Packages

**Packages analyzed:**
1. `@oneaudi/fa-vtp-configuration` (v5.14.0)
2. `@oneaudi/vtp-configuration-service` (v0.0.1)
3. `@oneaudi/fa-vtp-dealer-info` (v1.17.0)
4. `@oneaudi/fa-vtp-soldout` (v1.1.0)
5. `@oneaudi/vtp-shared` (v2.7.0)
6. Top-level `shared/` directory

**Team:** Mariokart  
**Supplier:** Accenture Song  
**Date:** 2026-03-22

---

## 1. VTP Configuration Feature App (`@oneaudi/fa-vtp-configuration`)

### Overview

- **Purpose:** A _headless_ Feature App (renders nothing visually) that reads configuration content from AEM Content Fragments and pushes it into the shared `vtp-configuration-service`. This acts as the central configuration bridge between AEM content authoring and all VTP Feature Apps.
- **App Store ID:** `b84607fc-e63a-4531-a0f5-92ee1ce147a1`
- **Entry point:** [src/app/FeatureHubAppDefinition.tsx](source-repos/audi-eu-vtp/packages/configuration/src/app/FeatureHubAppDefinition.tsx)
- **Version:** 5.14.0
- **Renders:** `null` — this FA produces no UI output

### Features Found

#### Headless Configuration Injection
- **What it does:** On creation, it reads AEM content via `audi-content-service`, maps the flat AEM field structure into a nested `VTPConfiguration` object, and calls `configService.setConfiguration()` to broadcast it to all other VTP Feature Apps.
- **Key files:** [src/app/FeatureHubAppDefinition.tsx](source-repos/audi-eu-vtp/packages/configuration/src/app/FeatureHubAppDefinition.tsx)
- **UI elements:** None (headless)
- **User interactions:** None

#### AEM Content Mapping Logic
- **What it does:** The `mapContent()` function translates AEM Headless content (with flat field keys like `scopes_financeEnabled`, `urls_scStartPageLink`, `consumptionEmission_wltpLink`) into the structured `VTPConfiguration` type.
- **Key files:** [src/app/FeatureHubAppDefinition.tsx](source-repos/audi-eu-vtp/packages/configuration/src/app/FeatureHubAppDefinition.tsx#L56-L170)
- **Mapped configuration groups:**
  - `scopes_*` → `VTPConfiguration.scopes` (feature toggles)
  - `urls_*` → `VTPConfiguration.urls` (external links)
  - `assets_*` → `VTPConfiguration.assets` (content references to images)
  - `financeLayer_*` → `VTPConfiguration.financeLayer` (finance layer settings)
  - `featureApps_*` → `VTPConfiguration.featureApps` (nested FA config)
  - `sortParams_*` → `VTPConfiguration.sortParams` (sort options)
  - `overWriteSorting_*` → `VTPConfiguration.overWriteSorting` (conditional sort overrides)
  - `consumptionEmission_*` → `VTPConfiguration.consumptionEmission` (WLTP/NEDC settings)
  - `scs_*` → `VTPConfiguration.scs` (SCS API path)
  - `cta` → `VTPConfiguration.cta[]` (CTA button configurations with nested options)

### Content Fragment Models
- **Key files:** [models/mainPriceModel.ts](source-repos/audi-eu-vtp/packages/configuration/models/mainPriceModel.ts), [models/priceBreakdownModel.ts](source-repos/audi-eu-vtp/packages/configuration/models/priceBreakdownModel.ts), [models/create-models.ts](source-repos/audi-eu-vtp/packages/configuration/models/create-models.ts)
- **Price Configuration Model fields:**
  - `businessModel`: enum (`dealer_stock`, `nsc_stock`, `agency_model`, `agency_model_dealer`)
  - `availability`: enum (`now`, `soon`, `date`)
  - `items`: nested Content Fragments for price fragments
- **Main Price Fragment fields:** `priceType` (retail/regular/rate/custom), `path`, `label`

### Feature Service Dependencies
- `gfa:locale-service` ^1.0.0
- `vtp-configuration-service` ^1.0.0 (consumes AND provides via `ownFeatureServiceDefinitions`)
- `audi-content-service` ^1.0.0

### Content Author Configuration
- **All VTP configuration** is authored here. Content authors control:
  - Which CTAs appear and their behavior (type, label, URL, method, target, display option, reservation filter, payment option filter, dealer ID filtering)
  - Finance enablement and options (MINIMAL/FULL/STATIC_WITH_DISCLAIMER)
  - Sort parameters and default sorting
  - Currency pattern, date pattern, mileage unit
  - Consumption/emission display (NEDC, WLTP, or both)
  - SCS market path for API calls
  - Not-found page redirect URL
  - Vehicle identification type (VIN, commissionNumber, croppedVin, numberplate)
  - Warranty plus logo URL
  - Info layer links
  - Finance layer disclaimer configuration

### Test Coverage
- [src/test/feature-app-setup.test.tsx](source-repos/audi-eu-vtp/packages/configuration/src/test/feature-app-setup.test.tsx): Validates that `FeatureApp` exists and has a default export (structural test only).

---

## 2. VTP Configuration Service (`@oneaudi/vtp-configuration-service`)

### Overview

- **Purpose:** A Feature Hub **Feature Service** (not a Feature App) that provides a shared, reactive configuration store. All VTP Feature Apps consume this service to read the `VTPConfiguration` object.
- **Entry point:** [src/index.ts](source-repos/audi-eu-vtp/packages/configuration-service/src/index.ts)
- **Version:** 0.0.1 (internal library, version not tracked actively)
- **Published as:** `@oneaudi/vtp-configuration-service` npm package

### Features Found

#### Pub/Sub Configuration Service
- **What it does:** Implements a publisher/subscriber pattern. The Configuration FA calls `setConfiguration()`, and all subscribed Feature Apps receive the configuration object via callbacks.
- **Key files:** [src/v1/configuration-service.ts](source-repos/audi-eu-vtp/packages/configuration-service/src/v1/configuration-service.ts)
- **API:**
  - `getConfiguration(): VTPConfiguration | null` — returns current config or null
  - `setConfiguration(config)` — sets config and notifies all subscribers
  - `subscribeConfiguration(callback)` — registers a listener; immediately fires if config already set
  - `unsubscribe(callback)` — removes a listener

#### SSR State Serialization
- **What it does:** During SSR, serializes configuration state so the CSR hydration can restore it without re-fetching.
- **Key files:** [src/v1/configuration-service.ts](source-repos/audi-eu-vtp/packages/configuration-service/src/v1/configuration-service.ts#L36-L67)
- **Business rules:**
  - If `s2:async-ssr-manager` AND `s2:serialized-state-manager` are present → SSR mode, register serializer
  - If only `s2:serialized-state-manager` is present (no SSR manager) → CSR after SSR, deserialize state

#### Feature Service Definition Factory
- **What it does:** `defineConfigurationService()` returns a `FeatureServiceProviderDefinition` that Feature Apps include in `ownFeatureServiceDefinitions` to make the configuration service available to themselves and other FAs on the page.
- **Key files:** [src/index.ts](source-repos/audi-eu-vtp/packages/configuration-service/src/index.ts#L273-L298)
- **Service ID:** `vtp-configuration-service`
- **Version:** `1.0.0`

### Data Models

The service exports the core `VTPConfiguration` interface and all related types:

#### `VTPConfiguration` (main type)
- **Key files:** [src/index.ts](source-repos/audi-eu-vtp/packages/configuration-service/src/index.ts#L64-L105)
- **Fields:**
  - `scopes` — feature toggles (`iframeForms`, `financeEnabled`, `hideEcom`, `financeOption`, `interpretDisclaimersTextStyle`, `hideCalculationDisclaimer`, `hideRateChangeCTA`, `hideFinanceForEcom`, `vehicleIdentification`, `forcePhoneAsPrimary`, `phoneWithNumber`)
  - `cta[]` — CTA button array
  - `urls` — external URLs (`scStartPageLink`, `eecImageUrl`, `warrantyPlusLogoURL`)
  - `assets` — content reference assets
  - `financeLayer` — finance layer settings (`hideFinanceTable`, `disclaimerListItems[]`)
  - `disclaimerType` — `Global | Product | Calculation`
  - `scs` — SCS API settings (`scsMarketPath`)
  - `infoLayerLinks[]` — links for info layers on various vehicle attributes
  - `mileageUnit`, `currency`, `currencyPattern`, `datePattern`
  - `consumptionEmission` — NEDC/WLTP display settings with blacklist and links
  - `sortParams` — sorting options and default
  - `overWriteSorting` — conditional sort override based on filter
  - `enableMandatoryAreaSearch`, `enableGoogleCookieConsent`, `deactivateSelectAllDealers`
  - `mainPriceConfiguration`, `priceConfiguration` — price display settings
  - `notFound` — redirect URL or path for 404 pages
  - `warrantyTechDataItems[]` — warranty display settings
  - `useEfficiencyImage` — boolean flag

#### `CTAConfig` (call-to-action config)
- **Key files:** [src/index.ts](source-repos/audi-eu-vtp/packages/configuration-service/src/index.ts#L170-L202)
- **Key fields:** `type` (17 CTA types including `aoz`, `bevAgency`, `contact`, `custom`, `ecom`, `glc-cash-checkout`, `glc-financing-checkout`, `glc-reservation`, `leasing`, `liteReservation`, `nws`, `finance-checkout`, `cash-checkout`, `phone`, `details`), `label`, `url`, `method` (GET/POST), `target` (same-window/new-window/open-in-layer), `displayOption` (tiles/carinfo/both/none), `reservation` (reserved/notReserved/both), `buyableOnline`, `dataProfile`, `paymentOptions`, `glcFinancingProductIds`, `filterByDealerId`

#### Enums
- `FinanceOptionType`: `MINIMAL | FULL | STATIC_WITH_DISCLAIMER`
- `VehicleIdentificationType`: `commissionNumber | vin | croppedVin | numberplate`
- `PatternType`: Currency formatting patterns (`{{currency}}{{price}}`, etc.)
- `DatePatternType`: 10 date format patterns
- `TestCycleTypesFromConfig`: `NEDC | WLTP`
- `HighlightTechdataKeys`: `PreuseDetail | GearBox | FuelType | InitialRegistrationDate | ElectricRange | ExtColor`

#### Warranty Types
- `warrantyCodeLookup` maps warranty names to i18n keys (NAP, warranty, plus, 5-years, asg-extended, asg, gwplus5-years-extended, twelve-months, clp, cpo)

---

## 3. Dealer Info Feature App (`@oneaudi/fa-vtp-dealer-info`)

### Overview

- **Purpose:** Displays dealer information for a specific vehicle on the Vehicle Detail Page (PDP). Loads dealer contact data and renders the Partner Business Card Feature App.
- **App Store ID:** `9fea9015-4853-4c93-851e-fe338f9c1c19`
- **Entry point:** [src/app/FeatureHubAppDefinition.tsx](source-repos/audi-eu-vtp/packages/dealer-info/src/app/FeatureHubAppDefinition.tsx)
- **Version:** 1.17.0
- **Design variants:** Both `alpha` and `classic` designs

### Features Found

#### Dealer Information Display (via Partner Business Card)
- **What it does:** Renders the `fa-partner-business-card` Feature App inside itself using `FeatureAppLoader`. Does NOT directly render dealer info — it delegates to another Feature App.
- **Key files:** [src/app/components/DealerInfo.tsx](source-repos/audi-eu-vtp/packages/dealer-info/src/app/components/DealerInfo.tsx)
- **Configuration passed to Partner Business Card:**
  - `partnerID` — dealer ID from vehicle data
  - `isBEVAgency` — whether dealer is BEV agency model
  - `isBEVAgencyDealer` — whether dealer is BEV agency dealer model
  - `investorSharedVehicle` — whether vehicle is investor-shared
  - `dynamicLabel` — original group name from dealer context link data
  - `vtpDealerResultsURL` — URL to dealer results page (from AEM content)
  - `dealers` — chain dealer list (id + name)
  - `nationWideSelling` — whether vehicle is sold nationwide
  - `imprint` — dealer imprint
  - `locale` — vehicle's country code
  - `registrationNumber` — dealer registration number
  - `showVariant` — always `'Show dealer name and address only'`
  - `variantConfig` — display toggles (`displayOfficialName`, `displayPhone`, `displayEmail`, `hasOuterSpacings`, `isGoogleMapLink`)
  - `vehicleBasic` — full vehicle basic data

#### Vehicle Data Fetching
- **What it does:** If `vehicleRaw` is not passed as a prop, fetches it using `useVehicleRaw()` hook from the shared package.
- **Key files:** [src/app/FeatureApp.tsx](source-repos/audi-eu-vtp/packages/dealer-info/src/app/FeatureApp.tsx)
- **Business rule:** Returns null (renders nothing) if no vehicle data available.

#### Chain Dealer Fetching
- **What it does:** Fetches chain dealers for investor-shared vehicles using `useChainDealers()` hook from shared. Only renders after dealer data is fetched.
- **Key files:** [src/app/components/DealerInfo.tsx](source-repos/audi-eu-vtp/packages/dealer-info/src/app/components/DealerInfo.tsx#L31-L41)
- **Business rule:** If chain dealer fetch errors, returns empty array. If not finished fetching, returns null.

### Business Rules
- **BEV Agency detection:** `vehicle.businessModel.code === 'agency_model'` → `isBEVAgency`; code `'agency_model_dealer'` → `isBEVAgencyDealer`
- **Partner Business Card version:** Uses `appVersion` from AEM content, falling back to `v3.2.0-rc.4`
- **Design system:** Uses `dynamicStylingHelper('app.js', 'app-alpha.js')` to select classic vs alpha bundle URL

### Feature Service Dependencies
- `audi-stockcars-store-service` 1.0.0
- `dbad:audi-i18n-service` ^1.0.0
- `gfa:locale-service` 1.0.0
- `audi-content-service` ^1.0.0
- `vtp-configuration-service` ^1.0.0
- `audi:envConfigService` 1.0.0
- Optional: `audi-tracking-service`, `gfa:layer-manager`, `s2:logger`, SSR services

### Content Author Configuration
- **`appVersion`** — version of Partner Business Card FA to load (default: `v3.2.0-rc.4`)
- **`vtpDealerResultsURL`** — URL pattern for "dealer results" link

### Test Coverage
- [src/app/FeatureApp.test.tsx](source-repos/audi-eu-vtp/packages/dealer-info/src/app/FeatureApp.test.tsx): Tests that the dealer-info element renders when vehicle data is available. Currently **skipped** (`test.skip`).

---

## 4. Soldout Feature App (`@oneaudi/fa-vtp-soldout`)

### Overview

- **Purpose:** Renders content for vehicles that are no longer available. Sets the HTTP response status to 404 via the `page-info-service`. This is a _headless_ Feature App — it renders no visible UI of its own but signals a 404 status.
- **App Store ID:** `852f355b-09dc-4797-a432-a06e6a66bff1`
- **Entry point:** [src/app/FeatureHubAppDefinition.tsx](source-repos/audi-eu-vtp/packages/soldout/src/app/FeatureHubAppDefinition.tsx)
- **Version:** 1.1.0
- **Description in App Store:** "VTP sold out page renders when a vehicle is not available & returns http 404 not found"
- **Category:** "Text" (not "Stock Car")

### Features Found

#### HTTP 404 Signal
- **What it does:** Uses `pageInfoService.setPageInfo()` to set `response.status: 404` and `response.message: 'No Content Found'`. Preserves the existing SEO description from `pageInfoService.getPageInfo()`.
- **Key files:** [src/app/FeatureApp.tsx](source-repos/audi-eu-vtp/packages/soldout/src/app/FeatureApp.tsx)
- **UI elements:** Returns `null` — renders nothing
- **Business rule:** Only sets page info if `pageInfoService` is available

### i18n Messages
- **Key files:** [src/app/i18n/messages.ts](source-repos/audi-eu-vtp/packages/soldout/src/app/i18n/messages.ts)
- **Messages defined (but not used in the headless FeatureApp):**
  - `nemo.ui.sc.details.soldout.headline`: "Sorry, this vehicle is already sold."
  - `nemo.ui.sc.details.soldout.copy`: "Further attractive vehicle offers from Audi partners await you..."
  - `nemo.ui.sc.details.soldout.button`: "Start a new search"
  - `nemo.ui.sc.details.paging.searchpage`: "Back to search page"

### API/Serverless Functions
- **Key files:** [src/api/](source-repos/audi-eu-vtp/packages/soldout/src/api/) — contains `hello-world_get.ts`, `welcome_get.unprotected.ts`, and a `serverless.yml`
- **Note:** These appear to be boilerplate serverless function stubs, not production API endpoints.

### Feature Service Dependencies
- `s2:logger` ^1.0.0
- Optional: `page-info-service` 1.0.0, `s2:serialized-state-manager` ^1.0.0

### Test Coverage
- [src/test/feature-app-setup.test.tsx](source-repos/audi-eu-vtp/packages/soldout/src/test/feature-app-setup.test.tsx): Structural test validating default export exists.

---

## 5. Shared Package (`@oneaudi/vtp-shared`)

### Overview

- **Purpose:** The core shared library used by all VTP Feature Apps (PLP, PDP, dealer-info, configuration, soldout). Contains components, hooks, contexts, utilities, types, and the "rethink" redesigned components.
- **Version:** 2.7.0
- **Entry point:** [src/index.ts](source-repos/audi-eu-vtp/packages/shared/src/index.ts)
- **Published as:** `@oneaudi/vtp-shared`, consumed as `dist/index.js`
- **Not a Feature App** — this is a library package.

### Features Found — Components

#### CTA Buttons System
- **What it does:** Renders configurable call-to-action buttons based on VTP Configuration. Supports multiple CTA types, dealer filtering, POST forms with vehicle data, finance checkout, lite reservation, custom CTAs.
- **Key files:** [src/components/ctaButtons/CTAButtons.tsx](source-repos/audi-eu-vtp/packages/shared/src/components/ctaButtons/CTAButtons.tsx), [src/components/ctaButtons/editor.ts](source-repos/audi-eu-vtp/packages/shared/src/components/ctaButtons/editor.ts)
- **CTA types supported:** `availability-notification`, `contact`, `bevAgency`, `custom`, `central-customer-hotline`, `ecom`, `finance-checkout`, `cash-checkout`, `leasing`, `nws`, `reserve`, `liteReservation`, `phone`, `details`, `dealer`, `whatsApp`, `financeOptions`, `financeInfo`
- **Business rules:**
  - CTAs filtered by `displayByDealerId()` — include/exclude lists for specific dealers
  - Phone CTA generated dynamically from vehicle dealer data
  - Lite reservation requires valid dealer email and PDF base URL
  - Reserve/availability-notification CTAs toggled by `vehicle.reservation` flag
  - Finance checkout buttons only shown when `showFinancingLink` is true and vehicle has financing data
  - Custom CTAs from config are appended with `secondary` variant

#### CTA POST Form
- **What it does:** Submits vehicle data, AOZ products, financing data, and dealer filter IDs to external systems via POST.
- **Key files:** [src/components/CTA.tsx](source-repos/audi-eu-vtp/packages/shared/src/components/CTA.tsx)
- **Data profile options:** `generic` (full data), `finance-checkout-only`, `no-finance-data`

#### Finance Components (Legacy)
- **What it does:** Renders price and financing information with market-specific templates.
- **Key files:** [src/components/finance/Finance.tsx](source-repos/audi-eu-vtp/packages/shared/src/components/finance/Finance.tsx)
- **Market-specific templates:**
  - **Japan** (`jp`) + used cars → `PriceInformationJapan`
  - **Spain** (`es`) + used cars → `PriceInformationSpain`
  - Default → `PriceInformation` + `LeasingInformation` + `TaxationInformation`
- **Sub-components:** `AvailableSoon`, `ExceptionalFinancing`, `FinancingAllInPricingElement`, `FinancingWrapper`, `TaxationInformation`, `FinanceDisclaimer` (static), dynamic finance folder, leasing folder, market-specific folders

#### Finance Components (Rethink/PDP)
- **What it does:** Redesigned finance components for the PDP.
- **Key files:** [src/rethink/pdp/components/finance/](source-repos/audi-eu-vtp/packages/shared/src/rethink/pdp/components/finance/)
- **Components:** `PriceInformationRethink`, `RateInformationRethink`, `FinanceRethink`, `FinanceLayerRethink`, `FinanceLayerParameter`, `FinanceDisclaimerMessages`, `FinanceContext`, `FinanceTracking`, `CTA`, price breakdown, finance modal, buttons, dynamic/static sub-folders

#### Vehicle Order Status
- **What it does:** Displays delivery status badges for vehicles.
- **Key files:** [src/components/VehicleOrderStatus.tsx](source-repos/audi-eu-vtp/packages/shared/src/components/VehicleOrderStatus.tsx)
- **Business rules:**
  - States 7-10 → "in-delivery"
  - States 11-12 → "at-dealer"
  - Only numeric states in the valid range are shown

#### Available Badge (Rethink)
- **What it does:** Displays availability status badge for vehicles (PLP/PDP).
- **Key files:** [src/rethink/pdp/components/AvailableBadge.tsx](source-repos/audi-eu-vtp/packages/shared/src/rethink/pdp/components/AvailableBadge.tsx)
- **Business rules:** Availability determined by `carType` (nc/uc), `businessModel`, `availableFrom`, `availableFromCode`, `reservation`, and `dealer.city`

#### Consumption/Emission Tile Elements
- **What it does:** Renders consumption and emission data on vehicle tiles.
- **Key files:** [src/components/tiles/ConsumptionTileElement.tsx](source-repos/audi-eu-vtp/packages/shared/src/components/tiles/ConsumptionTileElement.tsx), [src/components/tiles/plp/](source-repos/audi-eu-vtp/packages/shared/src/components/tiles/plp/)
- **Warranty sub-components**: [src/components/tiles/warranty/](source-repos/audi-eu-vtp/packages/shared/src/components/tiles/warranty/)

#### MyAudi Wishlist Login Layer
- **What it does:** Prompts unauthenticated users to log in to save vehicles to their MyAudi wishlist.
- **Key files:** [src/components/MyAudiWishlistLoginLayer.tsx](source-repos/audi-eu-vtp/packages/shared/src/components/MyAudiWishlistLoginLayer.tsx)
- **Features:** Stores vehicle ID in localStorage before redirecting to login, lists 3 benefits of saving vehicles
- **Local Storage key:** `myaudi_vehicle_id_to_wishlist`

#### Trade-In Section
- **What it does:** Shows a trade-in section on the PDP with a link to an external trade-in form.
- **Key files:** [src/components/TradeInSection/TradeInSection.tsx](source-repos/audi-eu-vtp/packages/shared/src/components/TradeInSection/TradeInSection.tsx)
- **Business rules:**
  - Only shown for new cars if `tradeInNc` is true, used cars if `tradeInUc` is true
  - URL formatted with vehicle data placeholders

#### Dealer Chain
- **What it does:** Displays a list of chain dealers for investor-shared vehicles.
- **Key files:** [src/components/dealerChain/DealerChain.tsx](source-repos/audi-eu-vtp/packages/shared/src/components/dealerChain/DealerChain.tsx), [src/components/dealerChain/DealerChainInfoButton.tsx](source-repos/audi-eu-vtp/packages/shared/src/components/dealerChain/DealerChainInfoButton.tsx)

#### Other UI Components
- **Lazy loading:** `LazyLoad`, `LazyImage`, `LazyPicture`
- **Buttons:** `Buttons`, `GenericButton`
- **Image slider:** `imageSlider/`
- **Slider:** generic slider component
- **Expandable:** expandable/accordion component
- **Notification Layer / Focus Layer V2:** modal/overlay components
- **Fade In Animation:** animation wrapper
- **SVG / Icon:** icon system
- **Footnotes:** reference and service context
- **OneCMS:** utilities for AEM headless content
- **Deviations Component:** product deviation display
- **Cards With Pagination:** paginated card grid
- **Efficiency Class Element:** efficiency label rendering
- **EnvKV Consumption and Emission:** German energy label components
- **Rendered Marker / RenderedMarkerPLPPDP:** Google Maps markers
- **TechData Footnote Label:** technical data with footnote references
- **Finance Checkout Popover:** popover for checkout options
- **Jumpout Link:** external link component

### Features Found — Hooks

#### API Hooks
- **`useVehicleRaw(vehicleId)`** — fetches complete vehicle data from SCS API. [src/hooks/api/useVehicleRaw.ts](source-repos/audi-eu-vtp/packages/shared/src/hooks/api/useVehicleRaw.ts)
- **`useChainDealers(vehicleId)`** — fetches chain dealers for investor-shared vehicles. [src/hooks/api/useChainDealers.ts](source-repos/audi-eu-vtp/packages/shared/src/hooks/api/useChainDealers.ts)
- **`useVehicle(vehicleId)`** — fetches basic vehicle data
- **`useVehicleUrl(vehicleId)`** — generates SCS request URL
- **`useScsUrlParts()`** — assembles SCS URL parts from configuration

#### Text/Formatting Hooks
- `useFormattedPrice` — price formatting
- `useFormattedDate` — date formatting
- `useConsumptionLabels` / `useTilesConsumptionLabels` — consumption label generation
- `useEmissionLabels` / `useTilesEmissionLabels` — emission label generation
- `usePKVConsumptionLabels` / `usePKVEmissionLabels` — PKV-specific labels
- `useCO2ClassLabels` — CO2 class labels
- `useAvailableFrom` / `useAvailableSoonLabels` — availability text
- `useBevAgencyLabels` — BEV agency-specific labels
- `useDealerLabels` — dealer label generation
- `useNationWideSellingLabels` — nationwide selling text
- `useWarrantyLabel` — warranty display text
- `useFuelTypesLookup` — fuel type name lookup
- `useDynamicAltText` — dynamic alt text for images
- `useVehicleInspectionDueDate` — inspection date formatting
- `useFilterPresetUrl` — filter preset URL generation
- `useDescription` — vehicle description hook

#### DOM Hooks
- `useBodyScrollLock` — lock body scrolling
- `useContentRendered` — detect when content is rendered
- `useDesktopOrMobileView` — responsive breakpoint detection
- `useInView` — intersection observer hook
- `useShareUrl` — generate sharing URLs

#### Other Hooks
- **`useDynamicFinancing`** — manages dynamic financing state, fetches FSAG products and default responses from CRS. [src/rethink/hooks/useDynamicFinancing.ts](source-repos/audi-eu-vtp/packages/shared/src/rethink/hooks/useDynamicFinancing.ts)
- **`usePriceConfiguration`** — resolves price template paths against vehicle data. [src/hooks/usePriceConfiguration.tsx](source-repos/audi-eu-vtp/packages/shared/src/hooks/usePriceConfiguration.tsx)
- `useTranslations` — translation hook
- `useClientServerUtils` — SSR/CSR detection

### Features Found — Contexts

#### ServicesContext
- **What it does:** Root context providing all VTP services to the component tree.
- **Key files:** [src/context/ServicesContext.tsx](source-repos/audi-eu-vtp/packages/shared/src/context/ServicesContext.tsx)
- **Provides:** `getConfiguration()`, `getEnvironmentConfig()`, `getAdditionalService()`, `getFeatureServices()`, `featureAppConfig`, `appContextId`, `audiStockcarsStoreService`, `audiMarketContextService`
- **Business rule:** Uses JavaScript `Symbol.for('__VTP_SERVICES_CONTEXT__')` to ensure a singleton context across multiple Feature Apps sharing the same React instance
- **Business rule:** Defers rendering children until `envConfig` is loaded (if `audi:envConfigService` is in dependencies)

#### StockContext
- **What it does:** Provides vehicle data context for detail page components.
- **Key files:** [src/context/StockContext.tsx](source-repos/audi-eu-vtp/packages/shared/src/context/StockContext.tsx)
- **Uses:** Same `Symbol.for()` singleton pattern

#### AuthContext (Rethink)
- **What it does:** Manages authentication state via `@oneaudi/audi-auth-service`.
- **Key files:** [src/rethink/context/AuthContext.tsx](source-repos/audi-eu-vtp/packages/shared/src/rethink/context/AuthContext.tsx)
- **Features:** Tracks `isAuthenticated`, `isAuthenticating`, subscribes to login/logout events

#### MyAudiWishlistContext (Rethink)
- **What it does:** Manages the MyAudi vehicle wishlist — fetching, adding, removing vehicles.
- **Key files:** [src/rethink/context/MyAudiWishlistContext.tsx](source-repos/audi-eu-vtp/packages/shared/src/rethink/context/MyAudiWishlistContext.tsx)
- **API:** Uses `authService.fetch('/graphql')` against `omnigraph` resource host with a GraphQL wishlist query
- **Features:** `isInWishlist()`, `add()`, `remove()` methods; notification display on add/remove

#### FilterContext (Rethink/PLP)
- **What it does:** Manages the complete filter state for the PLP including sort, filter overlay, model filter data, location search, dealer data.
- **Key files:** [src/rethink/plp/FilterContext.tsx](source-repos/audi-eu-vtp/packages/shared/src/rethink/plp/FilterContext.tsx)
- **State:** `sortParam`, `isFilterOverlayOpened`, `configuredFilters`, `formattedCheckboxFilterDataFromSCS`, `modelFilterData`, `filterData`, `wholeMarketDealerData`, `radius`, `searchedCoords`, `prevSearchedCoords`

### Features Found — Filter Components (Rethink/PLP)

- **Key files:** [src/rethink/plp/components/](source-repos/audi-eu-vtp/packages/shared/src/rethink/plp/components/)
- **Components found:**
  - `Filter` — main filter component
  - `FilterChips` / `ChipsList` — active filter chip display
  - `FilterNavigation` / `FilterNavigationBar` / `FilterNavigationItem` — filter category navigation
  - `FilterOverlay` / `FilterOverlayBody` / `FilterOverlayFooter` / `FilterOverlayNavigation` / `FilterOverlayAccordion` / `FilterOverlayAccordionSection` — full filter overlay
  - `CheckboxFilter` / `CheckboxSimple` / `CheckboxColorTile` — checkbox-based filters
  - `RangeFilter` / `RangeIncrements` — numeric range sliders
  - `ModelFilter` / `ModelFilterAccordion` / `ModelFilterAccordionSection` / `ModelImage` — model/carline filter
  - `LocationFilter` / `LocationFilterBody` / `LocationFilterHeader` — geographic location filter
  - `EquipmentFilter` — equipment/feature filter
  - `DealersListWrapper` / `DealersListItems` / `DealersListItem` / `DealersListHeader` / `DealerAccordionItemHeader` — dealer list within location filter
  - `SelectAllDealersCheckbox` — select all dealers toggle
  - `MapWrapper` / `MapBlockedWrapper` / `CookieConsentControl` / `CookieConsentRequest` — Google Maps integration with cookie consent
  - `MasOverlay` — mandatory area search overlay
  - `OptionsToolbar` — sorting/view options toolbar
  - `InfoButton` — info layer trigger button
  - `ZeroResultsBanner` — shown when no results match filters
  - `CarTypeIcons` — vehicle type icon components

### Features Found — Filter Service

- **Key files:** [src/rethink/plp/filterService.ts](source-repos/audi-eu-vtp/packages/shared/src/rethink/plp/filterService.ts)
- **Features:**
  - SEO filter resolution from URL slugs via SCS API
  - Filter initialization from URL hash (`#filter=…&preset=…`)
  - Local storage persistence of filter state
  - Carline name/group structure resolution via GraphQL
  - Model filter data preparation

### Features Found — Utilities

#### Vehicle Utilities
- **`getVehicleIdFromUrl()`** — extracts vehicle ID from URL (supports `sc_detail`, SEO URLs with `/id…/`, and `?vehicleId=` param). [src/utils/vehicle.ts](source-repos/audi-eu-vtp/packages/shared/src/utils/vehicle.ts)
- **`generatePdpUrl()`** — generates SEO-friendly PDP URLs from model year, description, and ID
- **`isPHEV()`** — detects plug-in hybrid vehicles
- **`getConsumptionValueUnitTuple()`** — extracts consumption values per test cycle

#### Image Utilities
- **`getImageUrl()`** — adapts image URLs for different image services (vtpimages.audi.de, vtpimages.audi.com, mediaservice). [src/utils/imageUtils.ts](source-repos/audi-eu-vtp/packages/shared/src/utils/imageUtils.ts)
- **`vtpImageUrlAdaption()`** — resizes VTP images by width/height parameters
- **`resolvePathFromContentReference()`** — resolves content references to paths

#### URL/Pattern Utilities
- **`formatUrl()`** — replaces placeholders in URL patterns with vehicle data. [src/utils/pattern.ts](source-repos/audi-eu-vtp/packages/shared/src/utils/pattern.ts)
  - Supported placeholders: `{{sc_vehicle_id}}`, `{{sc_car_id}}`, `{{sc_audicode}}`, `{{sc_dealer_id}}`, `{{sc_dealer_name}}`, `{{sc_raw_dealer_id}}`, `{{sc_commission_number}}`, `{{sc_market_reference}}`, `{{sc_model_code}}`, `{{sc_carline}}`, `{{sc_carline_description}}`, `{{sc_entry_url}}`, `{{sc_model_year}}`, `{{sc_vin}}`, `{{sc_stock_keeping_unit}}`, `{{sc_loan_type}}`, `{{sc_mileage}}`, `{{sc_initialregistration_month}}`, `{{sc_initialregistration_year}}`, `{{sc_initial_registration_date}}`, price placeholders via `{{sc_retail}}` etc.

#### API Utilities
- **`fetchAnythingScs()`** — core SCS API fetch function with Token header. [src/utils/api.ts](source-repos/audi-eu-vtp/packages/shared/src/utils/api.ts)
- **`fetchVehicleRaw()`** — fetch complete vehicle from SCS
- **`fetchMatches()`** — fetch matching vehicles with scoring
- **`fetchCompare()`** — fetch vehicle comparison data

#### Store Helper
- **`getLocalStoreName()`** — generates market-specific local storage key. [src/utils/storeHelper.ts](source-repos/audi-eu-vtp/packages/shared/src/utils/storeHelper.ts)
- **`getFilterPresetHash()`** — assembles URL hash from stored filters
- **`getSearchInfoForTracking()`** — extracts search info for tracking events

#### Campaign Utilities
- **`isCampaignActive()`** — checks if a campaign is within its date range. [src/utils/campaigns.ts](source-repos/audi-eu-vtp/packages/shared/src/utils/campaigns.ts)

#### Lean Checkout Utilities
- **`getCashCheckout()`** — assembles cash checkout payload for e-commerce service. [src/utils/leanCheckoutUtils.ts](source-repos/audi-eu-vtp/packages/shared/src/utils/leanCheckoutUtils.ts)
- **`getLeanCheckoutPayload()`** — assembles financing/reservation checkout payload
- **`doesButtonMatchPaymentOptions()`** — filters CTAs by vehicle payment options

#### Filter Display by Dealer ID
- **`displayByDealerId()`** — determines CTA visibility based on dealer include/exclude lists. [src/utils/displayByDealerId.ts](source-repos/audi-eu-vtp/packages/shared/src/utils/displayByDealerId.ts)

#### Consumption/Emission Label Generation
- **`getConsumptionLabels()`** — generates consumption labels considering PHEV/multi-fuel engines and NEDC/WLTP test cycles. [src/utils/getConsumptionLabels.ts](source-repos/audi-eu-vtp/packages/shared/src/utils/getConsumptionLabels.ts)
- **`getEmissionLabels()`** — generates emission labels. [src/utils/getEmissionLabels.ts](source-repos/audi-eu-vtp/packages/shared/src/utils/getEmissionLabels.ts)

#### Filter Service (Legacy)
- **`loadFilteredResults()`** — fetches paginated filter results from SCS API with sorting. [src/services/filters.ts](source-repos/audi-eu-vtp/packages/shared/src/services/filters.ts)
- **`resetFilterSortingIfNecessary()`** — resets distance sorting when geo filter is removed

#### Price Information Utilities
- **`pricePropertyLookUp()`** — resolves price template paths against vehicle data. [src/hooks/usePriceConfiguration.tsx](source-repos/audi-eu-vtp/packages/shared/src/hooks/usePriceConfiguration.tsx)
- **`getPriceComponent()`** — traverses price details with array support
- **`getTypedPrice()`** — looks up formatted price by type

#### Session Storage (Rethink)
- **Key files:** [src/rethink/utils/sessionStorage.ts](source-repos/audi-eu-vtp/packages/shared/src/rethink/utils/sessionStorage.ts)
- **`readFinance()`** / **`saveFinanceInStore()`** / **`updateFinanceInStore()`** — persist financing data in session storage per vehicle type (NC/UC)

#### CRS API Client (Rethink)
- **Key files:** [src/rethink/api/index.ts](source-repos/audi-eu-vtp/packages/shared/src/rethink/api/index.ts)
- **`getProducts()`** — fetch FSAG financing products
- **`getDefaultProduct()`** — fetch default financing calculation
- Built-in caching via `getCachedValue()`/`updateCachedValue()`

### Features Found — Feature App Initialization

- **What it does:** Shared initialization logic for all VTP Feature Apps that display vehicle details. Handles SSR/CSR lifecycle, vehicle fetching, i18n setup, error handling.
- **Key files:** [src/FeatureAppInitialization.tsx](source-repos/audi-eu-vtp/packages/shared/src/FeatureAppInitialization.tsx)
- **Three initialization paths:**
  1. **SSR** — schedules async rerender, fetches vehicle data, serializes state
  2. **CSR after SSR** — deserializes state from SSR, skips re-fetch
  3. **CSR only** — fetches everything client-side
- **Error handling:**
  - Vehicle 404 → redirects to `notFound` URL (301) or returns 404 status
  - Non-404 errors → silently hides FA (no vehicle data = no render)

### Data Models

#### `VehicleBasic` — Primary Vehicle Model
- **Key files:** [src/interfaces/vehicles.ts](source-repos/audi-eu-vtp/packages/shared/src/interfaces/vehicles.ts)
- **100+ fields** including: `id`, `audiCode`, `commissionNumber`, `vin`, `croppedVin`, `carId`, `bodyType`, `brand`, `businessModel`, `buyableOnline`, `campaigns`, `dealer`, `driveType`, `fuel`, `gearBox`, `model`, `modelYear`, `extColor`, `intColor`, `typedPrices[]`, `pictures[]`, `financing`, `used` (mileage, registration, warranty types), `reservation`, `nationWideSelling`, `investorSharedVehicle`, `warrantyInfo`, `warrantyPlus`, `envkvData`, `envkv2024`, `fodInfo`, `tradeInNc`, `tradeInUc`, `vehicleOrderStatus`, `paymentOptions[]`, `availableFrom`, `availableFromCode`, `employeeVehicle`, `leasingCar`, `liveConsulting`, `threesixtyDegrees`, `tyreLabels[]`, `batteryHealth`

#### `Dealer` Interface
- **Fields:** `id`, `name`, `city`, `country`, `email`, `street`, `zipCode`, `region`, `phoneNumbers[]`, `geoLocation`, `imprint`, `registrationNumber`, `dealerContextLinkData[]`, `dealerPersons[]`, `services`

#### `CompleteVehicleEntry` (from SCS response)
- **Combines:** `basic: VehicleBasic`, `detail: VehicleDetail`, `search: VehicleSearch`
- `VehicleDetail` includes: `additionalEquipments[]`, `features[]`, `techData`, `batteryHealth`, `financingRequest`, `financingProductsRequest`, `skuItems`

#### Compare Data Model
- **Key files:** [src/interfaces/compare.ts](source-repos/audi-eu-vtp/packages/shared/src/interfaces/compare.ts)
- **Structures:** `CompareData`, `CompareFeatures`, `CompareAttributes` (30+ attribute categories), `CompareEfficiencyConsumptionData`, `CompareTechnicalData`, `CompareSummary`, `CompareDealer`, `CompareHeader`

#### Price Configuration
- **Key files:** [src/interfaces/priceconfig.interface.ts](source-repos/audi-eu-vtp/packages/shared/src/interfaces/priceconfig.interface.ts)
- **Models:** `PriceConfigurationCF` (from AEM), `PriceGroup`, `PriceGroupItem`, `PriceItem`, `MainPriceConfiguration`
- **Features:** Business model-specific pricing, availability-based pricing, nested price fragments with footnotes

#### Finance Data Models
- **Key files:** [src/interfaces/common.ts](source-repos/audi-eu-vtp/packages/shared/src/interfaces/common.ts)
- **Types:** `FinanceStoreData`, `ConfigureFinanceComponents`, `LocalFilterStore`, `EnvConfig`, `FinanceTemplate` (enum: SPAIN_USED_CARS, DEFAULT)

#### OneCMS Content Types (PLP Filters)
- **Key files:** [src/rethink/plp/types/oneCMS.ts](source-repos/audi-eu-vtp/packages/shared/src/rethink/plp/types/oneCMS.ts)
- **Types:** `OneCMSContent`, `QuickFilters` (10 quick filter slots), `FilterCategory` (10 filter group slots), `CampaignFA`, `SortingParameter`, `CarlinePhotosFields`

#### MyAudi Wishlist Types
- **Key files:** [src/rethink/types/index.ts](source-repos/audi-eu-vtp/packages/shared/src/rethink/types/index.ts)
- **Types:** `MyAudiWishlistItem` (`id`, `vehicleId`), `MyAudiWishlistVehicleCategory` (`USED | NEW`)

### API Dependencies

| API/Service | Purpose |
|---|---|
| **SCS API** (`scs.baseUrl`) | Vehicle data, search, filter, compare, matching, dealers |
| **CRS API** (`crsServiceBaseUrl`) | FSAG financial products, rate calculations, disclaimers |
| **OneGraph** (GraphQL) | Carline names/groups for model filter |
| **Omnigraph** (via auth service) | MyAudi wishlist GraphQL queries |
| **Partner Business Card FA** | Dealer info rendering (loaded via FeatureAppLoader) |

### Feature Flags / Toggles

| Flag/Toggle | Location | What it controls |
|---|---|---|
| `scopes.financeEnabled` | VTPConfiguration | Whether finance data is shown |
| `scopes.hideEcom` | VTPConfiguration | Hide e-commerce features |
| `scopes.financeOption` | VTPConfiguration | Finance display mode (MINIMAL/FULL/STATIC_WITH_DISCLAIMER) |
| `scopes.hideFinanceForEcom` | VTPConfiguration | Hide finance when ecom active |
| `scopes.hideCalculationDisclaimer` | VTPConfiguration | Hide calculation disclaimer |
| `scopes.hideRateChangeCTA` | VTPConfiguration | Hide rate change button |
| `scopes.iframeForms` | VTPConfiguration | Use iframe forms |
| `scopes.forcePhoneAsPrimary` | VTPConfiguration | Force phone as primary CTA |
| `scopes.phoneWithNumber` | VTPConfiguration | Show phone number in CTA |
| `scopes.interpretDisclaimersTextStyle` | VTPConfiguration | Interpret HTML in disclaimers |
| `enableMandatoryAreaSearch` | VTPConfiguration | Require location before showing results |
| `enableGoogleCookieConsent` | VTPConfiguration | Show cookie consent for Google Maps |
| `deactivateSelectAllDealers` | VTPConfiguration | Hide "select all dealers" checkbox |
| `useEfficiencyImage` | VTPConfiguration | Use efficiency class image |
| `financeLayer.hideFinanceTable` | VTPConfiguration | Hide finance table in layer |

### Market/Locale Variations

- **Spain (`es`)** — Special used car price template (`PriceInformationSpain`)
- **Japan (`jp`)** — Special used car price template (`PriceInformationJapan`), grid view option
- **Currency pattern** — configurable per market (currency before/after price, symbol vs text)
- **Date pattern** — 10 format options per market
- **Mileage unit** — `km` or `miles` per market
- **SCS market path** — market-specific API path segment
- **Consumption representation** — NEDC/WLTP per market with blacklist
- **Business models** — `dealer_stock`, `nsc_stock`, `agency_model`, `agency_model_dealer` vary by market

### Tracking

- **Key files:** [src/rethink/plp/tracking.ts](source-repos/audi-eu-vtp/packages/shared/src/rethink/plp/tracking.ts), [src/components/ctaButtons/tracking.ts](source-repos/audi-eu-vtp/packages/shared/src/components/ctaButtons/tracking.ts)
- **Tracked events:** `feature_app_ready`, filter clicks, filter overlay close, CTA clicks, finance layer interactions
- **Tracking data:** `componentName: 'vtp-filter'`, search name (used/new), applied filters, results count, available categories

### Test Coverage

- **Key files:** [test/](source-repos/audi-eu-vtp/packages/shared/test/) — includes mock data, MSW handlers, test utilities, `AllTheProviders.tsx`
- **Notable test files:**
  - [src/utils/campaigns.spec.ts](source-repos/audi-eu-vtp/packages/shared/src/utils/campaigns.spec.ts)
  - [src/utils/displayByDealerId.spec.ts](source-repos/audi-eu-vtp/packages/shared/src/utils/displayByDealerId.spec.ts)
  - [src/utils/filterByList.spec.ts](source-repos/audi-eu-vtp/packages/shared/src/utils/filterByList.spec.ts)
  - [src/utils/formatCount.spec.ts](source-repos/audi-eu-vtp/packages/shared/src/utils/formatCount.spec.ts)
  - [src/utils/getConsumptionLabels.spec.ts](source-repos/audi-eu-vtp/packages/shared/src/utils/getConsumptionLabels.spec.ts)
  - [src/utils/getEmissionLabels.spec.ts](source-repos/audi-eu-vtp/packages/shared/src/utils/getEmissionLabels.spec.ts)
  - [src/utils/getRelatedProduct.spec.ts](source-repos/audi-eu-vtp/packages/shared/src/utils/getRelatedProduct.spec.ts)
  - [src/utils/leanCheckoutUtils.spec.ts](source-repos/audi-eu-vtp/packages/shared/src/utils/leanCheckoutUtils.spec.ts)
  - [src/utils/storeHelper.spec.ts](source-repos/audi-eu-vtp/packages/shared/src/utils/storeHelper.spec.ts)
  - [src/utils/vehicle.spec.ts](source-repos/audi-eu-vtp/packages/shared/src/utils/vehicle.spec.ts)
  - [src/services/filters.spec.ts](source-repos/audi-eu-vtp/packages/shared/src/services/filters.spec.ts)
  - Various rethink component tests (AvailableBadge, InfoItemsWrapper, KeyFeaturesWrapper, finance tests)
  - [src/utils/imageUtils.test.ts](source-repos/audi-eu-vtp/packages/shared/src/utils/imageUtils.test.ts)

### I18n Keys
- **Key files:** [src/i18n/translations.ts](source-repos/audi-eu-vtp/packages/shared/src/i18n/translations.ts), [src/i18n/messages.ts](source-repos/audi-eu-vtp/packages/shared/src/i18n/messages.ts)
- **Translation key patterns:**
  - `stockcars.button.audi.code.*` — Audi code display
  - `stockcars.popover.audi.code.*` — Audi code popover
  - `nemo.ui.sc.result.*` — result page actions (favorites, details)
  - `stockcars.button.favourites.*` — favorites button labels
  - `nemo.ui.sc.favorites.layer.*` — favorites confirmation layer

---

## 6. Top-Level Shared Directory (`shared/`)

### Overview

- **Purpose:** Development infrastructure — webpack config for building packages and a demo setup for local development.
- **Key files:** [shared/webpack.config.js](source-repos/audi-eu-vtp/shared/webpack.config.js), [shared/demo/](source-repos/audi-eu-vtp/shared/demo/)
- **Contents:**
  - `webpack.config.js` — shared webpack configuration using `@oneaudi/oneaudi-os-build-scripts`, alpha alias mapping, demo and SSR build scopes
  - `demo/app-content.json` — demo content payload
  - `demo/config-content.json` — demo configuration content
  - `demo/index.ssr.ejs` — SSR template
  - `aosd/` — open-source dependency declarations

### Notable Configuration

- **Alpha aliases:** Maps `@oneaudi/unified-web-components` → `@oneaudi/unified-web-alpha-components` for alpha design builds
- **Build scopes:** `modern` (module federation), `fh` (Feature Hub CSR), `ssr` (Feature Hub SSR), `demo` (development integrator)

---

## Cross-Package Dependencies Graph

```
┌──────────────────────────────┐
│  AEM Content Authors         │
│  (Content Fragments)         │
└─────────────┬────────────────┘
              │ content
              ▼
┌──────────────────────────────┐
│  fa-vtp-configuration        │  ← reads content, maps to VTPConfiguration
│  (headless FA)               │
└─────────────┬────────────────┘
              │ setConfiguration()
              ▼
┌──────────────────────────────┐
│  vtp-configuration-service   │  ← pub/sub shared service (Feature Service)
│  (Feature Service)           │
└──┬───────────┬───────────┬───┘
   │           │           │ subscribeConfiguration()
   ▼           ▼           ▼
┌──────┐  ┌────────┐  ┌──────────┐
│ PLP  │  │  PDP   │  │dealer-info│  ← all consume VTPConfiguration
│  FA  │  │  FA    │  │    FA     │
└──┬───┘  └───┬────┘  └────┬─────┘
   │          │             │
   └────┬─────┘─────────────┘
        │ imports
        ▼
┌──────────────────────────────┐
│  vtp-shared                  │  ← components, hooks, types, utils
│  (@oneaudi/vtp-shared)       │
└──────────────────────────────┘

┌──────────────────────────────┐
│  fa-vtp-soldout              │  ← standalone, sets 404 status
│  (headless FA)               │
└──────────────────────────────┘
```

---

## Coverage Gaps & Remaining Analysis

### What was covered
- All source files in configuration, configuration-service, dealer-info, and soldout packages
- All key shared package directories: components, hooks, interfaces, context, services, utils, i18n, rethink (PLP components, PDP components, hooks, context, types, utils, API)

### What remains for deeper analysis
- **Shared package test files:** Individual test files in `test/mockdata/` and `test/msw/` (mock service worker handlers) were not read — these would reveal expected SCS API response shapes
- **Rethink PLP filter utility internals:** [src/rethink/plp/utils/filterHelper.ts](source-repos/audi-eu-vtp/packages/shared/src/rethink/plp/utils/filterHelper.ts) and [src/rethink/plp/utils/locationFilterHelpers.ts](source-repos/audi-eu-vtp/packages/shared/src/rethink/plp/utils/locationFilterHelpers.ts) — complex filter logic not fully traced
- **Rethink PLP SEO utilities:** [src/rethink/plp/utils/seo/](source-repos/audi-eu-vtp/packages/shared/src/rethink/plp/utils/seo/) — SEO slug generation for filter URLs
- **Finance CRS cache implementation:** [src/components/finance/services/crs/](source-repos/audi-eu-vtp/packages/shared/src/components/finance/) — caching strategy for finance API calls
- **Individual rethink PLP component files:** 40+ filter component files were listed but not all read in detail
- **GraphQL queries:** [src/rethink/plp/one-graph-api/carlines.query.tsx](source-repos/audi-eu-vtp/packages/shared/src/rethink/plp/one-graph-api/carlines.query.tsx) — exact carline query shape
- **MAS (Mandatory Area Search) tracking:** [src/rethink/plp/MAStracking.ts](source-repos/audi-eu-vtp/packages/shared/src/rethink/plp/MAStracking.ts) — tracking for mandatory area search flow
- **Configuration editor schema:** [editor/editor.json](source-repos/audi-eu-vtp/packages/configuration/editor/editor.json) — Universal Editor schema for content authoring