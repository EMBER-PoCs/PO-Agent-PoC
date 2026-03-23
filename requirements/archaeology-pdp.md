# Code Archaeology Report: PDP (Product Details Page)

## Overview
- **Package name:** `@oneaudi/fa-vtp-pdp` (v4.30.1)
- **App Store ID:** `9877d64f-c7e8-42b7-80eb-d597ba12b311`
- **Purpose:** Comprehensive vehicle detail view — the page a user sees after clicking a vehicle from the PLP (Product Listing Page). Displays vehicle images, specs, equipment, consumption/emissions, warranties, accessories, campaigns, trade-in, dealer comments, and product deviations. Includes conversion CTAs (contact, finance checkout, reservation, phone).
- **Entry point:** [src/app/FeatureHubAppDefinition.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/FeatureHubAppDefinition.tsx) (Feature Hub registration), [src/app/FeatureApp.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/FeatureApp.tsx) (root component)
- **Team:** Mariokart (Accenture Song / oneAudi OS)
- **Design systems:** Supports both "unified" (classic) and "alpha" design libraries via `DynamicComponent`
- **Rendering:** Both CSR and SSR (`app-alpha.js`, `app-alpha-ssr.js`, `app.js`, `app-ssr.js`)
- **CMS:** AEM Headless with Content Fragment Models (OneCMS)
- **Key internal deps:** `@oneaudi/vtp-shared` (shared components, hooks, types), `@oneaudi/vtp-configuration-service` (CTA config, scopes, pricing)

## Dependencies

### Feature Service Dependencies (Required)
| Service | Version | Purpose |
|---------|---------|---------|
| `gfa:locale-service` | ^1.0.0 | Locale/language/country detection |
| `dbad:audi-i18n-service` | ^1.0.0 | Internationalization |
| `locale-service` | 1.0.0 | Locale info |
| `layer-manager` | ^2.5.0 | Layer/modal management |
| `vtp-configuration-service` | ^1.0.0 | VTP configuration (CTAs, prices, scopes) |
| `audi:envConfigService` | 1.0.0 | Environment configuration |
| `audi-footnote-reference-service` | ^3.0.0 | Footnotes for legal disclaimers |
| `audi-content-service` | ^1.0.0 | AEM content delivery |
| `audi-number-formatter-service` | ^1.0.0 | Currency/number formatting |
| `gfa:service-config-provider` | ^1.0.0 | Service config (e.g., Falcon content API) |
| `vw:authService` | ^4.0.0 | Authentication (MyAudi login) |

Source: [FeatureHubAppDefinition.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/FeatureHubAppDefinition.tsx#L38-L52)

### Feature Service Dependencies (Optional)
| Service | Version | Purpose |
|---------|---------|---------|
| `s2:logger` | ^1.0.0 | Logging |
| `s2:async-ssr-manager` | ^1.0.0 | SSR lifecycle |
| `s2:server-request` | ^1.0.0 | SSR request context |
| `audi-tracking-service` | ^2.0.0 | OneSight tracking |
| `s2:serialized-state-manager` | ^1.0.0 | SSR state serialization |
| `audi-footnote-service` | 1.0.0 | Footnote rendering |
| `navigation-service` | ^1.3.0 | Page navigation (e-commerce checkout) |
| `e-commerce-service` | ^1.0.0 | GLC lean checkout |
| `notification-display-service` | ^1.0.0 | Toast notifications |
| `page-info-service` | 1.0.0 | SEO / page info |
| `gfa:layer-manager` | 1.0.0 | GFA-specific layers |

Source: [FeatureHubAppDefinition.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/FeatureHubAppDefinition.tsx#L55-L67)

### Own Feature Service Definitions
- Defines `vtp-configuration-service` for itself. Source: [FeatureHubAppDefinition.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/FeatureHubAppDefinition.tsx#L68)

---

## Features Found

### 1. Vehicle Image Stage (Gallery)
- **What it does:** Displays vehicle images in a swipeable gallery with prev/next navigation, fullscreen layer view, and touch swipe support.
- **Key files:** [StageWrapper.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/stage/subcomponents/StageWrapper.tsx), [SwipePicture.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/stage/subcomponents/swipePicture/SwipePicture.tsx), [FullScreenPicture.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/stage/subcomponents/swipePicture/FullScreenPicture.tsx), [imagesFilter.ts](source-repos/audi-eu-vtp/packages/pdp/src/app/components/stage/utils/imagesFilter.ts)
- **UI elements:** Swipeable image container, prev/next arrow buttons, fullscreen expand button, fallback image, rendered marker overlay
- **User interactions:** Swipe left/right (touch), click prev/next arrows, click to enter fullscreen gallery, close gallery layer
- **Business logic:**
  - Images filtered by type: `vtp` (render), `photo` (dealer photos), `fallback`
  - For new cars (type=`N`): dealer photos hidden, only render images shown
  - For used cars (type=`U`): dealer photos shown alongside renders
  - `hideRenderImages` flag suppresses render images
  - Fallback images used when no other images available
  - Responsive image sizes served (400/600/900/1000/1440px) with WebP format via `mediaservice`
  - Lazy loading for non-first images (`loading="lazy"`)
  - Dynamic alt text generated via `useDynamicAltText`

### 2. 3D Webstream (AVP Integration)
- **What it does:** Enables a real-time 3D vehicle view via AVP (Audi Visualization Platform) web streaming technology.
- **Key files:** [AvpWebstreamWrapper.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/stage/subcomponents/AvpWebstreamIntegration/AvpWebstreamWrapper.tsx), [AvpWebstreamButton.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/stage/subcomponents/AvpWebstreamIntegration/AvpWebstreamButton.tsx), [StageContext.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/stage/hooks/StageContext.tsx)
- **UI elements:** "Open 3D view" button (TextButton on desktop, IconButton on mobile), full-overlay `<avp-3dws-deck>` web component
- **Conditions for display:** Only shown when ALL of: vehicle type is new car (`N`), `use3dWebstreaming` content field is `true`, `avpRessourceUrl` is set, `configId` is set
- **User interactions:** Click to open 3D view, stream connects, user rotates vehicle in 3D
- **Error handling:** `exception` event listener stops loading spinner; `update:active` event with `active=false` closes webstream
- **Z-index management:** Webstream z-index=1 (below conversion bar at z-2), fullscreen z-index=101 (above layer close button at z-100)

### 3. Vehicle Info (CarInfo)
- **What it does:** Renders vehicle headline, model description, and key specifications below the stage.
- **Key files:** [StageWrapper.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/stage/subcomponents/StageWrapper.tsx#L101) — uses shared `CarinfoWrapper`
- **UI elements:** Vehicle name, model description, key info items

### 4. Next Best Actions (NBA)
- **What it does:** Row of action buttons below the stage — configurable by content authors.
- **Key files:** [NextBestActions.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/nextBestActions/NextBestActions.tsx), [NextBestActionsButton.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/nextBestActions/NextBestActionsButton.tsx)
- **Available action types:**
  - **Favorite:** Toggle vehicle as favorite via local storage or MyAudi Wishlist API (authenticated) — [FavoriteButton.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/nextBestActions/FavoriteButton.tsx)
  - **Share:** Copy vehicle URL to clipboard with filter hash params — [ShareButton.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/nextBestActions/ShareButton.tsx)
  - **Audi Code:** Show vehicle's Audi Code in a popover with copy-to-clipboard — [AudiCodeButton.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/nextBestActions/AudiCodeButton.tsx)
- **Configuration:** Content authors configure which NBAs to show and their order via the `nbas` field (multi-select enum in content model)

### 5. Tab Navigation (Vehicle Detail Sections)
- **What it does:** Tabbed interface containing up to 6 sections, conditionally rendered based on available data.
- **Key files:** [TabNavigation.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/tabNavigation/TabNavigation.tsx)
- **Tabs (in order, conditionally shown):**

#### 5a. Equipment
- **What it does:** Shows optional (extra) and standard (serie) equipment as image cards + expandable table
- **Key files:** [Equipments.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/equipments/Equipments.tsx), [Product.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/equipments/Product.tsx), [StandardEquipmentTable.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/equipments/StandardEquipmentTable.tsx), [ModalContent.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/equipments/ModalContent.tsx)
- **UI elements:** Paginated image cards for special/standard equipment with images, modal detail layer, standard equipment grouped table, tyre label display, FOD info
- **Sub-features:** Equipment split into `withImages` and `rest` categories; additional equipment from dealer; tyre label URLs with product sheets; equipment video playback in modal
- **Condition:** Tab shown only when `vehicle.detail.features` has items

#### 5b. Technical Data
- **What it does:** Shows configurable set of technical specifications (engine, performance, dimensions, consumption data points, vehicle damages/defects, battery health)
- **Key files:** [TechnicalDataPdp.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/technicalData/TechnicalDataPdp.tsx), [TechnicalDataListPdp.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/technicalData/TechnicalDataListPdp.tsx)
- **UI elements:** Key-value list with initial view and expandable extended view (Show more/less), pollution badge, damages (repairs/defects), battery certificate, product safety info link (owners-manual-url), vehicle dimensions
- **Condition:** Tab shown when content-configurable tech data keys exist AND matching data exists (or defects/repairs/exhaust/taxBandVed exist)

#### 5c. Warranties
- **What it does:** Displays vehicle warranty cards with type, text, expiry, and downloadable conditions PDF
- **Key files:** [Warranties.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/warranties/Warranties.tsx), [Warranty.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/warranties/Warranty.tsx), [InfoButton.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/warranties/InfoButton.tsx)
- **UI elements:** Horizontally scrollable warranty cards, info (i) button opening URL in focus layer, conditions download link, date/number formatted warranty text with template substitution (`${key:date}`, `${key:number}`, `${key}`)
- **Condition:** Tab shown when `vehicle.basic.warrantyInfo.warranties` has entries
- **Special event:** Listens for `pdp:open-warranties-tab` custom DOM event to programmatically navigate to warranties tab and scroll

#### 5d. Consumption & Emissions
- **What it does:** Displays fuel consumption and CO₂ emission data. For Germany, shows the ENVKV (Energieverbrauchskennzeichnungsverordnung) label instead.
- **Key files:** [ConsumptionWrapper.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/consumption/ConsumptionWrapper.tsx), [useConsumptionData.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/hooks/useConsumptionData.tsx)
- **Market branching:** If `countryCode === 'DE'` → renders `<ENVKV>` (shared component); else → renders `<CEE>` (Consumption Emission Emissions) with configurable blacklist, test cycle types, and efficiency image
- **Configuration:** Content authors control which test cycles to display (WLTP, NEDC, or both), blacklisted items, WLTP/NEDC links, and efficiency image display
- **Condition:** Tab shown when DE market has `envkv2024` data with SVG label URLs, or non-DE market has consumption emission data

#### 5e. Dealer Comment
- **What it does:** Shows dealer-specific vehicle text or Nationwide Selling (NWS) text
- **Key files:** [DealerComment.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/dealerComment/DealerComment.tsx)
- **UI elements:** Headline, HTML content with truncation at 150 chars, expand/collapse toggle
- **Business logic:** NWS and BEV Agency vehicles get a different, i18n-sourced text; regular vehicles show `vehicle.detail.remarks`
- **Condition:** Tab shown when remarks exist (trimmed, non-empty) or vehicle is NWS/BEV Agency

#### 5f. Product Deviation
- **What it does:** Shows product deviation text (differences from standard specification) and/or downloadable PDF documents (side letters)
- **Key files:** [ProductDeviation.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/productDeviation/ProductDeviation.tsx)
- **UI elements:** HTML text with 150-char truncation + expand/collapse, or PDF download links (with `document-pdf` icon)
- **Condition:** Tab shown when `vehicle.basic.productDeviations` exists or `vehicle.detail.documents` has entries

### 6. Campaigns
- **What it does:** Displays dealer/OEM campaigns associated with the vehicle
- **Key files:** [Campaigns.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/campaigns/Campaigns.tsx), [Campaign.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/campaigns/Campaign.tsx)
- **UI elements:** Paginated cards with campaign banner image, title, info text, and "More information" link opening campaign microsite in a layer
- **Condition:** Only renders when `vehicle.basic.campaigns` exists

### 7. Accessories / Original Equipment (AOZ)
- **What it does:** Accessory/part marketplace integrated into the PDP for browsing and selecting Audi Original Accessories
- **Key files:** [AOZWrapper.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/aoz/AOZWrapper.tsx), [AozContextProvider.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/aoz/context/AozContextProvider.tsx), [AozFilter.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/aoz/subcomponents/AozFilter/AozFilter.tsx), [AozProducts.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/aoz/subcomponents/AozProducts/AozProducts.tsx), [AozProduct.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/aoz/subcomponents/AozProduct/AozProduct.tsx), [AozContact.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/aoz/subcomponents/AozContact/AozContact.tsx), [useAozSession.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/aoz/hooks/useAozSession.tsx)
- **UI elements:** Category dropdown filter ("Highlights", "All", subcategories), paginated product grid (6 or 8 per page based on screen size), product cards with image/price/checkbox selection, detail modal layer, contact button with selected products
- **User interactions:** Filter by category, select/deselect products, view product detail in layer, paginate, contact dealer with selected accessories
- **Business logic:**
  - Highlights shown as default category
  - Selected products persisted in `sessionStorage` keyed by vehicle type (`VTP_AOZ_NC` / `VTP_AOZ_UC`) and vehicle ID
  - Contact button type determined by vehicle business model (BEV agency → `bevAgency`, NWS → `nws`, regular → `contact`)
  - POST data sent to contact form includes selected AOZ products, audicode, financing data, vehicle data
  - Liquid base price display with unit conversion for fluids
- **Condition:** Only renders when `vehicle.detail.aoz` exists and vehicle has an ID and type

### 8. Trade-In Teaser
- **What it does:** Shows an editorial content teaser (Basic Teaser Feature App) for trade-in, spawned via `@oneaudi/falcon-tools`
- **Key files:** [TradeIn.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/TradeIn.tsx)
- **UI elements:** Embedded Feature App (Basic Teaser) rendered via `<Spawn>`
- **Business logic:**
  - Only shown if trade-in is enabled for the vehicle type: used cars require `tradeInUc === true`, new cars require `tradeInNc === true`
  - URL placeholder replacement with vehicle data (e.g., `{{sc_vehicle_id}}`)
- **Condition:** `hasTradeIn` must be true AND `tradeInTeaser` content model must be configured

### 9. Conversion Bar (Sticky CTA Bar)
- **What it does:** Sticky bottom bar with vehicle name, price, finance rate, and action buttons
- **Key files:** [ConversionBar.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/conversionBar/subcomponents/ConversionBar.tsx), [NavigationWrapper.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/conversionBar/subcomponents/Navigation/NavigationWrapper.tsx), [InformationWrapper.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/conversionBar/subcomponents/InformationWrapper.tsx), [RenderedButton.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/conversionBar/subcomponents/RenderedButton.tsx), [FinanceInfoConversionBar.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/conversionBar/subcomponents/FinanceInfoConversionBar.tsx), [DisplayConversionbarPrice.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/conversionBar/subcomponents/DisplayConversionbarPrice.tsx), [DisplayConversionbarRate.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/conversionBar/subcomponents/DisplayConversionbarRate.tsx)
- **UI elements:**
  - Vehicle name (symbolic carline description) + model code description
  - Retail price with currency formatting + footnote
  - Finance rate per month (if finance enabled and vehicle has financing)
  - "Change rate" link for dynamic finance recalculation
  - Price breakdown CTA
  - CTA buttons (primary + secondary in popover for >2 buttons; all visible for ≤2)
  - "I am interested" popover for overflow buttons on desktop/tablet
  - Mobile: primary button fullwidth, secondary as icon buttons
- **Sticky behavior:** Portaled to `<main>`, fixed at bottom, moves up when footer becomes visible via scroll listener
- **Button types supported** (from `RenderedButton.tsx`):
  - **GLC Lean Checkout:** `glc-reservation`, `glc-cash-checkout` — starts checkout via `eCommerceService.startCheckout()`
  - **Finance Checkout (Slim):** Opens a `FinanceCheckoutPopoverButton` dialog; POST method
  - **POST open-in-layer:** Opens form in focus layer (e.g., contact dealer) with POST data (audicode, financing, vehicle, AOZ products)
  - **POST new-window/same-window:** Submits hidden form with `inquiry_json_data`
  - **GET:** Direct link navigation to URL
  - **Phone:** `tel:` link with phone number (spaces stripped)
  - **Dealer ID filtering:** Buttons can be filtered by dealer ID (include/exclude list from `options_filterByDealerId_*`)

### 10. Back Navigation (Breadcrumb)
- **What it does:** Breadcrumb showing "Back to search page" link above the stage
- **Key files:** [Backlink.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/conversionBar/subcomponents/Navigation/Backlink.tsx)
- **Content-configurable:** `searchLink` field defines the back URL
- **UI element:** `<Breadcrumb>` component with previous page link

### 11. MyAudi Wishlist Integration
- **What it does:** When `enableMyAudiWishlist` is enabled, favorite button integrates with MyAudi authenticated wishlist API instead of local storage
- **Key files:** [FavoriteButton.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/nextBestActions/FavoriteButton.tsx)
- **Business logic:**
  - If not authenticated → opens login layer; stores vehicle ID in `localStorage` for post-login add
  - If authenticated → calls `myAudiWishlistApi.add()` / `.remove()` with vehicle type (NEW/USED)
  - Post-login: checks `MYAUDI_WISHLIST_LOCAL_STORAGE_KEY` and auto-adds vehicle after authentication

---

## Data Models

### Vehicle Data (from `@oneaudi/vtp-shared`)
- `CompleteVehicleEntry`: Top-level with `basic: VehicleBasic` and `detail: VehicleDetail`
- `VehicleBasic`: id, type (N/U), symbolicCarline, modelCode, dealer, audiCode, financing, campaigns, warrantyInfo.warranties, productDeviations, tradeInUc/tradeInNc, pollutionBadge, exhaust, taxBandVed, pictures, fallbackPictures, hideRenderImages, avpString, countryCode, envkv2024, tyreLabels, fodInfo, businessModel, availableFromCode, typedPrices, priceDetails
- `VehicleDetail`: pictures, aoz, features (equipment), techData, damages, documents, remarks, videos, vehicleDimensions, additionalEquipments, batteryHealth

### Content Fragment Model (AEM)
Root model: "Feature App - PDP" with fields:
| Field | Type | Description |
|-------|------|-------------|
| `searchLink` | text | Back navigation URL |
| `use3dWebstreaming` | boolean | Enable 3D webstream in stage |
| `shiftCurrencySymbolLeftToRight` | boolean | Move currency symbol position in finance forms |
| `includeRateInSummaryBreakdown` | boolean | Include rate inside summary breakdown |
| `showSelectedProductOptionsInBreakdown` | boolean | Include selected product options in summary |
| `avpRessourceUrl` | text | 3D webstream script URL |
| `configId` | text | 3D webstream config ID |
| `nbas` | enum (multi) | Next Best Actions selection: favorite, share, audiCode |
| `warrantyInfoLayer` | CF (multiple) | Warranty info layer config (type + URL) |
| `scsTechdataInitial` | CF (multiple) | Initial technical data selection |
| `scsTechdataExtended` | CF (multiple) | Extended technical data selection |
| `tradeInTeaser` | CF | Trade-in teaser Feature App content |
| `vtpConfiguration` | CF | Central VTP Configuration reference |

Source: [pdpModel.ts](source-repos/audi-eu-vtp/packages/pdp/models/pdpModel.ts)

Child model: "Feature App - PDP: Warranty Info Layer":
- `warrantyType`: string (matching SCS warranty type)
- `warrantyUrl`: URL for info layer

Child model: "Feature App - Configuration: Technical Data Configuration":
- `scsTechData`: enum (65+ options: acceleration, battery-energy-content-net, carbon-dioxide-emissions, electric-range, etc.)
- `additionalInfoUrl`: URL for additional info

Source: [technicalDataModel.ts](source-repos/audi-eu-vtp/packages/pdp/models/technicalDataModel.ts)

### Price Data Models
- `PriceConfigurationCF`: businessModel, availability, items[]
- `PriceGroup`: label, items[]
- `PriceGroupItem`: label, path, price, vatReclaimable, isBold, suffix, priceIncludeText, disclaimerText, items
- `PriceItem` (shared): priceType (retail/rate), price, label, priceFootnote

Source: [priceconfig.interface.ts](source-repos/audi-eu-vtp/packages/pdp/src/app/hooks/priceconfig.interface.ts)

### AOZ (Accessories) Data
- `Aoz`: highlights[], categories[]
- `AozCategory`: products[], subcategories[], text
- `AozProduct`: ean, name, differentiatingFeature, priceRegularPriceGrossConsumer, basePrice, basePriceUnit, priceCurrencyConsumer, images[]

---

## API Dependencies

The PDP does **not** directly call APIs. All data fetching is handled during `initializeFeatureApp()` in `@oneaudi/vtp-shared`, which:
1. Resolves the vehicle ID from the URL or config
2. Fetches complete vehicle data from **SCS (Stock Car Service)** via environment configuration
3. Passes `CompleteVehicleEntry` into `StockContextProvider`

- **SCS API:** Vehicle data source (basic + detail) — configured via `scs_scsMarketPath` in VTP configuration (e.g., `de/de`, `deuc/de`)
- **OneSight Tracking:** Events dispatched via `audi-tracking-service` (optional dep)
- **SEO:** `createCarlineSeoResolver` resolves SEO metadata via `page-info-service` + server request service
- **ENV Config:** Template URL pattern `https://env-config.one.audi/config/live/{key}.json`
- **Falcon Content API:** Used by Trade-In teaser for editorial content (`origin: https://oneaudi-falcon.prod.renderer.one.audi`)
- **AVP 3D Webstream:** Loads external script from `avpRessourceUrl` (e.g., `https://avp-3dws-deck.eu-west-1-web-streaming.avp.tech/avp-3dws-deck.js`)
- **Instavid:** Optional 360 spin integration loads `https://static.instavid360.com/p/latest/spin360.lite.js` (though hook exists, usage not confirmed in main codebase)
- **Auth proxy:** `authProxyUrl: '/qa-userinfo-emea/v2'` for MyAudi authentication

Note from [oneaudi-cli.json](source-repos/audi-eu-vtp/packages/pdp/oneaudi-cli.json): `useOneGraph: false` — this package does NOT use OneGraph.

---

## Business Rules

### Vehicle Type Differentiation (N=New, U=Used)
- **Image selection:** New cars show only render images; used cars show dealer photos + renders ([imagesFilter.ts](source-repos/audi-eu-vtp/packages/pdp/src/app/components/stage/utils/imagesFilter.ts#L14-L33))
- **3D Webstream:** Only available for new cars ([AvpWebstreamButton.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/stage/subcomponents/AvpWebstreamIntegration/AvpWebstreamButton.tsx#L44))
- **Trade-In:** Used cars check `tradeInUc`, new cars check `tradeInNc` ([TradeIn.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/TradeIn.tsx#L22-L23))
- **AOZ session storage:** New car = `VTP_AOZ_NC`, used car = `VTP_AOZ_UC` ([useAozSession.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/aoz/hooks/useAozSession.tsx#L7))
- **Favorites/Wishlist:** Vehicle type passed as 'NEW' or 'USED' to MyAudi API ([FavoriteButton.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/nextBestActions/FavoriteButton.tsx#L116))
- **Warranty labels:** Different i18n keys for used (`nemo.ui.sc.warranties.used.warranty.*`) vs new (`nemo.ui.sc.warranties.new.warranty.*`) ([Warranty.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/warranties/Warranty.tsx#L41-L49))
- **Tracking:** `productType` set to 'new car' or 'used car' ([Tracking.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/tracking/Tracking.tsx#L27))

### Business Model / Dealer Type Differentiation
- **NWS (Nationwide Selling):** Detected via `isNationWideSellingVehicle()` — affects dealer comment display, CTA button type
- **BEV Agency:** Detected via `isBevAgencyVehicle()` — affects dealer comment display, CTA button type (gets `bevAgency` type) ([useAozContactButton.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/aoz/hooks/useAozContactButton.tsx#L10-L13))
- **CTA Button Dealer Filtering:** Buttons can be filtered by dealer ID using `options_filterByDealerId_filterType` (include/exclude) and `options_filterByDealerId_filterList` ([RenderedButton.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/conversionBar/subcomponents/RenderedButton.tsx#L124-L140))
- **Finance gate:** Rate display requires `vehicle.basic.financing` to be truthy AND `configuration.scopes.financeEnabled` to be `true` ([DisplayConversionbarRate.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/conversionBar/subcomponents/DisplayConversionbarRate.tsx#L31-L33))

### Price Configuration
- `mainPriceConfiguration` from configuration service provides price type mappings
- Retail price and rate price extracted by `priceType` ('retail', 'rate')
- Currency formatting applied only when needed (`needsCurrencyFormatting()`)
- Currency symbol position configurable via `shiftCurrencySymbolLeftToRight`
- Footnotes attached to prices via `priceFootnote`

### URL Placeholder Replacement
- CTA button URLs support template variables like `{{sc_vehicle_id}}` — replaced via `formatUrl()` using vehicle data

---

## Market / Locale Variations

### Germany (DE) — ENVKV Compliance
- **Consumption tab:** When `countryCode === 'DE'`, renders `<ENVKV>` component (German energy consumption label regulation 2024) instead of generic `<CEE>` component ([ConsumptionWrapper.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/consumption/ConsumptionWrapper.tsx#L29-L31))
- **ENVKV 2024 data:** Uses `vehicle.basic.envkv2024` SVG label URLs ([useConsumptionData.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/hooks/useConsumptionData.tsx#L13-L18))

### Non-DE Markets
- Renders generic CEE component with configurable test cycle display (WLTP, NEDC, or both)
- Content authors control via `consumptionEmission_consumptionEmissionRepresentation` (array of cycle types) and `consumptionEmission_scsConsumptionEmissionBlacklist`
- Efficiency image display controlled by `useEfficiencyImage` + `urls_eecImageUrl`

### All Markets
- **Locale formatting:** Warranty dates formatted per market `datePattern` (from i18n); numbers formatted per locale (`formatCount(value, locale)`) ([Warranty.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/warranties/Warranty.tsx#L70))
- **Currency formatting:** Via `audi-number-formatter-service` with locale-specific configuration
- **AVP market:** Market code passed to `<avp-3dws-deck>` web component (`localeService.market`) ([AvpWebstreamWrapper.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/stage/subcomponents/AvpWebstreamIntegration/AvpWebstreamWrapper.tsx#L74))
- **i18n:** All user-visible labels come from i18n service with keys like `stockcars.*`, `nemo.ui.sc.*`
- **Country code in equipment:** Passed to `StandardEquipmentTable` for country-specific behavior ([Equipments.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/equipments/Equipments.tsx#L51))

---

## Feature Flags / Toggleable Configuration (from VTP Configuration)

No LaunchDarkly integration detected. Toggleable behaviors are content-managed via the VTP configuration service:

| Flag/Scope | Purpose | Source |
|------------|---------|--------|
| `scopes_financeEnabled` | Show/hide finance rate display and finance-related CTAs | [FinanceInfoConversionBar.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/conversionBar/subcomponents/FinanceInfoConversionBar.tsx#L27) |
| `scopes_iframeForms` | Use iframe-based forms in focus layers | [RenderedButton.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/conversionBar/subcomponents/RenderedButton.tsx#L92) |
| `scopes_financeOption` | Finance option type (MINIMAL, etc.) | Demo integrator |
| `scopes_hideCalculationDisclaimer` | Hide calculation disclaimer in finance | Demo integrator |
| `scopes_hideRateChangeCTA` | Hide "change rate" CTA | Demo integrator |
| `scopes_hideFinanceForEcom` | Hide finance for e-commerce | Demo integrator |
| `scopes_vehicleIdentification` | Vehicle ID type (commissionNumber, etc.) | Demo integrator |
| `scopes_forcePhoneAsPrimary` | Force phone button as primary CTA | Demo integrator |
| `scopes_phoneWithNumber` | Show phone number with phone button | Demo integrator |
| `scopes_interpretDisclaimersTextStyle` | Interpret HTML in disclaimer text | Demo integrator |
| `enableMyAudiWishlist` | Enable MyAudi Wishlist instead of local favorites | [FavoriteButton.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/nextBestActions/FavoriteButton.tsx#L50) |
| `use3dWebstreaming` | Enable 3D AVP webstream button in stage | [pdpModel.ts](source-repos/audi-eu-vtp/packages/pdp/models/pdpModel.ts#L14) |
| `financeLayer_hideFinanceTable` | Hide finance table in layer | Demo integrator |
| `includeRateInSummaryBreakdown` | Include rate in price summary breakdown | [pdpModel.ts](source-repos/audi-eu-vtp/packages/pdp/models/pdpModel.ts#L34) |
| `showSelectedProductOptionsInBreakdown` | Include selected product options in breakdown | [pdpModel.ts](source-repos/audi-eu-vtp/packages/pdp/models/pdpModel.ts#L42) |

---

## Edge Cases & Error Handling

### Null Vehicle Guard
- FeatureApp returns `null` when `!vehicle?.basic || !vehicle?.detail` — prevents rendering with incomplete data ([FeatureApp.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/FeatureApp.tsx#L33-L35))
- FeatureHub definition only renders App when `initialization.loadingPromise?.state.initialized` is true ([FeatureHubAppDefinition.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/FeatureHubAppDefinition.tsx#L146))

### SSR Safety
- All `window`/`document` accesses wrapped in `typeof window !== 'undefined'` checks throughout the codebase (SwipePicture, ConversionBar, RenderedButton, useFavorites, useAozSession, etc.)
- Fallback dimension for SSR: window width defaults to 400 when window undefined ([StageWrapper.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/stage/subcomponents/StageWrapper.tsx#L44))

### Image Fallbacks
- When no render or dealer images available, fallback images (key starting with `render_`) are used
- SVG fallback image imported for equipment products with no image
- Image URLs have `mediaservice` detection for responsive source sets

### Conditional Tab Rendering
- Each tab only appears when its data exists — empty/null tabs silently omitted (no error states shown)

### Comment Truncation
- Dealer comments and product deviations truncated at 150 visible characters (preserving HTML markup) with expand/collapse toggle ([useComment.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/hooks/useComment.tsx))

### AVP Error Handling
- `exception` event from AVP stops the loading spinner
- Missing `prString` or `configId` → component returns null

### Accessibility
- `aria-label` attributes on all interactive elements (buttons, links, images)
- Focus management: on scroll-to-top, focuses first visible focusable element ([FeatureApp.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/FeatureApp.tsx#L57-L72))
- Focus layer `primaryAriaLabel` and `secondaryAriaLabel` for gallery and warranty layers
- Keyboard navigation on tabs via `onKeyNavigate`
- `rel="noopener"` on external target `_blank` links
- Semantic heading hierarchy (h2 for section headings, h3 for subsections)
- `role="tablist"` and `role="tab"` for tab navigation

### Responsive Behavior
- 3 breakpoints: Mobile (<768px), Tablet (768-1023px), Desktop (≥1024px) — used for:
  - ConversionBar: popover alignment (center on mobile, right on tablet/desktop)
  - CTA buttons: full-width TextButton on mobile, IconButton on secondary mobile buttons
  - AOZ grid: 6 items per page default, 8 on large screens (≥`SYS_BREAKPOINT_2_XL`)
  - 3D button: IconButton on mobile, TextButton on desktop
  - Gallery image dimensions: 640×480 (mobile), 1024×768 (tablet), 1440×1080 (desktop)

### Conversion Bar Portal
- ConversionBar is portaled to `<main>` element via `ReactDOM.createPortal` to escape Feature App DOM constraints ([ConversionBar.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/conversionBar/subcomponents/ConversionBar.tsx#L123))

---

## Content Author Configuration

From the Content Fragment Model ([pdpModel.ts](source-repos/audi-eu-vtp/packages/pdp/models/pdpModel.ts)):

| Field | What Content Authors Control |
|-------|------------------------------|
| `searchLink` | URL for "Back to search page" breadcrumb |
| `use3dWebstreaming` | Enable/disable 3D vehicle view button |
| `avpRessourceUrl` | URL of the 3D webstream script |
| `configId` | Config ID for 3D webstream service |
| `nbas` | Which Next Best Actions to show (favorites, share, audiCode) and in what order |
| `warrantyInfoLayer` | Warranty type → info layer URL mappings (multiple) |
| `scsTechdataInitial` | Which technical data fields to show initially (ordered) |
| `scsTechdataExtended` | Which technical data fields to show in "Show more" (ordered) |
| `tradeInTeaser` | Content Fragment for Trade-In editorial teaser Feature App |
| `vtpConfiguration` | Central VTP config reference with all CTA definitions, finance settings, consumption settings |
| `shiftCurrencySymbolLeftToRight` | Currency symbol position in finance forms |
| `includeRateInSummaryBreakdown` | Rate display in breakdown |
| `showSelectedProductOptionsInBreakdown` | Product options in breakdown |

Content authors also control CTA button configuration via VTP Configuration CF:
- Button type, label, URL, method (GET/POST), target (new-window, same-window, open-in-layer), display options, dealer filtering

---

## Tracking

All tracking via `audi-tracking-service` (OneSight V2). Key tracked events:

| Event | Event Name | Component | Source |
|-------|-----------|-----------|--------|
| Feature App Ready | `vtp details` | `vtp-carInfo` | [Tracking.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/tracking/Tracking.tsx#L36) |
| Gallery Open | `vtp stage - gallery layer open` | `vtp-stage` | [Tracking.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/tracking/Tracking.tsx#L80) |
| Gallery Close | `vtp stage - gallery layer close` | `vtp-stage` | [Tracking.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/tracking/Tracking.tsx#L100) |
| Image Change | `vtp stage - change gallery image` | `vtp-stage` | [Tracking.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/tracking/Tracking.tsx#L117) |
| Thumbnail Toggle | `vtp stage - thumbnail open/close` | `vtp-stage` | [Tracking.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/tracking/Tracking.tsx#L146) |
| Favorite | `vtp add/remove from favorites` | `vtp-details` | [Tracking.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/tracking/Tracking.tsx#L200) |
| Share | `vtp share` | `vtp-details` | [Tracking.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/tracking/Tracking.tsx#L218) |
| Audi Code | `vtp audi code` | `vtp-details` | [Tracking.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/tracking/Tracking.tsx#L236) |
| Equipment Impression | `vtp standard and optional equipment - impression` | — | [equipments/tracking.ts](source-repos/audi-eu-vtp/packages/pdp/src/app/components/equipments/tracking.ts#L3) |
| Equipment Info Click | `vtp *equipment info click - layer open` | — | [equipments/tracking.ts](source-repos/audi-eu-vtp/packages/pdp/src/app/components/equipments/tracking.ts#L35) |
| Tyre Label Click | `vtp standard equipment - tyre label click` | — | [equipments/tracking.ts](source-repos/audi-eu-vtp/packages/pdp/src/app/components/equipments/tracking.ts#L89) |
| CTA Button Click | Various (via `trackClick` shared) | `vtp-carInfo` | ConversionBar RenderedButton |
| Warranty Click | warranty display events | — | [warranties/tracking.ts](source-repos/audi-eu-vtp/packages/pdp/src/app/components/warranties/tracking.ts) |
| Tech Data Drawer | drawer click | `technical-data` | [TechnicalDataPdp.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/technicalData/TechnicalDataPdp.tsx#L74) |
| "I am interested" | via `trackIamInterestedClick` | — | [InformationWrapper.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/conversionBar/subcomponents/InformationWrapper.tsx#L109) |
| GLC Checkout | via `trackGroupLeanCheckoutClick` | — | RenderedButton |

Product tracking data includes: `productId` (vehicle ID), `productName` (modelCode description), `manufacturer: 'Audi'`, `primaryCategory` (carline group code), `subCategory1` (carline code), `productType` (new/used car).

---

## Test Coverage Insights

### Key Test Files Found
| File | What It Tests |
|------|--------------|
| [FeatureApp.test.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/FeatureApp.test.tsx) | Renders all sections with vehicle, returns null without vehicle, focus-on-scroll behavior |
| [TabNavigation.test.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/tabNavigation/TabNavigation.test.tsx) | All tabs render when data present; pdp:open-warranties-tab event handling |
| [StageWrapper.test.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/stage/subcomponents/StageWrapper.test.tsx) | New/used car stage logic, image filtering, fallback images |
| [SwipePicture.test.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/stage/subcomponents/swipePicture/SwipePicture.test.tsx) | Gallery navigation, touch swipe, fullscreen |
| [AvpWebstreamButton.test.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/stage/subcomponents/AvpWebstreamIntegration/AvpWebstreamButton.test.tsx) | 3D button visibility conditions |
| [AvpWebstreamWrapper.test.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/stage/subcomponents/AvpWebstreamIntegration/AvpWebstreamWrapper.test.tsx) | Webstream event handling |
| [ConversionBar.test.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/conversionBar/subcomponents/ConversionBar.test.tsx) | Sticky bar rendering, portal behavior |
| [DisplayConversionbarPrice.test.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/conversionBar/subcomponents/DisplayConversionbarPrice.test.tsx) | Price formatting, null handling |
| [DisplayConversionbarRate.test.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/conversionBar/subcomponents/DisplayConversionbarRate.test.tsx) | Rate display conditions |
| [FinanceInfoConversionBar.test.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/conversionBar/subcomponents/FinanceInfoConversionBar.test.tsx) | Finance info display |
| [RenderedButton.test.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/conversionBar/subcomponents/RenderedButton.test.tsx) | Button type rendering, dealer filtering |
| [Equipments.test.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/equipments/Equipments.test.tsx) | Equipment rendering |
| [TechnicalDataPdp.test.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/technicalData/TechnicalDataPdp.test.tsx) | Tech data rendering |
| [DealerComment.test.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/dealerComment/DealerComment.test.tsx) | NWS vs regular vehicle comment |
| [ProductDeviation.test.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/productDeviation/ProductDeviation.test.tsx) | Deviation text and PDF display |
| [AOZWrapper.test.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/aoz/AOZWrapper.test.tsx) | AOZ conditional rendering |
| [useAozSession.test.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/aoz/hooks/useAozSession.test.tsx) | Session storage persistence |
| [useFavorites.test.ts](source-repos/audi-eu-vtp/packages/pdp/src/app/hooks/useFavorites.test.ts) | Local storage favorites |
| [useComment.test.ts](source-repos/audi-eu-vtp/packages/pdp/src/app/hooks/useComment.test.ts) | Comment truncation with markup preservation |
| [TradeIn.test.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/TradeIn.test.tsx) | Trade-in conditional rendering |
| [Tracking.test.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/components/tracking/Tracking.test.tsx) | Tracking event payloads |

---

## Integration with Other Packages

| Package | Integration Point |
|---------|------------------|
| `@oneaudi/vtp-shared` | Heavily used — StockContextProvider, ServicesContextProvider, CarinfoWrapper, FinanceProvider, CEE/ENVKV components, useButtons(), openFocusLayerV2(), trackClick(), useShareUrl(), PriceBreakdownCta, SharedChangeRateLink, formatUrl(), ExpandableElement, MyAudiWishlistContextProvider, AuthContextProvider, DynamicThemeProviderWrapper, OneCMSLayoutWrapper, FootnoteReference, etc. |
| `@oneaudi/vtp-configuration-service` | CTA button configuration, scopes, price configuration, finance settings |
| `@oneaudi/stck-store` | `getSummaryOnePointFive` for finance data in POST bodies |
| `@oneaudi/falcon-tools` | `Spawn` component for Trade-In teaser rendering, `Migration`/`Type` for CFM model generation |
| `@oneaudi/fa-one-layer` | `LayerContentHTML` for warranty info layers |
| PLP package | Implicit: `searchLink` (back navigation); `extractAndPersistFiltersFromHash` (preserves PLP filter state from URL hash); share URLs include filter params |

---

## What Was NOT Fully Analyzed

1. **`@oneaudi/vtp-shared` internals** — The shared package is the largest dependency. Components like `CarinfoWrapper`, `CEE`, `ENVKV`, `FinanceProvider`, `useButtons()`, `PriceBreakdownCta`, `openFocusLayerV2()`, and `initializeFeatureApp()` are critical but live in the shared package, not PDP. A full archaeology of `vtp-shared` would significantly expand the feature/business rule inventory.
2. **Demo integrator mock data** — [Integrator.tsx](source-repos/audi-eu-vtp/packages/pdp/src/app/demo/Integrator.tsx) contains extensive mock data (CTA configs, price configs, etc.) that reveals configuration patterns but was only partially analyzed.
3. **Webpack configuration** — Build scopes (alpha vs unified, SSR vs CSR, demo) only skimmed.
4. **Infrastructure (CDK)** — [infrastructure/](source-repos/audi-eu-vtp/packages/pdp/infrastructure/) contains AWS CDK deployment config, not analyzed.
5. **Docs package** — [docs/](source-repos/audi-eu-vtp/packages/pdp/docs/) contains a documentation Feature App, not analyzed.
6. **Content overrides** — Market-specific content override system in `content-overrides/` not analyzed for PDP-specific overrides.
7. **All test file assertions** — Test files were scanned for structure but not every assertion was read; more requirements may be embedded in test expectations.
8. **Lighthouse / performance configuration** — `.lighthouserc.json` and performance budgets not analyzed.
9. **Cypress E2E tests** — Located in the monorepo root `cypress/` folder, not in the PDP package. May contain PDP-specific E2E scenarios.
10. **Finance calculator/layer details** — The finance layer, calculator, and checkout popover logic are primarily in `vtp-shared` and `vtp-configuration-service`, with PDP just consuming them. Deep finance flow analysis requires analyzing those packages.