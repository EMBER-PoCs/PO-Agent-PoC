# Structured Requirements: EU VTP (Vehicle Transaction Pages)

**Version:** 1.0  
**Status:** Draft  
**Date:** 2026-03-22  
**Source:** Code Archaeology Reports (PLP, PDP, Supporting Packages)

---

## Stakeholders

- **End User (Vehicle Shopper):** Browses, filters, sorts, and compares vehicles; saves favorites; views vehicle details; contacts dealers; initiates purchase or finance workflows.
- **Content Author (AEM):** Configures Feature App behavior per market — CTAs, filters, sort options, finance toggles, consumption display, URLs, design variant — via AEM Content Fragment Models.
- **Dealer:** Listed on vehicle tiles and detail pages; receives contact inquiries; provides photos, comments, and campaigns for vehicles.
- **Market Manager:** Defines market-specific legal and business rules (e.g., DE EnVKV compliance, ES sorting exceptions, FR efficiency labels).
- **Audi Operations (oneAudi OS):** Deploys and maintains Feature Apps via the App Store; manages SSR rendering, infrastructure, and shared services.
- **myAudi Authenticated User:** Logs in to sync favorites/wishlist across devices via the myAudi platform.

---

## Feature Area 1: Vehicle Search & Browsing

### REQ-001: Vehicle Tile Grid Display
**As a** vehicle shopper, **I want** to see available vehicles displayed as a responsive grid of tiles, **so that** I can quickly browse the inventory.
**Priority:** Must
**Acceptance Criteria:**
- [ ] Vehicles display in a responsive CSS grid: 1 column on XS, 2 columns at MD, 2–3 at LG, 3–4 at 2XL
- [ ] Each tile shows: vehicle image gallery, carline name (h3 heading), model code description, pricing/finance info, key features, dealer info, availability badge, CTA buttons, favorites icon, and consumption/emission data
- [ ] Clicking a tile image or headline navigates to the vehicle detail page (PDP)
- [ ] Grid renders within the PLP Feature App (`@oneaudi/fa-vtp-plp`)
**Source:** archaeology-plp.md (Section 1: Vehicle Tile Grid Display)

### REQ-002: Progressive Load More Pagination
**As a** vehicle shopper, **I want** to load more vehicles incrementally by clicking a button, **so that** I can browse beyond the initial set without a full page reload.
**Priority:** Must
**Acceptance Criteria:**
- [ ] Initial page loads 12 vehicles
- [ ] "Load More" button fetches the next 12 vehicles and appends them to the grid
- [ ] "Load More" button is hidden when all vehicles have been loaded (total count ≤ fetched count)
- [ ] A progress indicator is shown during loading
- [ ] After new vehicles appear, focus moves to the first newly loaded tile (accessibility)
- [ ] "Load More" button is hidden when total count is null or undefined
**Source:** archaeology-plp.md (Section 4: Load More)

### REQ-003: Return-to-Position Scroll
**As a** vehicle shopper, **I want** to be scrolled back to the vehicle I previously clicked when I return from the detail page, **so that** I don't lose my place in the listing.
**Priority:** Should
**Acceptance Criteria:**
- [ ] When a user clicks a vehicle tile, the selected vehicle ID is stored
- [ ] When the user navigates back to the PLP, the page scrolls to the previously selected vehicle tile
**Source:** archaeology-plp.md (Business Rules: Return-to-Position Scroll)

### REQ-004: New Cars / Used Cars Toggle
**As a** vehicle shopper, **I want** to switch between new car and used car listings, **so that** I can browse the appropriate inventory type.
**Priority:** Must
**Acceptance Criteria:**
- [ ] NC/UC toggle buttons are displayed in the quick filters area
- [ ] Clicking the toggle navigates to the opposite car type page (configured via `plpOtherCarsPagePathname`)
- [ ] `isNewCarsPage` boolean determines the current context
- [ ] Market path differentiates new (`de/nc`) vs used (`de/uc`) at the API level
**Source:** archaeology-plp.md (Section 5: SwitchNCUC, Business Rules: Vehicle Type Differentiation)

### REQ-005: Zero Results State
**As a** vehicle shopper, **I want** to see a helpful message when no vehicles match my filters, **so that** I know to adjust my search criteria.
**Priority:** Must
**Acceptance Criteria:**
- [ ] When filters produce zero results, a "No results found" message and "Reset filters" button are displayed
- [ ] Clicking "Reset filters" clears all filters and re-fetches unfiltered results
- [ ] When the favorites list is empty, a zero-favorites state is shown with headline, copy, and a "New search" link
**Source:** archaeology-plp.md (Edge Cases: Zero Results State)

### REQ-006: Share Results URL
**As a** vehicle shopper, **I want** to copy a shareable URL with my current filter selections, **so that** I can share my search with others.
**Priority:** Could
**Acceptance Criteria:**
- [ ] A share button is displayed in the results bar
- [ ] Clicking share copies the current URL (including filter hash) to the clipboard
- [ ] A toast notification confirms successful copy
- [ ] Existing URL hashes are stripped before appending the filter hash
**Source:** archaeology-plp.md (Section 10: Share Results)

### REQ-007: SEO Filter Resolution
**As a** search engine crawler or user following an SEO link, **I want** filter state to be resolved from SEO-friendly URLs, **so that** filtered vehicle pages are indexable and shareable.
**Priority:** Should
**Acceptance Criteria:**
- [ ] SEO filters are resolved from the URL before the app renders initial results
- [ ] Filter data is fetched from the SCS API on initial load and on URL hash changes
- [ ] URL filter hash is cleaned up after initial filter fetch
**Source:** archaeology-plp.md (Section 13: SEO Filter Resolution)

### REQ-008: Filter Handler Only Mode
**As a** content author, **I want** to configure a PLP instance that only handles filters without rendering a visible listing, **so that** I can embed filter logic on non-listing pages.
**Priority:** Could
**Acceptance Criteria:**
- [ ] When `filterHandlerOnly` is true, the PLP renders no visible UI
- [ ] Filter state is still managed and synchronized
**Source:** archaeology-plp.md (Business Rules: Filter System, Feature Flags)

---

## Feature Area 2: Vehicle Filtering

### REQ-009: Quick Filters Panel
**As a** vehicle shopper, **I want** quick-access filters displayed as a sidebar (desktop) or fullscreen overlay (mobile), **so that** I can narrow results without navigating to a separate filter page.
**Priority:** Must
**Acceptance Criteria:**
- [ ] Desktop: quick filters render as a sidebar panel
- [ ] Mobile: quick filters render as a fullscreen overlay, triggered by a filter button in the toolbar
- [ ] Up to 12 quick filters are configurable by content authors via AEM Content Fragments
- [ ] Supported quick filter types: checkbox, color-checkbox (color tiles), car-type-icon-checkbox (body type icons), model-checkbox (carline groups), range (sliders), and location
- [ ] Filter type is determined by a suffix on the filter key (`.checkbox`, `.color-checkbox`, `.car-type-icon-checkbox`, `.model-checkbox`, `.range`, `.location`)
- [ ] Active filter selections are shown as filter chips
- [ ] An "Advanced" button opens the full filter overlay
- [ ] Quick filters are only visible in `results` context (not `favorites`)
**Source:** archaeology-plp.md (Section 5: Quick Filters)

### REQ-010: Advanced Filters Overlay
**As a** vehicle shopper, **I want** a full-screen filter overlay with categorized filters, **so that** I can apply detailed filtering criteria.
**Priority:** Must
**Acceptance Criteria:**
- [ ] Advanced filters open in a full overlay with category navigation tabs
- [ ] Up to 7 filter categories are configurable, each with up to 25 filter groups
- [ ] Filter overlay includes: navigation bar, accordion sections, footer with result count and apply/reset actions
- [ ] Supported filter types include: checkbox, range, equipment block, model, location, color, car-type-icon
- [ ] Each filter supports configurable layout width (50%, 100%, or responsive 50%/100%)
- [ ] Selecting/deselecting filters updates results dynamically
**Source:** archaeology-plp.md (Section 5, Data Models: filterCategories), archaeology-supporting.md (Shared: Filter Components)

### REQ-011: Filter Chip Display
**As a** vehicle shopper, **I want** to see my active filter selections as removable chips, **so that** I can see what's applied and quickly remove individual filters.
**Priority:** Must
**Acceptance Criteria:**
- [ ] Active filters are represented as dismissible chips
- [ ] Removing a chip deselects that filter and re-fetches results
**Source:** archaeology-supporting.md (Shared: FilterChips / ChipsList)

### REQ-012: Model / Carline Filter
**As a** vehicle shopper, **I want** to filter vehicles by model/carline using checkbox groups, **so that** I can find specific Audi models.
**Priority:** Must
**Acceptance Criteria:**
- [ ] Model checkboxes display carline groups with vehicle counts
- [ ] A "Show more options" / "Hide options" toggle controls visibility of additional models
- [ ] Responsive breakpoint logic determines initial visible count (3 on XS/S/L/XL, 6 on M)
- [ ] Supported carlines include: a1, a2, a3, a4, a5, a6, a7, a8, q2, q3, q4, q5, q6, q7, q8, q8etron, tt, r8, q6etron, etrongt
**Source:** archaeology-plp.md (Section 5: QuickModelFilter, Data Models: Carline type)

### REQ-013: Equipment Filter
**As a** vehicle shopper, **I want** to filter vehicles by equipment features, **so that** I can find vehicles with specific options.
**Priority:** Should
**Acceptance Criteria:**
- [ ] An equipment filter intro category and up to 4 sub-categories are configurable
- [ ] Equipment filters render as grouped checkbox blocks
**Source:** archaeology-plp.md (Data Models: equipmentFilter_introCategory, equipmentFilter_subCategories)

### REQ-014: Range Filters
**As a** vehicle shopper, **I want** to filter vehicles by numeric ranges (price, power, mileage, etc.), **so that** I can constrain results to my preferences.
**Priority:** Must
**Acceptance Criteria:**
- [ ] Range filters render as slider controls with min/max values
- [ ] Supported range filter dimensions include: price (retail, rate, all-in), power (kW, PS), mileage, displacement, acceleration, CO2 emission, electric range, model year, luggage capacity, initial registration year, previous owners, doors, finance amount, production year
**Source:** archaeology-plp.md (Business Rules: Filter Types)

### REQ-015: Filter Info Layers
**As a** vehicle shopper, **I want** info buttons next to certain filters that open explanatory content, **so that** I understand what each filter means.
**Priority:** Could
**Acceptance Criteria:**
- [ ] Content authors can configure info layer links per filter group
- [ ] Clicking an info button opens a layer with content from the configured URL
**Source:** archaeology-plp.md (Data Models: filterInfoLayers)

### REQ-016: Filter State Persistence
**As a** vehicle shopper, **I want** my filter selections to be persisted across page interactions, **so that** I don't lose my filters when navigating.
**Priority:** Should
**Acceptance Criteria:**
- [ ] Active filters are persisted in the URL hash and localStorage
- [ ] Returning to the PLP restores previously active filters
- [ ] Filter state is included in shared URLs
**Source:** archaeology-plp.md (Business Rules: Filter System)

### REQ-017: Mandatory Area Search
**As a** market manager, **I want** to require users to enter a location before displaying vehicle results, **so that** results are geographically relevant.
**Priority:** Could
**Acceptance Criteria:**
- [ ] When `enableMandatoryAreaSearch` is enabled, users must enter a location before seeing results
- [ ] A mandatory area search modal is triggered when the user attempts to close the filter overlay without entering a location
**Source:** archaeology-plp.md (Feature Flags: enableMandatoryAreaSearch), archaeology-supporting.md (MasOverlay)

### REQ-018: Dealer List Filter
**As a** vehicle shopper, **I want** to filter vehicles by specific dealers, **so that** I can find vehicles at my preferred dealership.
**Priority:** Should
**Acceptance Criteria:**
- [ ] A dealer list is displayed within the location filter area
- [ ] Individual dealers can be selected/deselected
- [ ] A "Select all dealers" checkbox is available (can be deactivated via `deactivateSelectAllDealers`)
- [ ] Dealer selection affects vehicle results
**Source:** archaeology-supporting.md (Shared: DealersListWrapper, SelectAllDealersCheckbox)

---

## Feature Area 3: Vehicle Sorting

### REQ-019: Sort Dropdown
**As a** vehicle shopper, **I want** to sort vehicle results by various criteria, **so that** I can find the most relevant vehicles quickly.
**Priority:** Must
**Acceptance Criteria:**
- [ ] A sort dropdown is displayed in the results bar alongside the vehicle count
- [ ] Sort options are content-configurable per market (e.g., relevance, price asc/desc, distance, campaign vehicles)
- [ ] Default sort option is content-configurable via `sortParams_defaultOption`
- [ ] Sort is disabled when there are 0 or 1 results
- [ ] Sort state is persisted in the session
- [ ] A sort info popover explains the current sorting logic
**Source:** archaeology-plp.md (Section 3: Sorting)

### REQ-020: Distance Sorting Auto-Application
**As a** vehicle shopper, **I want** results to automatically sort by distance when I set a location filter, **so that** I see the nearest vehicles first.
**Priority:** Should
**Acceptance Criteria:**
- [ ] When a location filter is set, sorting auto-switches to distance ascending (`byDistance:asc`)
- [ ] Distance sorting is only available when a location is set
- [ ] **Exception:** ES market does NOT auto-switch to distance sorting when location is set
**Source:** archaeology-plp.md (Section 3: Sorting, Business Rules: Sorting Override)

### REQ-021: Configurable Sort Override
**As a** content author, **I want** to configure a filter that auto-triggers a specific sort key, **so that** certain filter combinations present results in the optimal order.
**Priority:** Could
**Acceptance Criteria:**
- [ ] `overWriteSorting_filterToTriggerSorting` specifies which filter triggers the override
- [ ] `overWriteSorting_overrideDefaultSortingKey` specifies the target sort key
- [ ] The override does not re-trigger if the same filter value is already active
**Source:** archaeology-plp.md (Business Rules: Sorting Override Mechanism)

---

## Feature Area 4: Vehicle Detail View (PDP)

### REQ-022: Vehicle Detail Page Layout
**As a** vehicle shopper, **I want** to view comprehensive details about a specific vehicle, **so that** I can make an informed purchase decision.
**Priority:** Must
**Acceptance Criteria:**
- [ ] PDP displays: image gallery stage, vehicle headline and model info, next best actions, tabbed detail sections, campaigns, accessories, trade-in teaser, and a sticky conversion bar
- [ ] PDP renders within the `@oneaudi/fa-vtp-pdp` Feature App
- [ ] PDP does not render if vehicle basic or detail data is missing (returns null)
**Source:** archaeology-pdp.md (Overview, Section 3: Vehicle Info)

### REQ-023: Vehicle Info Header (CarInfo)
**As a** vehicle shopper, **I want** to see the vehicle name, model description, and key specs prominently below the gallery, **so that** I can immediately identify the vehicle.
**Priority:** Must
**Acceptance Criteria:**
- [ ] Vehicle symbolic carline description and model code are displayed
- [ ] Key specification items are shown below the name
**Source:** archaeology-pdp.md (Section 3: Vehicle Info)

### REQ-024: Tab Navigation for Detail Sections
**As a** vehicle shopper, **I want** to navigate between detail sections using tabs, **so that** I can explore different aspects of the vehicle.
**Priority:** Must
**Acceptance Criteria:**
- [ ] Up to 6 tabs are conditionally rendered based on data availability: Equipment, Technical Data, Warranties, Consumption & Emissions, Dealer Comment, Product Deviation
- [ ] Tabs use semantic `role="tablist"` and `role="tab"` attributes
- [ ] Keyboard navigation is supported on tabs (`onKeyNavigate`)
- [ ] Empty tabs (no data) are silently omitted
**Source:** archaeology-pdp.md (Section 5: Tab Navigation)

### REQ-025: Equipment Tab
**As a** vehicle shopper, **I want** to see optional and standard equipment for a vehicle, **so that** I understand what features are included.
**Priority:** Must
**Acceptance Criteria:**
- [ ] Optional equipment is shown as paginated image cards with detail modal layers
- [ ] Standard equipment is shown in a grouped table
- [ ] Equipment is split into items with images and items without images
- [ ] Additional dealer-provided equipment is shown separately
- [ ] Tyre label URLs and product sheets are displayed where available
- [ ] Equipment video playback is supported in the detail modal
- [ ] Tab is shown only when `vehicle.detail.features` has items
**Source:** archaeology-pdp.md (Section 5a: Equipment)

### REQ-026: Technical Data Tab
**As a** vehicle shopper, **I want** to view technical specifications for a vehicle, **so that** I can evaluate performance and dimensions.
**Priority:** Must
**Acceptance Criteria:**
- [ ] Technical data is displayed as a key-value list with an initial view and expandable extended view (Show more/less)
- [ ] Content authors configure which tech data fields appear initially vs. in the extended view (65+ available fields)
- [ ] Pollution badge, damages/defects, battery certificate, product safety info link, and vehicle dimensions are shown when available
- [ ] Tab is shown when configured tech data keys exist and matching vehicle data exists
**Source:** archaeology-pdp.md (Section 5b: Technical Data)

### REQ-027: Warranties Tab
**As a** vehicle shopper, **I want** to see warranty information for a vehicle, **so that** I understand the vehicle's warranty coverage.
**Priority:** Must
**Acceptance Criteria:**
- [ ] Warranties are displayed as horizontally scrollable cards with warranty type, text, expiry, and conditions PDF download
- [ ] Warranty text supports template substitution for date and number values (`${key:date}`, `${key:number}`, `${key}`)
- [ ] An info button opens a URL in a focus layer (configurable per warranty type)
- [ ] Warranty labels differ for new vs used vehicles (different i18n keys)
- [ ] Supported warranty types: NAP, warranty, plus, 5-years, asg-extended, asg, gwplus5-years-extended, twelve-months, clp, cpo
- [ ] Tab listens for `pdp:open-warranties-tab` custom DOM event to programmatically navigate and scroll
- [ ] Tab is shown when `vehicle.basic.warrantyInfo.warranties` has entries
**Source:** archaeology-pdp.md (Section 5c: Warranties), archaeology-supporting.md (warrantyCodeLookup)

### REQ-028: Dealer Comment Tab
**As a** vehicle shopper, **I want** to read dealer-specific comments or notes about a vehicle, **so that** I get additional context from the seller.
**Priority:** Should
**Acceptance Criteria:**
- [ ] Dealer comments display as HTML content truncated at 150 characters with expand/collapse toggle
- [ ] NWS (Nationwide Selling) and BEV Agency vehicles display an i18n-sourced text instead of dealer remarks
- [ ] Tab is shown when remarks exist (trimmed, non-empty) or the vehicle is NWS/BEV Agency
**Source:** archaeology-pdp.md (Section 5e: Dealer Comment)

### REQ-029: Product Deviation Tab
**As a** vehicle shopper, **I want** to see product deviation information (differences from standard specification), **so that** I understand any non-standard features.
**Priority:** Should
**Acceptance Criteria:**
- [ ] Product deviation text is shown with 150-character truncation and expand/collapse toggle
- [ ] Downloadable PDF documents (side letters) are shown with document-pdf icons
- [ ] Tab is shown when `vehicle.basic.productDeviations` exists or `vehicle.detail.documents` has entries
**Source:** archaeology-pdp.md (Section 5f: Product Deviation)

### REQ-030: Campaigns Display
**As a** vehicle shopper, **I want** to see active campaigns associated with a vehicle, **so that** I can take advantage of promotions.
**Priority:** Should
**Acceptance Criteria:**
- [ ] Campaigns are displayed as paginated cards with banner image, title, info text, and "More information" link
- [ ] The "More information" link opens the campaign microsite in a layer
- [ ] Campaigns only render when `vehicle.basic.campaigns` exists
- [ ] Campaign date range determines if a campaign is active
**Source:** archaeology-pdp.md (Section 6: Campaigns), archaeology-supporting.md (isCampaignActive)

### REQ-031: Accessories / Original Equipment (AOZ)
**As a** vehicle shopper, **I want** to browse and select Audi Original Accessories for a vehicle, **so that** I can request accessories with my dealer inquiry.
**Priority:** Should
**Acceptance Criteria:**
- [ ] Accessories are displayed as a filterable, paginated product grid (6 items per page default, 8 on large screens)
- [ ] Filter allows selecting by category (Highlights, All, subcategories)
- [ ] Highlights are shown as the default category
- [ ] Product cards show image, name, price with currency, and a checkbox for selection
- [ ] A detail modal layer shows additional product information
- [ ] Selected products are persisted in `sessionStorage` keyed by vehicle type and vehicle ID
- [ ] A contact button sends selected AOZ products along with vehicle data to the contact form
- [ ] Liquid products show base price with unit conversion
- [ ] Section only renders when `vehicle.detail.aoz` exists and the vehicle has an ID and type
**Source:** archaeology-pdp.md (Section 7: Accessories)

### REQ-032: Trade-In Teaser
**As a** vehicle shopper, **I want** to see a trade-in offer teaser on the vehicle detail page, **so that** I can explore trading in my current vehicle.
**Priority:** Could
**Acceptance Criteria:**
- [ ] Trade-in teaser is rendered as an embedded Basic Teaser Feature App via `<Spawn>`
- [ ] For used cars: shown only if `tradeInUc === true`
- [ ] For new cars: shown only if `tradeInNc === true`
- [ ] URL placeholders (e.g., `{{sc_vehicle_id}}`) are replaced with vehicle data
- [ ] Trade-in teaser content model must be configured by content authors
**Source:** archaeology-pdp.md (Section 8: Trade-In Teaser)

### REQ-033: Back Navigation Breadcrumb
**As a** vehicle shopper, **I want** a "Back to search page" breadcrumb on the detail page, **so that** I can return to the listing easily.
**Priority:** Must
**Acceptance Criteria:**
- [ ] A breadcrumb component displays a link back to the search page
- [ ] The back URL is content-configurable via the `searchLink` field
**Source:** archaeology-pdp.md (Section 10: Back Navigation)

### REQ-034: Next Best Actions (NBA)
**As a** vehicle shopper, **I want** quick action buttons below the image gallery (Favorite, Share, Audi Code), **so that** I can quickly perform common actions.
**Priority:** Should
**Acceptance Criteria:**
- [ ] A row of action buttons is displayed below the stage
- [ ] Available actions: Favorite (toggle), Share (copy URL to clipboard), Audi Code (show code in popover with copy)
- [ ] Content authors configure which NBAs to show and their order via the `nbas` field
**Source:** archaeology-pdp.md (Section 4: Next Best Actions)

---

## Feature Area 5: Vehicle Image Gallery

### REQ-035: PLP Tile Image Gallery
**As a** vehicle shopper, **I want** to swipe through vehicle images on listing tiles, **so that** I can preview the vehicle from multiple angles.
**Priority:** Must
**Acceptance Criteria:**
- [ ] Each tile shows an image slider with pagination arrows (Previous/Next) and a counter (e.g., "1/5")
- [ ] Swipe and arrow navigation are supported
- [ ] Images support responsive `srcSet` and WebP format via media service
- [ ] Lazy loading is used for off-screen images, with eager loading for adjacent slides
- [ ] Clicking a gallery image navigates to the detail page
**Source:** archaeology-plp.md (Section 2: Vehicle Image Gallery)

### REQ-036: PLP Image Selection Logic
**As a** vehicle shopper, **I want** to see the most appropriate images for each vehicle type, **so that** I get an accurate visual representation.
**Priority:** Must
**Acceptance Criteria:**
- [ ] New cars (type `N`): show only render images
- [ ] Used cars (type `U`): show dealer photos alongside render images
- [ ] `hideRenderImages` flag suppresses render images when set
- [ ] `nationWideSelling` vehicles never show dealer images
- [ ] Fallback images are used when no render or dealer images are available
**Source:** archaeology-plp.md (Section 2: Business Rules), archaeology-pdp.md (Section 1: Business Logic)

### REQ-037: PDP Full Image Gallery
**As a** vehicle shopper, **I want** a large, swipeable image gallery on the detail page with fullscreen view, **so that** I can examine the vehicle closely.
**Priority:** Must
**Acceptance Criteria:**
- [ ] Gallery supports swipe (touch) and prev/next arrow navigation
- [ ] A fullscreen expand button opens a fullscreen gallery layer
- [ ] Responsive image sizes are served (400/600/900/1000/1440px) with WebP format
- [ ] Lazy loading applies to non-first images
- [ ] Dynamic alt text is generated for accessibility (`useDynamicAltText`)
- [ ] Fallback images are shown when no other images are available
**Source:** archaeology-pdp.md (Section 1: Vehicle Image Stage)

### REQ-038: 3D Webstream (AVP Integration)
**As a** vehicle shopper, **I want** to view a real-time 3D rendering of a new vehicle, **so that** I can explore the vehicle interactively.
**Priority:** Could
**Acceptance Criteria:**
- [ ] A "3D view" button is displayed (TextButton on desktop, IconButton on mobile)
- [ ] 3D view is available only when ALL conditions are met: vehicle is new car (type `N`), `use3dWebstreaming` is true, `avpRessourceUrl` is set, `configId` is set
- [ ] The 3D viewer loads as a web component (`<avp-3dws-deck>`) in a full overlay
- [ ] Error events from AVP stop the loading spinner
- [ ] Z-index management: webstream z-index=1 (below conversion bar), fullscreen z-index=101
**Source:** archaeology-pdp.md (Section 2: 3D Webstream)

---

## Feature Area 6: Favorites / Wishlist

### REQ-039: Local Favorites (PLP)
**As a** vehicle shopper, **I want** to mark vehicles as favorites and view them on a dedicated page, **so that** I can compare my shortlisted vehicles.
**Priority:** Must
**Acceptance Criteria:**
- [ ] A favorite icon button (heart) is available on each vehicle tile
- [ ] Clicking adds/removes the vehicle from favorites (Redux store `FAVORITE_VEHICLES`)
- [ ] A favorites link with count badge is shown in the toolbar
- [ ] The favorites page shows saved vehicles using the same tile format
- [ ] In favorites view, removing a favorite shows a Yes/No confirmation overlay
- [ ] In results view, removal is immediate (no confirmation)
- [ ] Zero favorites shows an empty state with headline, description, and "New search" link
**Source:** archaeology-plp.md (Section 7: Favorites System)

### REQ-040: myAudi Cloud Wishlist
**As a** myAudi authenticated user, **I want** my favorites to be synced to my myAudi account, **so that** I can access my saved vehicles across devices.
**Priority:** Should
**Acceptance Criteria:**
- [ ] When `enableMyAudiWishlist` is true, favorites use cloud sync via myAudi Wishlist API (GraphQL via Omnigraph)
- [ ] Unauthenticated users clicking favorite see a login layer prompting authentication
- [ ] Vehicle ID is stored in localStorage before login redirect; after login, the vehicle is auto-added
- [ ] Vehicle type (NEW/USED) is passed to the wishlist API
- [ ] Toast notifications confirm add/remove actions
- [ ] When `enableMyAudiWishlist` is true, the local favorites link in the toolbar is hidden
**Source:** archaeology-plp.md (Section 7), archaeology-pdp.md (Section 11: MyAudi Wishlist), archaeology-supporting.md (MyAudiWishlistContext)

### REQ-041: PDP Favorite Button
**As a** vehicle shopper, **I want** to favorite a vehicle from the detail page, **so that** I can save it without returning to the listing.
**Priority:** Must
**Acceptance Criteria:**
- [ ] A favorite action is available via Next Best Actions on the PDP
- [ ] Behavior matches PLP: local storage or myAudi wishlist depending on `enableMyAudiWishlist`
- [ ] Post-login auto-add is supported (checks localStorage key)
**Source:** archaeology-pdp.md (Section 4: NBA - Favorite, Section 11: MyAudi Wishlist)

---

## Feature Area 7: Finance & Pricing Display

### REQ-042: Retail Price Display
**As a** vehicle shopper, **I want** to see the retail price of a vehicle, **so that** I know how much it costs.
**Priority:** Must
**Acceptance Criteria:**
- [ ] Retail price is displayed on PLP tiles and in the PDP conversion bar
- [ ] Currency formatting is market-specific (via `currencyPattern` and `audi-number-formatter-service`)
- [ ] Currency symbol position is configurable (`shiftCurrencySymbolLeftToRight`)
- [ ] Price footnotes are attached where configured (`priceFootnote`)
**Source:** archaeology-pdp.md (Section 9: Conversion Bar), archaeology-supporting.md (Price Configuration)

### REQ-043: Finance Rate Display
**As a** vehicle shopper, **I want** to see the monthly finance rate for a vehicle, **so that** I can evaluate affordability.
**Priority:** Must
**Acceptance Criteria:**
- [ ] Finance rate per month is shown when `scopes_financeEnabled` is true AND the vehicle has financing data
- [ ] A "Change rate" link allows dynamic finance recalculation (unless `scopes_hideRateChangeCTA` is true)
- [ ] Finance is hidden for e-commerce vehicles when `scopes_hideFinanceForEcom` is true
**Source:** archaeology-pdp.md (Section 9: ConversionBar), archaeology-supporting.md (Feature Flags)

### REQ-044: Price Breakdown
**As a** vehicle shopper, **I want** to see a detailed breakdown of the vehicle price, **so that** I understand all cost components.
**Priority:** Should
**Acceptance Criteria:**
- [ ] A price breakdown CTA is available in the conversion bar
- [ ] Price breakdown configuration is defined per business model and availability status
- [ ] Rate can be included in the summary breakdown (`includeRateInSummaryBreakdown`)
- [ ] Selected product options can be included in the breakdown (`showSelectedProductOptionsInBreakdown`)
**Source:** archaeology-pdp.md (Section 9: Conversion Bar, Content Author Configuration)

### REQ-045: Finance Option Modes
**As a** content author, **I want** to configure the level of finance detail shown, **so that** I can match market/legal requirements.
**Priority:** Must
**Acceptance Criteria:**
- [ ] Three finance option modes are supported: `MINIMAL`, `FULL`, `STATIC_WITH_DISCLAIMER`
- [ ] Finance layer disclaimer items are configurable
- [ ] Finance table can be hidden in the layer (`financeLayer_hideFinanceTable`)
- [ ] Calculation disclaimer can be hidden (`scopes_hideCalculationDisclaimer`)
**Source:** archaeology-supporting.md (VTPConfiguration: scopes, financeLayer)

### REQ-046: Dynamic Financing
**As a** vehicle shopper, **I want** to recalculate financing terms with different parameters, **so that** I can explore payment options.
**Priority:** Should
**Acceptance Criteria:**
- [ ] Dynamic financing fetches FSAG products and default calculations from the CRS API
- [ ] Financing data is persisted in session storage per vehicle type (NC/UC)
- [ ] Finance results are cached to minimize API calls
**Source:** archaeology-supporting.md (useDynamicFinancing, CRS API Client, Session Storage)

### REQ-047: Market-Specific Price Templates
**As a** market manager, **I want** price and finance display to adapt to market-specific formats, **so that** pricing complies with local conventions.
**Priority:** Must
**Acceptance Criteria:**
- [ ] Spain (ES) used cars use a dedicated price template (`PriceInformationSpain`)
- [ ] Japan (JP) used cars use a dedicated price template (`PriceInformationJapan`)
- [ ] All other markets use the default price/leasing/taxation template
- [ ] Currency, date, and number formatting follow market-specific patterns
**Source:** archaeology-supporting.md (Shared: Finance Components - Market-specific templates)

---

## Feature Area 8: Consumption & Emissions

### REQ-048: Standard Consumption & Emission Display (Non-DE Markets)
**As a** vehicle shopper, **I want** to see consumption and emission data for a vehicle, **so that** I can evaluate its environmental impact.
**Priority:** Must
**Acceptance Criteria:**
- [ ] WLTP data is prioritized over NEDC; both can be shown if configured
- [ ] Content authors control which test cycles to display (`consumptionEmission_consumptionEmissionRepresentation`)
- [ ] A blacklist can hide certain emission types (`consumptionEmission_scsConsumptionEmissionBlacklist`)
- [ ] WLTP and NEDC info links are configurable
- [ ] On PLP tiles, an inline summary is shown with a "Details" link to open a layer
- [ ] The detail layer shows full consumption/emission values, efficiency class image, and footnote references
- [ ] Efficiency class image display is controlled by `useEfficiencyImage` and `urls_eecImageUrl`
- [ ] PHEV vehicles receive special consumption label handling (multi-fuel)
**Source:** archaeology-plp.md (Section 9), archaeology-pdp.md (Section 5d), archaeology-supporting.md (getConsumptionLabels, getEmissionLabels)

### REQ-049: German EnVKV Compliance (DE Market)
**As a** German market user, **I want** to see the legally required EnVKV energy label, **so that** consumption data meets German regulatory requirements.
**Priority:** Must
**Acceptance Criteria:**
- [ ] When `countryCode === 'DE'`, the `<ENVKV>` component renders instead of the standard CEE component
- [ ] ENVKV 2024 data uses SVG label URLs from `vehicle.basic.envkv2024`
- [ ] This applies on both PLP tiles and the PDP consumption tab
- [ ] Tab is shown when the vehicle has `envkv2024` data with SVG label URLs
**Source:** archaeology-plp.md (Section 9), archaeology-pdp.md (Section 5d, Market Variations: Germany)

### REQ-050: French Market Efficiency Label
**As a** French market user, **I want** to see the French-specific efficiency label, **so that** the display meets French regulatory requirements.
**Priority:** Must
**Acceptance Criteria:**
- [ ] When `country === 'fr'`, the efficiency display uses `vehicle.eecLabel` instead of the computed SVG path
**Source:** archaeology-plp.md (Market Variations: FR market)

---

## Feature Area 9: Dealer Information Display

### REQ-051: Dealer Info on PLP Tiles
**As a** vehicle shopper, **I want** to see the dealer name on each vehicle tile, **so that** I know which dealer offers the vehicle.
**Priority:** Must
**Acceptance Criteria:**
- [ ] Dealer name is displayed on each vehicle tile via the `DealerInfoRethink` shared component
**Source:** archaeology-plp.md (Section 1: UI elements, Integration table)

### REQ-052: Dealer Info Feature App (PDP)
**As a** vehicle shopper, **I want** to see detailed dealer information on the vehicle detail page, **so that** I can contact or visit the dealer.
**Priority:** Must
**Acceptance Criteria:**
- [ ] Dealer info is displayed by embedding the `fa-partner-business-card` Feature App
- [ ] Configuration passed includes: partner ID, BEV agency flags, investor-shared status, dynamic label, dealer results URL, chain dealers, NWS flag, imprint, locale, registration number
- [ ] Display variant is "Show dealer name and address only" with toggleable options for official name, phone, email, and Google Map link
- [ ] Returns null (renders nothing) if no vehicle data is available
**Source:** archaeology-supporting.md (Section 3: Dealer Info Feature App)

### REQ-053: Chain Dealer Display
**As a** vehicle shopper, **I want** to see chain dealers for investor-shared vehicles, **so that** I know all locations that can fulfill the purchase.
**Priority:** Should
**Acceptance Criteria:**
- [ ] Chain dealers are fetched for investor-shared vehicles via the `useChainDealers()` hook
- [ ] Chain dealer list (ID + name) is passed to the Partner Business Card
- [ ] If chain dealer fetch errors, an empty array is used gracefully
**Source:** archaeology-supporting.md (Section 3: Chain Dealer Fetching, Shared: Dealer Chain)

---

## Feature Area 10: Contact / CTA System

### REQ-054: Configurable CTA Buttons (PLP Tiles)
**As a** vehicle shopper, **I want** actionable buttons on each vehicle tile, **so that** I can take the next step (view details, contact dealer, etc.).
**Priority:** Must
**Acceptance Criteria:**
- [ ] Up to 2 CTA buttons per tile (primary + secondary), plus a favorite icon button
- [ ] CTA configuration is loaded from `vtp-configuration-service`
- [ ] If no CTAs are configured, a fallback "Go to Details" button is shown
- [ ] Supported CTA types: `details`, `contact`, `nws`, `bevAgency`, `ecom`, `phone`, `central-customer-hotline`
- [ ] Phone CTA can show the phone number directly (`phoneWithNumber` flag)
- [ ] Phone CTA can be forced as primary (`scopes_forcePhoneAsPrimary`)
**Source:** archaeology-plp.md (Section 11: CTA Buttons)

### REQ-055: Conversion Bar (PDP Sticky CTA)
**As a** vehicle shopper, **I want** a sticky bottom bar with the vehicle name, price, and action buttons on the detail page, **so that** I can always access conversion actions while scrolling.
**Priority:** Must
**Acceptance Criteria:**
- [ ] Conversion bar is sticky at the bottom of the viewport, portaled to `<main>` via `ReactDOM.createPortal`
- [ ] Bar displays: vehicle name (carline + model code), retail price with currency, finance rate per month (if applicable), CTA buttons
- [ ] For ≤2 buttons: all are visible; for >2: primary is visible, secondary buttons in an "I am interested" popover
- [ ] Mobile: primary button full-width, secondary as icon buttons
- [ ] Bar moves up when the footer scrolls into view
**Source:** archaeology-pdp.md (Section 9: Conversion Bar)

### REQ-056: CTA Button Types (Full Catalog)
**As a** content author, **I want** to configure from a comprehensive set of CTA types, **so that** I can match the conversion flow to market needs.
**Priority:** Must
**Acceptance Criteria:**
- [ ] Supported CTA types: `availability-notification`, `contact`, `bevAgency`, `custom`, `central-customer-hotline`, `ecom`, `finance-checkout`, `cash-checkout`, `leasing`, `nws`, `reserve`, `liteReservation`, `phone`, `details`, `dealer`, `whatsApp`, `financeOptions`, `financeInfo`, `glc-reservation`, `glc-cash-checkout`, `glc-financing-checkout`, `aoz`
- [ ] Each CTA has configurable: label, URL, method (GET/POST), target (same-window/new-window/open-in-layer), display option (tiles/carinfo/both/none)
- [ ] Reserve/availability-notification CTAs toggle based on `vehicle.reservation` flag
- [ ] Finance checkout buttons only shown when finance is enabled and the vehicle has financing data
- [ ] CTAs can be filtered by `buyableOnline` and `paymentOptions`
**Source:** archaeology-supporting.md (CTAConfig, CTA Button System), archaeology-pdp.md (Section 9: Button Types)

### REQ-057: CTA Dealer ID Filtering
**As a** content author, **I want** to show or hide CTA buttons based on the vehicle's dealer, **so that** dealer-specific conversion paths are respected.
**Priority:** Should
**Acceptance Criteria:**
- [ ] CTA buttons support include/exclude dealer ID lists (`options_filterByDealerId_filterType`, `options_filterByDealerId_filterList`)
- [ ] Included dealer IDs: button only shown for those dealers
- [ ] Excluded dealer IDs: button hidden for those dealers
**Source:** archaeology-pdp.md (Business Rules: CTA Button Dealer Filtering), archaeology-supporting.md (displayByDealerId)

### REQ-058: CTA POST Data Submission
**As a** vehicle shopper, **I want** CTA buttons to submit relevant vehicle and context data when contacting a dealer, **so that** the dealer receives all necessary inquiry information.
**Priority:** Must
**Acceptance Criteria:**
- [ ] POST CTAs submit hidden forms with vehicle data, audicode, financing data, AOZ selected products, and dealer filter IDs
- [ ] Data profile options control what data is included: `generic` (full), `finance-checkout-only`, `no-finance-data`
- [ ] URL placeholders (e.g., `{{sc_vehicle_id}}`, `{{sc_audicode}}`, `{{sc_dealer_id}}`) are replaced with actual vehicle data
- [ ] CTAs can open in layer (iframe), new window, or same window
**Source:** archaeology-pdp.md (Section 9: RenderedButton), archaeology-supporting.md (CTA POST Form, formatUrl placeholders)

### REQ-059: GLC Lean Checkout
**As a** vehicle shopper, **I want** to start an e-commerce checkout directly from the PDP, **so that** I can purchase or reserve the vehicle online.
**Priority:** Should
**Acceptance Criteria:**
- [ ] `glc-reservation`, `glc-cash-checkout`, and `glc-financing-checkout` CTAs start checkout via `eCommerceService.startCheckout()`
- [ ] Checkout payload includes vehicle data, financing information, and selected accessories
- [ ] GLC financing product IDs are configurable per CTA
**Source:** archaeology-pdp.md (Section 9: GLC Lean Checkout), archaeology-supporting.md (leanCheckoutUtils)

### REQ-060: Lite Reservation
**As a** vehicle shopper, **I want** to make a lightweight reservation for a vehicle, **so that** I can express interest without full checkout.
**Priority:** Could
**Acceptance Criteria:**
- [ ] Lite reservation requires a valid dealer email and PDF base URL
- [ ] Reservation CTA is only shown when the vehicle supports reservation
**Source:** archaeology-supporting.md (CTA Buttons System: liteReservation)

---

## Feature Area 11: Vehicle Sold Out Experience

### REQ-061: Sold Out Page (404)
**As a** vehicle shopper, **I want** to see a helpful message when a vehicle is no longer available, **so that** I can start a new search.
**Priority:** Must
**Acceptance Criteria:**
- [ ] When a vehicle is no longer available, the `fa-vtp-soldout` Feature App sets the HTTP status to 404
- [ ] The soldout FA uses `page-info-service` to set `response.status: 404` and `response.message: 'No Content Found'`
- [ ] Existing SEO description is preserved
- [ ] The FA itself renders no visible UI (headless); the 404 page content is rendered by AEM
- [ ] i18n messages define: headline ("Sorry, this vehicle is already sold"), copy text, "Start a new search" button, and "Back to search page" link
**Source:** archaeology-supporting.md (Section 4: Soldout Feature App)

### REQ-062: Vehicle Not Found Redirect
**As a** vehicle shopper, **I want** to be redirected to a configured page when accessed vehicle data returns a 404, **so that** I see a proper error page.
**Priority:** Must
**Acceptance Criteria:**
- [ ] When the SCS API returns 404 for a vehicle, the app redirects to the `notFound` URL (301 redirect) or returns 404 status
- [ ] Non-404 API errors cause the Feature App to silently hide (no vehicle data = no render)
**Source:** archaeology-supporting.md (FeatureAppInitialization: Error handling)

---

## Feature Area 12: Configuration Feature App

### REQ-063: Headless Configuration Injection
**As a** VTP system, **I want** the configuration Feature App to read AEM content and broadcast it to all VTP Feature Apps, **so that** all apps share consistent configuration.
**Priority:** Must
**Acceptance Criteria:**
- [ ] `fa-vtp-configuration` reads AEM Content Fragments via `audi-content-service`
- [ ] Content is mapped from flat AEM field keys (e.g., `scopes_financeEnabled`) to a nested `VTPConfiguration` object
- [ ] Configuration is broadcast via `configService.setConfiguration()` to all subscribers
- [ ] The FA renders no visible UI (returns null)
**Source:** archaeology-supporting.md (Section 1: VTP Configuration Feature App)

### REQ-064: Configuration Service Pub/Sub
**As a** VTP Feature App, **I want** to subscribe to configuration changes, **so that** I receive the latest settings as soon as they are available.
**Priority:** Must
**Acceptance Criteria:**
- [ ] `vtp-configuration-service` implements a publisher/subscriber pattern
- [ ] API: `getConfiguration()`, `setConfiguration()`, `subscribeConfiguration()`, `unsubscribe()`
- [ ] Subscribers are immediately notified if configuration is already set at subscription time
- [ ] During SSR, configuration state is serialized for CSR hydration
- [ ] Service is a singleton shared across all VTP Feature Apps on the page
**Source:** archaeology-supporting.md (Section 2: VTP Configuration Service)

---

## Feature Area 13: Content Author Configuration

### REQ-065: PLP Content Author Controls
**As a** content author, **I want** to configure PLP behavior via AEM Content Fragment Models, **so that** I can tailor the listing experience per market.
**Priority:** Must
**Acceptance Criteria:**
- [ ] Configurable fields include: app context (results/favorites), details page URL pattern, filter start page path, favorites page path, new search URL, isNewCarsPage flag, other cars page path, power unit (PS/HP), mileage unit (km/miles)
- [ ] Filter configuration: up to 7 filter categories (each with up to 25 groups), up to 12 quick filter slots, carline photo overrides, filter info layers, equipment filter categories
- [ ] Location filter: Google Maps auth params, default radius, radius options
- [ ] filterHandlerOnly mode, campaign configuration, and central VTP configuration reference
**Source:** archaeology-plp.md (Content Author Configuration)

### REQ-066: PDP Content Author Controls
**As a** content author, **I want** to configure PDP behavior via AEM Content Fragment Models, **so that** I can tailor the detail page per market.
**Priority:** Must
**Acceptance Criteria:**
- [ ] Configurable fields include: search link (back breadcrumb URL), 3D webstreaming toggle and URLs, next best actions selection and order (favorite/share/audiCode), warranty info layer mappings, initial and extended technical data selections, trade-in teaser content, currency symbol position, rate/price breakdown toggles
- [ ] Central VTP configuration is referenced for CTAs, finance, consumption, and other shared settings
**Source:** archaeology-pdp.md (Content Author Configuration)

### REQ-067: VTP Configuration Content Author Controls
**As a** content author, **I want** to configure shared VTP settings (CTAs, finance, sorting, consumption) for all VTP Feature Apps, **so that** settings are consistent across PLP and PDP.
**Priority:** Must
**Acceptance Criteria:**
- [ ] CTA configuration: type, label, URL, method, target, display option, reservation filter, payment option filter, dealer ID filtering, GLC product IDs
- [ ] Finance: enablement toggle, option type, disclaimer items, e-commerce hiding
- [ ] Sorting: sort options array, default option, conditional overrides
- [ ] Consumption/emission: NEDC/WLTP cycle selection, blacklist, links, efficiency image
- [ ] SCS: market path for API routing
- [ ] Other: currency/date patterns, mileage unit, mandatory area search, Google cookie consent, not-found redirect, vehicle identification type, warranty logo URL
**Source:** archaeology-supporting.md (Section 1: Content Author Configuration, VTPConfiguration type)

### REQ-068: Price Configuration Model
**As a** content author, **I want** to configure price display per business model and availability, **so that** pricing matches the market's sales model.
**Priority:** Must
**Acceptance Criteria:**
- [ ] Price configuration is defined per business model (`dealer_stock`, `nsc_stock`, `agency_model`, `agency_model_dealer`) and availability (`now`, `soon`, `date`)
- [ ] Each configuration contains nested price fragments with type (retail/regular/rate/custom), path, and label
- [ ] Price groups support bold styling, VAT reclaimable flag, suffix, footnote, and disclaimer text
**Source:** archaeology-supporting.md (Content Fragment Models: Price Configuration)

---

## Feature Area 14: SSR / Performance Requirements

### REQ-069: Server-Side Rendering Support
**As a** performance-conscious platform, **I want** all VTP Feature Apps to support SSR, **so that** pages load faster and are SEO-friendly.
**Priority:** Must
**Acceptance Criteria:**
- [ ] PLP and PDP render skeleton placeholders during SSR (not real vehicle data)
- [ ] SSR is detected via `typeof window === 'undefined'`
- [ ] All `window`/`document` accesses are wrapped in SSR-safe guards
- [ ] SSR state is serialized for CSR hydration (via `s2:serialized-state-manager`)
- [ ] SSR services (`s2:async-ssr-manager`, `s2:server-request`, `s2:serialized-state-manager`) are optional dependencies
- [ ] Fallback dimensions (e.g., window width = 400) are used when window is undefined
- [ ] Both classic and alpha design variants have SSR bundles (`app-ssr.js`, `app-alpha-ssr.js`)
**Source:** archaeology-plp.md (Section 12: SSR), archaeology-pdp.md (Edge Cases: SSR Safety), archaeology-supporting.md (Configuration Service: SSR State Serialization, FeatureAppInitialization)

### REQ-070: Skeleton Loading States
**As a** vehicle shopper, **I want** to see skeleton placeholders while content loads, **so that** I understand the page is loading.
**Priority:** Must
**Acceptance Criteria:**
- [ ] PLP shows skeleton placeholders for: toolbar, tiles, individual tile, and quick filters
- [ ] Skeletons display during SSR and during Redux loading state
- [ ] Real content replaces skeletons after data is fetched
**Source:** archaeology-plp.md (Section 12: SSR Support)

### REQ-071: Image Performance Optimization
**As a** performance-conscious platform, **I want** images to be optimized for performance, **so that** pages load quickly.
**Priority:** Must
**Acceptance Criteria:**
- [ ] Images use lazy loading with eager loading for adjacent slide positions
- [ ] WebP format is preferred when available via media service
- [ ] Responsive image sizes are served based on viewport (400/600/900/1000/1440px)
- [ ] Responsive `srcSet` attributes are used for resolution optimization
**Source:** archaeology-plp.md (Section 2: GalleryImage), archaeology-pdp.md (Section 1: Image loading)

### REQ-072: Lighthouse Performance Thresholds
**As a** platform, **I want** the PLP to meet minimum Lighthouse scores, **so that** I maintain acceptable performance and accessibility standards.
**Priority:** Should
**Acceptance Criteria:**
- [ ] PLP Lighthouse accessibility minimum score: 0.3 (error threshold)
- [ ] PLP Lighthouse performance minimum score: 0.75 (warning threshold)
**Source:** archaeology-plp.md (Infrastructure: Lighthouse Configuration)

---

## Feature Area 15: Accessibility Requirements

### REQ-073: Keyboard and Screen Reader Accessibility
**As a** user with disabilities, **I want** the VTP to be navigable via keyboard and screen readers, **so that** I can browse and interact with vehicles.
**Priority:** Must
**Acceptance Criteria:**
- [ ] All interactive elements have `aria-label` attributes
- [ ] Focus management: after "Load More", focus moves to the first new tile; on PDP scroll-to-top, focus moves to the first visible focusable element
- [ ] Focus layers have `primaryAriaLabel` and `secondaryAriaLabel` for gallery and warranty layers
- [ ] Tab navigation uses `role="tablist"` and `role="tab"` with keyboard `onKeyNavigate`
- [ ] External links use `rel="noopener"` with `target="_blank"`
- [ ] Semantic heading hierarchy is maintained (h2 for sections, h3 for subsections)
**Source:** archaeology-pdp.md (Edge Cases: Accessibility), archaeology-plp.md (Section 4: Load More)

---

## Feature Area 16: Tracking / Analytics

### REQ-074: PLP Tracking Events
**As a** business analyst, **I want** key PLP interactions to be tracked, **so that** I can analyze user behavior on the listing page.
**Priority:** Must
**Acceptance Criteria:**
- [ ] Tracked events include: `feature_app_ready` (with search name, results count, filter state, sort option), gallery image change, sort change, go to detail page, add/remove from favorites, load more, tab click, CTA clicks (contact/ecom/call), modal layer open/close, NC/UC switch, advanced filter open, quick filter open/close, share
- [ ] Component update data includes: `implementer: 2`, available filter categories, sorting option, search name (new/used), result count, active filters, viewType
- [ ] All tracking uses `audi-tracking-service` (OneSight V2)
**Source:** archaeology-plp.md (Tracking section)

### REQ-075: PDP Tracking Events
**As a** business analyst, **I want** key PDP interactions to be tracked, **so that** I can analyze user behavior on the detail page.
**Priority:** Must
**Acceptance Criteria:**
- [ ] Tracked events include: `feature_app_ready` (with product data), gallery open/close, image change, thumbnail toggle, favorite add/remove, share, Audi code, equipment impression/info click/tyre label click, CTA button clicks, warranty display, tech data drawer toggle, "I am interested" click, GLC checkout
- [ ] Product tracking data includes: `productId`, `productName`, `manufacturer: 'Audi'`, `primaryCategory` (carline group), `subCategory1` (carline), `productType` (new/used car)
- [ ] All tracking uses `audi-tracking-service` (OneSight V2)
**Source:** archaeology-pdp.md (Tracking section)

### REQ-076: Filter Tracking Events
**As a** business analyst, **I want** filter interactions to be tracked, **so that** I can understand how users apply filters.
**Priority:** Must
**Acceptance Criteria:**
- [ ] Tracked events include: filter overlay open/close, individual filter clicks, results count after filter, applied filter list
- [ ] Tracking data includes: `componentName: 'vtp-filter'`, search name (used/new), applied filters array, available categories
**Source:** archaeology-supporting.md (Shared: Tracking)

---

## Feature Area 17: Location Filter with Google Maps

### REQ-077: Location Search with Autocomplete
**As a** vehicle shopper, **I want** to search for vehicles near a specific location using Google Maps autocomplete, **so that** I can find vehicles near me.
**Priority:** Must
**Acceptance Criteria:**
- [ ] A location search input with Google Maps autocomplete is displayed
- [ ] Selecting a location filters results by the configured radius
- [ ] Radius options are content-configurable (default: 10, 20, 50, 100, 200)
- [ ] Default radius is configurable (default: 10)
- [ ] Location search state is persisted in localStorage with 30-day expiry
**Source:** archaeology-plp.md (Section 6: Location Filter)

### REQ-078: Browser Geolocation ("Locate Me")
**As a** vehicle shopper, **I want** to use my browser's geolocation to find vehicles near me, **so that** I don't have to type my location.
**Priority:** Should
**Acceptance Criteria:**
- [ ] A "Locate me" icon button triggers browser geolocation
- [ ] On success, the location filter is applied with the user's coordinates
- [ ] Geolocation errors are logged to console (no user-facing error message)
**Source:** archaeology-plp.md (Section 6: Location Filter)

### REQ-079: Google Maps GDPR Consent
**As a** EU user, **I want** Google Maps features to require my explicit consent, **so that** my privacy is protected per GDPR requirements.
**Priority:** Must
**Acceptance Criteria:**
- [ ] Google Maps requires two-click consent before activation
- [ ] A consent modal with a toggle switch is displayed
- [ ] Consent can be persistent or per-session
- [ ] `enableGoogleCookieConsent` controls whether the consent flow is active
**Source:** archaeology-plp.md (Section 6: Business Rules), archaeology-supporting.md (CookieConsentControl, CookieConsentRequest)

---

## Feature Area 18: Market-Specific Behaviors

### REQ-080: Germany (DE) Market Variations
**As a** German market user, **I want** the VTP to comply with German-specific regulations and conventions, **so that** the experience is legally compliant and locally relevant.
**Priority:** Must
**Acceptance Criteria:**
- [ ] EnVKV-compliant consumption display using `<ENVKV>` component when `countryCode === 'DE'`
- [ ] Fallback sorting explanation text in German if i18n key is empty
**Source:** archaeology-plp.md (Market Variations: DE), archaeology-pdp.md (Market Variations: Germany)

### REQ-081: France (FR) Market Variations
**As a** French market user, **I want** the VTP to display French-specific efficiency labels, **so that** the display meets French regulatory requirements.
**Priority:** Must
**Acceptance Criteria:**
- [ ] When `country === 'fr'`, the efficiency display uses `vehicle.eecLabel` instead of the computed SVG path
**Source:** archaeology-plp.md (Market Variations: FR)

### REQ-082: Spain (ES) Market Variations
**As a** Spanish market user, **I want** the VTP to respect Spanish-specific sorting behavior, **so that** the experience matches local expectations.
**Priority:** Must
**Acceptance Criteria:**
- [ ] ES market does NOT auto-switch to distance sorting when a location filter is set (exception to REQ-020)
- [ ] Spain used cars use a dedicated finance/price template
**Source:** archaeology-plp.md (Market Variations: ES), archaeology-supporting.md (Market Variations: Spain)

### REQ-083: Configurable Units and Formatting
**As a** content author, **I want** to configure power units, mileage units, currency, and date formats per market, **so that** the display matches local conventions.
**Priority:** Must
**Acceptance Criteria:**
- [ ] Power unit: PS or HP (configurable)
- [ ] Mileage unit: km or miles (configurable)
- [ ] Currency: symbol and formatting pattern configurable (6+ patterns supported)
- [ ] Date format: 10 date pattern options available
- [ ] SCS market path: per-market API path segment (e.g., `de/nc`, `es/uc`)
**Source:** archaeology-plp.md (Data Models: powerUnit, mileageUnit), archaeology-supporting.md (VTPConfiguration: PatternType, DatePatternType)

### REQ-084: Business Model Differentiation
**As a** VTP system, **I want** to adapt behavior based on the vehicle's business model, **so that** the correct conversion flow is presented.
**Priority:** Must
**Acceptance Criteria:**
- [ ] Supported business models: `dealer_stock`, `nsc_stock`, `agency_model`, `agency_model_dealer`
- [ ] NWS (Nationwide Selling) vehicles: detected via `isNationWideSellingVehicle()`; affects dealer comments, image display, CTA types
- [ ] BEV Agency vehicles: detected via `isBevAgencyVehicle()`; affects dealer comments, CTA types (gets `bevAgency` type), AOZ contact button type
- [ ] Investor-shared vehicles: chain dealers are fetched and displayed
**Source:** archaeology-pdp.md (Business Rules: Business Model Differentiation), archaeology-supporting.md (VTPConfiguration: businessModel)

---

## Feature Area 19: Design System

### REQ-085: Classic and Alpha Design System Support
**As a** platform, **I want** the VTP to support both the classic (legacy) and alpha (new) Audi design systems, **so that** pages can transition to the new design incrementally.
**Priority:** Must
**Acceptance Criteria:**
- [ ] Both CSR and SSR bundles are built for each design variant: `app.js`/`app-ssr.js` (classic), `app-alpha.js`/`app-alpha-ssr.js` (alpha)
- [ ] `dynamicStylingHelper('legacyValue', 'alphaValue')` selects the correct value based on active design system
- [ ] `DynamicComponent` renders different UIs for classic vs alpha
- [ ] Classic uses: `@oneaudi/unified-web-components` / `@oneaudi/unified-web-common`
- [ ] Alpha uses: `@oneaudi/unified-web-alpha-components` / `@oneaudi/unified-web-alpha-common`
- [ ] Webpack alpha aliases remap classic imports to alpha equivalents for alpha builds
**Source:** archaeology-plp.md (Infrastructure: Design Variants), archaeology-supporting.md (Top-Level Shared: Alpha aliases)

---

## Feature Area 20: Vehicle Order Status & Availability

### REQ-086: Vehicle Order Status Badge
**As a** vehicle shopper, **I want** to see delivery status badges on vehicles, **so that** I know if a vehicle is in transit or available at the dealer.
**Priority:** Should
**Acceptance Criteria:**
- [ ] Order status states 7–10 display an "in-delivery" badge
- [ ] Order status states 11–12 display an "at-dealer" badge
- [ ] Only numeric states in the valid range are shown
**Source:** archaeology-supporting.md (Shared: VehicleOrderStatus)

### REQ-087: Availability Badge
**As a** vehicle shopper, **I want** to see an availability status badge on vehicle tiles and the detail page, **so that** I know if the vehicle is available now, coming soon, or reserved.
**Priority:** Should
**Acceptance Criteria:**
- [ ] Availability is determined by: car type (NC/UC), business model, `availableFrom`, `availableFromCode`, `reservation` flag, and dealer city
- [ ] Badge renders appropriate status text
**Source:** archaeology-supporting.md (Shared: AvailableBadge)

---

## Feature Area 21: Internationalization (i18n)

### REQ-088: Full Internationalization Support
**As a** multi-market platform, **I want** all user-visible text to come from i18n translation keys, **so that** the VTP works in any supported language.
**Priority:** Must
**Acceptance Criteria:**
- [ ] All user-visible labels use i18n keys via `@oneaudi/i18n-service`
- [ ] Key patterns include: `stockcars.*`, `nemo.ui.sc.*`
- [ ] Locale detection uses `gfa:locale-service` for language and country
- [ ] Warranty, favorites, CTA, consumption, filter, and status labels are all i18n-managed
**Source:** archaeology-plp.md (Dependencies), archaeology-pdp.md (Market Variations: All Markets), archaeology-supporting.md (I18n Keys)

---

## Feature Area 22: Toolbar

### REQ-089: PLP Toolbar
**As a** vehicle shopper, **I want** a top bar with filter access, favorites link, and navigation, **so that** I can quickly access key PLP functions.
**Priority:** Must
**Acceptance Criteria:**
- [ ] Filter button with active filter count badge is shown in `results` context (mobile)
- [ ] Favorites link with count is shown (hidden when `enableMyAudiWishlist` is true)
- [ ] Back button is shown in `favorites` context; uses `window.history.back()` or redirects to `/`
- [ ] SSR guard: logs info instead of navigating when running server-side
**Source:** archaeology-plp.md (Section 8: Toolbar)

---

## Feature Area 23: Vehicle Identification

### REQ-090: Vehicle Identification Methods
**As a** VTP system, **I want** to support multiple vehicle identification methods, **so that** vehicles can be identified by the most appropriate identifier per market.
**Priority:** Must
**Acceptance Criteria:**
- [ ] Supported identification types: `commissionNumber`, `vin`, `croppedVin`, `numberplate`
- [ ] Vehicle ID is extracted from the URL via `getVehicleIdFromUrl()` supporting: `sc_detail` parameter, SEO URLs with `/id…/` pattern, and `?vehicleId=` query parameter
- [ ] SEO-friendly PDP URLs are generated from model year, description, and ID
**Source:** archaeology-supporting.md (VehicleIdentificationType, getVehicleIdFromUrl)

---

## Feature Area 24: Audi Code

### REQ-091: Audi Code Display
**As a** vehicle shopper, **I want** to view and copy a vehicle's Audi Code, **so that** I can reference it in communications with dealers.
**Priority:** Could
**Acceptance Criteria:**
- [ ] Audi Code is displayed via a Next Best Action button on the PDP
- [ ] Clicking shows the code in a popover with a copy-to-clipboard function
- [ ] An analytics event is tracked when the Audi Code is accessed
**Source:** archaeology-pdp.md (Section 4: NBA — AudiCodeButton)

---

## Feature Area 25: Share

### REQ-092: Share Vehicle (PDP)
**As a** vehicle shopper, **I want** to share a vehicle detail page URL, **so that** I can send a specific vehicle to someone.
**Priority:** Could
**Acceptance Criteria:**
- [ ] A Share action is available in the PDP Next Best Actions
- [ ] Clicking copies the vehicle URL (with filter hash params) to the clipboard
- [ ] A tracking event is fired on share
**Source:** archaeology-pdp.md (Section 4: NBA — ShareButton)

---

## Feature Area 26: Footnotes & Legal Disclaimers

### REQ-093: Footnote and Disclaimer System
**As a** legally compliant platform, **I want** footnotes and disclaimers to be displayed alongside prices, consumption, and other legal data, **so that** regulatory requirements are met.
**Priority:** Must
**Acceptance Criteria:**
- [ ] Footnote references are rendered inline using `audi-footnote-reference-service`
- [ ] Footnote text is managed by `audi-footnote-service`
- [ ] Disclaimers support configurable types: `Global`, `Product`, `Calculation`
- [ ] Disclaimer text style interpretation is toggleable (`scopes_interpretDisclaimersTextStyle`)
**Source:** archaeology-plp.md (Dependencies), archaeology-pdp.md (Dependencies), archaeology-supporting.md (VTPConfiguration: disclaimerType)

---

## Assumptions

- All vehicles are sourced from the SCS (Stock Car Service) API; no other vehicle data sources exist for the EU VTP
- Content authors have access to AEM Headless / Universal Editor and can manage Content Fragment Models
- The Feature Hub orchestrates all VTP Feature Apps on a given page; VTP FAs do not function independently outside the Feature Hub
- SSR rendering is handled by Renderman; SSR services are available in the SSR environment
- Google Maps API access is available for all markets requiring location filtering
- The myAudi authentication service is available when `enableMyAudiWishlist` is enabled
- The CRS (Customer Rate Service) API is available for markets where dynamic financing is enabled
- The Partner Business Card Feature App is deployed and available at the configured version for dealer info rendering
- Design system choice (classic vs alpha) is determined at build/deploy time, not at runtime by content authors

---

## Open Questions

1. **Content overrides system:** How does the `content-overrides/` directory modify per-market behavior? What specific overrides exist?
2. **Cypress E2E coverage:** What PLP and PDP scenarios are covered by the monorepo-level Cypress tests?
3. **Finance calculator details:** What is the full user flow for the dynamic finance calculator/layer? (Logic lives in `vtp-shared`, not fully traced.)
4. **OneGraph production usage:** Is the OneGraph integration (PLP demo component) planned for production vehicle data queries?
5. **Instavid 360 spin:** Is the 360 spin integration (hook exists) actively used in production?
6. **LaunchDarkly:** Are there plans to introduce LaunchDarkly feature flags, or will all toggling remain content-managed?
7. **Japan (JP) market:** The JP-specific price template is referenced, but is the EU VTP deployed in Japan?
8. **Compare feature:** Compare data models and APIs (`fetchCompare()`) exist in the shared package — is a compare feature in scope?
9. **Availability notification CTA:** What is the full flow for `availability-notification` CTA type?
10. **WhatsApp CTA:** What is the implementation for the `whatsApp` CTA type?
11. **Mandatory Area Search tracking:** What specific events are tracked for the mandatory area search flow?
12. **Sold out page content:** Since the soldout FA is headless, how is the visible "sold out" page content rendered? Is it an AEM page template?
13. **Multi-page composed experience:** Can PLP and PDP exist on the same AEM page, or are they always separate pages?
14. **`hideEcom` scope:** Under what conditions is e-commerce fully disabled?
15. **Employee vehicles:** The `employeeVehicle` field exists — are there special display or access rules for employee vehicles?

---

## Dependencies

- **SCS API (Stock Car Service):** Primary vehicle data source — required for all vehicle listing, filtering, detail, and favorites functionality
- **CRS API (Customer Rate Service):** Required for dynamic finance calculations (FSAG products, rate responses)
- **Google Maps API:** Required for location autocomplete and geolocation features
- **AEM Headless (Content Service):** Required for content fragment delivery and all content author configuration
- **Feature Hub:** Required orchestration layer for all VTP Feature App lifecycle, dependency injection, and service sharing
- **Renderman (SSR):** Required for server-side rendered page delivery
- **myAudi Auth Service:** Required for cloud-synced wishlist and login flows
- **Omnigraph (GraphQL):** Required for myAudi wishlist queries
- **OneGraph (GraphQL):** Used for carline name resolution in model filters
- **Partner Business Card FA:** Required for dealer information rendering on PDP
- **AVP 3D Webstream Service:** Required for 3D vehicle visualization (optional, new cars only)
- **Falcon Content API:** Required for Trade-In teaser editorial content
- **Audi Design System Libraries:** Required UI components (`unified-web-components` classic, `unified-web-alpha-components` alpha)
- **OneSight Tracking Service:** Required for analytics event dispatch

Save this file to [requirements/structured-requirements-eu-vtp.md](requirements/structured-requirements-eu-vtp.md).

---

The document covers **93 requirements** across **26 feature areas**, all directly sourced from the three archaeology reports. Could you enable file editing tools so I can save it directly, or would you like to copy the code block above into the file?