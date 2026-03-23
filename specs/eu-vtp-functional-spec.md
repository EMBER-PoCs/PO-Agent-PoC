# Functional Specification: EU VTP (Vehicle Transaction Pages)

**Version:** 1.0  
**Date:** 2026-03-22  
**Status:** Draft  

---

## 1. Overview

The EU VTP (Vehicle Transaction Pages) is a suite of React-based microfrontend Feature Apps that deliver the vehicle browsing, filtering, and purchasing experience across European Audi markets. The system is composed of independently deployed Feature Apps orchestrated by the Feature Hub within Adobe Experience Manager (AEM) pages, rendered server-side by Renderman.

This specification documents the current behavior of the EU VTP as reverse-engineered from the `audi-eu-vtp` monorepo codebase. It covers the Vehicle Listing Page (VLP/PLP), Vehicle Details Page (VDP/PDP), and all supporting Feature Apps and services.

### System Composition

| Page | Feature Apps Present |
|------|---------------------|
| Vehicle Listing Page | `fa-vtp-configuration` + `fa-vtp-plp` |
| Vehicle Detail Page | `fa-vtp-configuration` + `fa-vtp-pdp` + `fa-vtp-dealer-info` |
| Sold-Out Page | `fa-vtp-soldout` |

All VTP Feature Apps share configuration via a headless configuration Feature App (`fa-vtp-configuration`) that reads AEM Content Fragments and broadcasts settings through a pub/sub configuration service (`vtp-configuration-service`). A shared library (`vtp-shared`) provides common components, hooks, types, and utilities consumed by all Feature Apps.

---

## 2. Goals and Non-Goals

### Goals

- Provide a complete vehicle browsing and detail experience for Audi's European markets
- Support content-author-driven configuration of features, CTAs, filters, pricing, and display options per market
- Comply with market-specific legal regulations (DE EnVKV, FR efficiency labels)
- Support both the classic ("unified") and alpha Audi design systems concurrently
- Deliver performant pages via SSR with CSR hydration
- Enable cloud-synced favorites via myAudi authentication
- Provide configurable e-commerce checkout flows (GLC lean checkout)
- Support comprehensive analytics tracking of user interactions

### Non-Goals

- New feature development or future roadmap items — this spec documents existing behavior only
- US/Canada VTP — those are separate Feature App compositions with different architecture
- AEM page template design or AEM author workflows beyond Content Fragment configuration
- Backend SCS/CRS API specifications — those are external dependencies consumed as-is
- Feature Hub or Renderman infrastructure — those are platform services outside VTP scope

---

## 3. Requirements Traceability

| Spec Section | Requirement IDs | Status |
|-------------|----------------|--------|
| 4.1 Vehicle Search & Browsing | REQ-001, REQ-002, REQ-003, REQ-004, REQ-005, REQ-006, REQ-007, REQ-008 | Documented |
| 4.2 Vehicle Filtering | REQ-009, REQ-010, REQ-011, REQ-012, REQ-013, REQ-014, REQ-015, REQ-016, REQ-017, REQ-018 | Documented |
| 4.3 Vehicle Sorting | REQ-019, REQ-020, REQ-021 | Documented |
| 4.4 Vehicle Detail View | REQ-022, REQ-023, REQ-024, REQ-025, REQ-026, REQ-027, REQ-028, REQ-029, REQ-030, REQ-031, REQ-032, REQ-033, REQ-034 | Documented |
| 4.5 Vehicle Image Gallery | REQ-035, REQ-036, REQ-037, REQ-038 | Documented |
| 4.6 Favorites / Wishlist | REQ-039, REQ-040, REQ-041 | Documented |
| 4.7 Finance & Pricing | REQ-042, REQ-043, REQ-044, REQ-045, REQ-046, REQ-047 | Documented |
| 4.8 Consumption & Emissions | REQ-048, REQ-049, REQ-050 | Documented |
| 4.9 Dealer Information | REQ-051, REQ-052, REQ-053 | Documented |
| 4.10 Contact / CTA System | REQ-054, REQ-055, REQ-056, REQ-057, REQ-058, REQ-059, REQ-060 | Documented |
| 4.11 Vehicle Sold Out | REQ-061, REQ-062 | Documented |
| 4.12 Configuration System | REQ-063, REQ-064 | Documented |
| 4.13 Content Author Configuration | REQ-065, REQ-066, REQ-067, REQ-068 | Documented |
| 4.14 Location Filter & Maps | REQ-077, REQ-078, REQ-079 | Documented |
| 4.15 Market-Specific Behaviors | REQ-080, REQ-081, REQ-082, REQ-083, REQ-084 | Documented |
| 4.16 Design System | REQ-085 | Documented |
| 4.17 Vehicle Order Status & Availability | REQ-086, REQ-087 | Documented |
| 4.18 Internationalization | REQ-088 | Documented |
| 4.19 Toolbar | REQ-089 | Documented |
| 4.20 Vehicle Identification | REQ-090 | Documented |
| 4.21 Audi Code | REQ-091 | Documented |
| 4.22 Share | REQ-006, REQ-092 | Documented |
| 4.23 Footnotes & Legal Disclaimers | REQ-093 | Documented |
| 5.1 SSR / Performance | REQ-069, REQ-070, REQ-071, REQ-072 | Documented |
| 5.2 Accessibility | REQ-073 | Documented |
| 5.3 Tracking / Analytics | REQ-074, REQ-075, REQ-076 | Documented |

---

## 4. Functional Requirements

### 4.1 Vehicle Search & Browsing

#### 4.1.1 Vehicle Tile Grid Display (REQ-001)

##### Description

The PLP (`@oneaudi/fa-vtp-plp`) renders available vehicles as a responsive grid of tiles. Each tile is a self-contained card displaying a summary of the vehicle with actionable elements.

##### User Flow

1. User navigates to the Vehicle Listing Page
2. The PLP Feature App initializes alongside the configuration FA on the same AEM page
3. Vehicle data is fetched from the SCS API via `fetchAnythingScs()`
4. Vehicles render as a responsive grid of tiles

##### Business Rules

- Grid layout is responsive: 1 column on XS, 2 at MD, 2–3 at LG, 3–4 at 2XL
- Page size is fixed at 12 vehicles per fetch

##### Data Model

Each tile displays:

| Element | Source |
|---------|--------|
| Image gallery | `vehicle.basic.images` (render + dealer photos) |
| Carline name | `vehicle.basic.symbolicCarlineDescription` (h3 heading) |
| Model code description | `vehicle.basic.modelCodeDescription` |
| Pricing / finance info | `vehicle.basic.financing`, price configuration |
| Key features | `vehicle.basic` selected fields |
| Dealer info | `vehicle.basic.dealer` |
| Availability badge | Computed from car type, business model, `availableFrom`, `reservation` |
| CTA buttons | VTPConfiguration `cta[]` |
| Favorite icon | Local Redux store or myAudi wishlist |
| Consumption/emission data | `vehicle.basic.consumption`, `vehicle.basic.envkv2024` |

##### Acceptance Criteria (from REQ-001)

- Vehicles display in a responsive CSS grid: 1 column on XS, 2 columns at MD, 2–3 at LG, 3–4 at 2XL
- Each tile shows: vehicle image gallery, carline name (h3 heading), model code description, pricing/finance info, key features, dealer info, availability badge, CTA buttons, favorites icon, and consumption/emission data
- Clicking a tile image or headline navigates to the vehicle detail page (PDP)
- Grid renders within the PLP Feature App (`@oneaudi/fa-vtp-plp`)

#### 4.1.2 Progressive Load More Pagination (REQ-002)

##### Description

Results load progressively via a "Load More" button rather than traditional pagination or infinite scroll.

##### User Flow

1. Initial page load fetches 12 vehicles
2. User scrolls to the bottom of results
3. User clicks "Load More" button
4. Next 12 vehicles are fetched and appended to the grid
5. Focus moves to the first newly loaded tile (accessibility)
6. When all vehicles are loaded, the "Load More" button is hidden

##### Business Rules

- Batch size is 12 vehicles (hardcoded)
- "Load More" is hidden when `totalCount ≤ fetchedCount` or `totalCount` is null/undefined
- A progress indicator is shown during loading

##### Acceptance Criteria (from REQ-002)

- Initial page loads 12 vehicles
- "Load More" button fetches the next 12 vehicles and appends them to the grid
- "Load More" button is hidden when all vehicles have been loaded
- A progress indicator is shown during loading
- After new vehicles appear, focus moves to the first newly loaded tile
- "Load More" button is hidden when total count is null or undefined

#### 4.1.3 Return-to-Position Scroll (REQ-003)

##### Description

When a user navigates from a vehicle tile to the PDP and then returns, the PLP scrolls to the previously viewed vehicle.

##### Business Rules

- The selected vehicle ID is stored when a user clicks a tile
- On return navigation, `scrollToPreviouslySelectedVehicle()` restores scroll position
- State is stored in localStorage

##### Acceptance Criteria (from REQ-003)

- When a user clicks a vehicle tile, the selected vehicle ID is stored
- When the user navigates back to the PLP, the page scrolls to the previously selected vehicle tile

#### 4.1.4 New Cars / Used Cars Toggle (REQ-004)

##### Description

Users can switch between new car and used car listings via toggle buttons in the quick filters area.

##### Business Rules

- `isNewCarsPage` boolean determines current context
- Market path differentiates new (`de/nc`) vs used (`de/uc`) at the SCS API level
- Clicking the toggle navigates to the opposite car type page (configured via `plpOtherCarsPagePathname`)

##### Acceptance Criteria (from REQ-004)

- NC/UC toggle buttons are displayed in the quick filters area
- Clicking the toggle navigates to the opposite car type page
- `isNewCarsPage` boolean determines the current context
- Market path differentiates new vs used at the API level

#### 4.1.5 Zero Results State (REQ-005)

##### Description

When filters produce no matching vehicles, a helpful empty state is displayed.

##### Business Rules

- Zero results from filters: "No results found" message with "Reset filters" button
- Zero favorites: headline, copy text, and "New search" link

##### Acceptance Criteria (from REQ-005)

- When filters produce zero results, a "No results found" message and "Reset filters" button are displayed
- Clicking "Reset filters" clears all filters and re-fetches unfiltered results
- When the favorites list is empty, a zero-favorites state is shown with headline, copy, and a "New search" link

#### 4.1.6 Share Results URL (REQ-006)

##### Description

Users can copy a shareable URL that preserves current filter state.

##### Acceptance Criteria (from REQ-006)

- A share button is displayed in the results bar
- Clicking share copies the current URL (including filter hash) to the clipboard
- A toast notification confirms successful copy
- Existing URL hashes are stripped before appending the filter hash

#### 4.1.7 SEO Filter Resolution (REQ-007)

##### Description

Filter state can be resolved from SEO-friendly URLs, enabling indexable and shareable filtered vehicle pages.

##### Business Rules

- `createFilterSeoResolver` resolves SEO-friendly filter slugs before app initialization
- Filter data is fetched from the SCS API on initial load and on URL hash changes
- URL filter hash is cleaned up after initial filter fetch

##### Acceptance Criteria (from REQ-007)

- SEO filters are resolved from the URL before the app renders initial results
- Filter data is fetched from the SCS API on initial load and on URL hash changes
- URL filter hash is cleaned up after initial filter fetch

#### 4.1.8 Filter Handler Only Mode (REQ-008)

##### Description

A PLP instance can be configured to handle filter logic without rendering any visible UI, useful for embedding filter logic on non-listing pages.

##### Acceptance Criteria (from REQ-008)

- When `filterHandlerOnly` is true, the PLP renders no visible UI
- Filter state is still managed and synchronized

---

### 4.2 Vehicle Filtering

#### 4.2.1 Quick Filters Panel (REQ-009)

##### Description

A quick-access filter panel provides the most commonly used filters. On desktop, it renders as a sidebar; on mobile, it opens as a fullscreen overlay triggered by a filter button in the toolbar.

##### Business Rules

- Up to 12 quick filter slots, configured via AEM Content Fragments
- Quick filters are only visible in the `results` context (not `favorites`)
- An "Advanced" button opens the full filter overlay

##### Data Model — Quick Filter Types

Filter type is determined by a suffix on the filter key:

| Suffix | Type | Rendering |
|--------|------|-----------|
| `.checkbox` | Checkbox | Standard checkbox list |
| `.color-checkbox` | Color checkbox | Color tile selector |
| `.car-type-icon-checkbox` | Car type icon checkbox | Body type icon selector |
| `.model-checkbox` | Model checkbox | Carline group checkboxes |
| `.range` | Range | Slider with min/max |
| `.location` | Location | Location search input |

##### Acceptance Criteria (from REQ-009)

- Desktop: quick filters render as a sidebar panel
- Mobile: quick filters render as a fullscreen overlay
- Up to 12 quick filters are configurable by content authors
- Supported types: checkbox, color-checkbox, car-type-icon-checkbox, model-checkbox, range, location
- Active filter selections are shown as filter chips
- An "Advanced" button opens the full filter overlay
- Quick filters are only visible in `results` context

#### 4.2.2 Advanced Filters Overlay (REQ-010)

##### Description

A full-screen overlay with categorized filters for detailed filtering. Categories are organized as navigation tabs with accordion sections within each category.

##### Business Rules

- Up to 7 filter categories, each with up to 25 filter groups
- Each filter supports configurable layout width (50%, 100%, or responsive 50%/100%)
- Selecting/deselecting filters updates results dynamically

##### Acceptance Criteria (from REQ-010)

- Advanced filters open in a full overlay with category navigation tabs
- Up to 7 filter categories, each with up to 25 filter groups
- Filter overlay includes: navigation bar, accordion sections, footer with result count and apply/reset actions
- Supported filter types: checkbox, range, equipment block, model, location, color, car-type-icon
- Selecting/deselecting filters updates results dynamically

#### 4.2.3 Filter Chip Display (REQ-011)

##### Description

Active filter selections are shown as dismissible chips for quick visibility and removal.

##### Acceptance Criteria (from REQ-011)

- Active filters are represented as dismissible chips
- Removing a chip deselects that filter and re-fetches results

#### 4.2.4 Model / Carline Filter (REQ-012)

##### Description

A checkbox-based filter for Audi model/carline selection.

##### Data Model — Supported Carlines

`a1`, `a2`, `a3`, `a4`, `a5`, `a6`, `a7`, `a8`, `q2`, `q3`, `q4`, `q5`, `q6`, `q7`, `q8`, `q8etron`, `tt`, `r8`, `q6etron`, `etrongt`

##### Business Rules

- A "Show more options" / "Hide options" toggle controls visibility
- Responsive initial count: 3 on XS/S/L/XL, 6 on M
- Carline names resolved via OneGraph GraphQL

##### Acceptance Criteria (from REQ-012)

- Model checkboxes display carline groups with vehicle counts
- A toggle controls visibility of additional models
- Responsive breakpoint logic determines initial visible count

#### 4.2.5 Equipment Filter (REQ-013)

##### Description

Equipment features can be filtered via grouped checkbox blocks, with an intro category and up to 4 sub-categories.

##### Acceptance Criteria (from REQ-013)

- An equipment filter intro category and up to 4 sub-categories are configurable
- Equipment filters render as grouped checkbox blocks

#### 4.2.6 Range Filters (REQ-014)

##### Description

Numeric range filters render as sliders with min/max values for constraining results.

##### Data Model — Range Filter Dimensions

Price (retail, rate, all-in), power (kW, PS), mileage, displacement, acceleration, CO2 emission, electric range, model year, luggage capacity, initial registration year, previous owners, doors, finance amount, production year.

##### Acceptance Criteria (from REQ-014)

- Range filters render as slider controls with min/max values
- All listed range dimensions are supported

#### 4.2.7 Filter Info Layers (REQ-015)

##### Description

Info buttons next to filters open explanatory content layers.

##### Acceptance Criteria (from REQ-015)

- Content authors can configure info layer links per filter group
- Clicking an info button opens a layer with content from the configured URL

#### 4.2.8 Filter State Persistence (REQ-016)

##### Description

Filter selections persist across page interactions via URL hash and localStorage.

##### Business Rules

- Active filters are stored in the URL hash and localStorage
- Returning to the PLP restores filters
- Filter state is included in shared URLs

##### Acceptance Criteria (from REQ-016)

- Active filters are persisted in the URL hash and localStorage
- Returning to the PLP restores previously active filters
- Filter state is included in shared URLs

#### 4.2.9 Mandatory Area Search (REQ-017)

##### Description

When enabled, users must enter a location before vehicle results are displayed.

##### Business Rules

- Controlled by `enableMandatoryAreaSearch` content toggle
- A modal triggers when the user attempts to close the filter overlay without entering a location

##### Acceptance Criteria (from REQ-017)

- When `enableMandatoryAreaSearch` is enabled, users must enter a location before seeing results
- A mandatory area search modal is triggered when the user attempts to close the filter overlay without entering a location

#### 4.2.10 Dealer List Filter (REQ-018)

##### Description

Users can filter vehicles by specific dealers within the location filter area.

##### Business Rules

- "Select all dealers" checkbox can be deactivated via `deactivateSelectAllDealers`

##### Acceptance Criteria (from REQ-018)

- A dealer list is displayed within the location filter area
- Individual dealers can be selected/deselected
- A "Select all dealers" checkbox is available
- Dealer selection affects vehicle results

---

### 4.3 Vehicle Sorting

#### 4.3.1 Sort Dropdown (REQ-019)

##### Description

A sort dropdown in the results bar allows users to reorder vehicle results by various criteria.

##### Business Rules

- Sort options are content-configurable per market
- Default sort option is set via `sortParams_defaultOption`
- Sort is disabled when there are 0 or 1 results
- Sort state is persisted in the session
- A sort info popover explains the current sorting logic

##### Acceptance Criteria (from REQ-019)

- A sort dropdown is displayed in the results bar alongside the vehicle count
- Sort options are content-configurable per market
- Default sort option is content-configurable
- Sort is disabled when there are 0 or 1 results
- Sort state is persisted in the session
- A sort info popover explains the current sorting logic

#### 4.3.2 Distance Sorting Auto-Application (REQ-020)

##### Description

When a location filter is set, sorting automatically switches to distance ascending.

##### Business Rules

- Auto-switches to `byDistance:asc` when location is set
- Distance sorting is only available when a location is set
- **Market exception:** ES (Spain) does NOT auto-switch to distance sorting (see REQ-082)

##### Acceptance Criteria (from REQ-020)

- When a location filter is set, sorting auto-switches to distance ascending
- Distance sorting is only available when a location is set
- ES market does NOT auto-switch to distance sorting when location is set

#### 4.3.3 Configurable Sort Override (REQ-021)

##### Description

Content authors can configure a filter that, when triggered, overrides the default sort key.

##### Business Rules

- `overWriteSorting_filterToTriggerSorting` specifies the trigger filter
- `overWriteSorting_overrideDefaultSortingKey` specifies the target sort key
- The override does not re-trigger if the same filter value is already active

##### Acceptance Criteria (from REQ-021)

- Content authors can configure a filter that auto-triggers a specific sort key
- The override does not re-trigger if the same filter value is already active

---

### 4.4 Vehicle Detail View (PDP)

#### 4.4.1 Vehicle Detail Page Layout (REQ-022)

##### Description

The PDP (`@oneaudi/fa-vtp-pdp`) renders a comprehensive detail view for a single vehicle. Vehicle data is fetched from the SCS API using the vehicle ID extracted from the URL.

##### User Flow

1. User clicks a vehicle tile on the PLP (or navigates directly via URL)
2. Vehicle ID is extracted from the URL (supports `sc_detail` param, `/id…/` SEO paths, `?vehicleId=` query param)
3. `initializeFeatureApp()` fetches complete vehicle data (`basic` + `detail`) from SCS API
4. Vehicle data is provided via `StockContextProvider` to all PDP components
5. If vehicle returns 404, redirect to `notFound` URL (301 status) or return 404

##### PDP Layout Sections

| Section | Component | Description |
|---------|-----------|-------------|
| Image gallery stage | `Gallery` / `GalleryRethink` | Full image gallery with swipe, fullscreen, 3D view |
| Vehicle header | `CarInfoRethink` | Carline name, model code, key specs |
| Next Best Actions | `NextBestActionsRethink` | Favorite, Share, Audi Code buttons |
| Tab sections | `DetailSectionTabs` | Equipment, Technical Data, Warranties, Consumption, Dealer Comment, Product Deviation |
| Campaigns | `CampaignsSection` | Active campaign cards |
| Accessories (AOZ) | `AozSectionRethink` | Filterable accessory grid |
| Trade-in teaser | `TradeInTeaser` | Embedded Basic Teaser FA |
| Conversion bar | `ConversionBar` | Sticky bottom bar with price, CTAs |
| Back navigation | `Breadcrumb` | "Back to search page" link |

##### Acceptance Criteria (from REQ-022)

- PDP displays all listed sections
- PDP renders within the `@oneaudi/fa-vtp-pdp` Feature App
- PDP does not render if vehicle basic or detail data is missing

#### 4.4.2 Vehicle Info Header (REQ-023)

##### Description

The vehicle name, model description, and key specifications are displayed prominently below the image gallery.

##### Acceptance Criteria (from REQ-023)

- Vehicle symbolic carline description and model code are displayed
- Key specification items are shown below the name

#### 4.4.3 Tab Navigation for Detail Sections (REQ-024)

##### Description

Up to 6 detail tabs are conditionally rendered based on data availability.

##### Business Rules

- Tabs are: Equipment, Technical Data, Warranties, Consumption & Emissions, Dealer Comment, Product Deviation
- Empty tabs (no data) are silently omitted
- Keyboard navigation is supported via `onKeyNavigate`

##### Acceptance Criteria (from REQ-024)

- Up to 6 tabs conditionally rendered based on data availability
- Tabs use semantic `role="tablist"` and `role="tab"` attributes
- Keyboard navigation is supported
- Empty tabs are silently omitted

#### 4.4.4 Equipment Tab (REQ-025)

##### Description

Shows optional and standard equipment with image cards, detail modals, and equipment video playback support.

##### Business Rules

- Optional equipment: paginated image cards with detail modal layers
- Standard equipment: grouped table
- Equipment split into items with and without images
- Dealer-provided equipment shown separately
- Tyre labels and product sheets shown where available
- Tab shown only when `vehicle.detail.features` has items

##### Acceptance Criteria (from REQ-025)

- Optional equipment shown as paginated image cards with detail modal layers
- Standard equipment shown in a grouped table
- Additional dealer-provided equipment shown separately
- Tyre label URLs and product sheets displayed where available
- Equipment video playback supported in the detail modal
- Tab shown only when `vehicle.detail.features` has items

#### 4.4.5 Technical Data Tab (REQ-026)

##### Description

Technical specifications displayed as key-value lists with initial and expandable extended views. Content authors configure which of 65+ available fields appear and in what order.

##### Business Rules

- Initial view shows fields from `scsTechdataInitial`
- Extended view shown via Show more/less toggle, fields from `scsTechdataExtended`
- Pollution badge, damages/defects, battery certificate, product safety info link, and vehicle dimensions shown when available
- Tab shown when configured keys have matching vehicle data

##### Acceptance Criteria (from REQ-026)

- Technical data displayed as a key-value list with initial and expandable extended views
- Content authors configure which tech data fields appear
- Additional data items shown when available
- Tab shown when configured keys exist and matching vehicle data exists

#### 4.4.6 Warranties Tab (REQ-027)

##### Description

Warranty information displayed as horizontally scrollable cards with template substitution for dynamic values.

##### Data Model — Warranty Types

`NAP`, `warranty`, `plus`, `5-years`, `asg-extended`, `asg`, `gwplus5-years-extended`, `twelve-months`, `clp`, `cpo`

##### Business Rules

- Warranty text supports template substitution: `${key:date}`, `${key:number}`, `${key}`
- Info button opens URL in a focus layer (configurable per warranty type)
- Labels differ for new vs used vehicles (different i18n keys)
- Tab listens for `pdp:open-warranties-tab` custom DOM event
- Tab shown when `vehicle.basic.warrantyInfo.warranties` has entries

##### Acceptance Criteria (from REQ-027)

- Warranties displayed as horizontally scrollable cards
- Template substitution for date and number values
- Info button with configurable URL per warranty type
- Warranty labels differ for new vs used vehicles
- Tab listens for programmatic navigation event
- Tab shown when warranty data exists

#### 4.4.7 Dealer Comment Tab (REQ-028)

##### Description

Dealer-specific comments or notes about a vehicle, displayed as HTML content.

##### Business Rules

- Content truncated at 150 characters with expand/collapse toggle
- NWS and BEV Agency vehicles display i18n-sourced text instead of dealer remarks
- HTML content sanitized via DOMPurify
- Tab shown when remarks exist (trimmed, non-empty) or vehicle is NWS/BEV Agency

##### Acceptance Criteria (from REQ-028)

- Dealer comments display as HTML content truncated at 150 characters with expand/collapse
- NWS and BEV Agency vehicles display i18n text instead of dealer remarks
- Tab shown when remarks exist or vehicle is NWS/BEV Agency

#### 4.4.8 Product Deviation Tab (REQ-029)

##### Description

Differences from standard specification, with downloadable side letter PDF documents.

##### Business Rules

- Text truncated at 150 characters with expand/collapse
- PDF documents shown with document-pdf icons
- Tab shown when `vehicle.basic.productDeviations` exists or `vehicle.detail.documents` has entries

##### Acceptance Criteria (from REQ-029)

- Product deviation text shown with 150-character truncation and expand/collapse
- Downloadable PDF documents shown with document-pdf icons
- Tab shown when product deviations or documents exist

#### 4.4.9 Campaigns Display (REQ-030)

##### Description

Active campaigns associated with a vehicle are displayed as paginated cards.

##### Business Rules

- Cards show banner image, title, info text, and "More information" link
- "More information" opens campaign microsite in a layer
- Only render when `vehicle.basic.campaigns` exists
- Campaign activity determined by date range via `isCampaignActive()`

##### Acceptance Criteria (from REQ-030)

- Campaigns displayed as paginated cards with banner, title, info text, and link
- Link opens campaign microsite in a layer
- Only render when campaigns exist
- Campaign date range determines activity

#### 4.4.10 Accessories / Original Equipment (REQ-031)

##### Description

Audi Original Accessories (AOZ) displayed as a filterable, paginated product grid with selection persistence.

##### Business Rules

- Default: 6 items per page, 8 on large screens
- Filter by category: Highlights (default), All, subcategories
- Product cards: image, name, price with currency, checkbox for selection
- Detail modal for more information
- Selections persisted in `sessionStorage` keyed by vehicle type + vehicle ID
- Contact button sends selected AOZ products with vehicle data
- Liquid products show base price with unit conversion
- Only renders when `vehicle.detail.aoz` exists and vehicle has ID and type

##### Acceptance Criteria (from REQ-031)

- Accessories displayed as filterable, paginated product grid
- Filter allows category selection with Highlights as default
- Product cards show image, name, price, and selection checkbox
- Detail modal for additional information
- Selections persisted in sessionStorage
- Contact button sends selected products with vehicle data
- Section only renders when AOZ data exists

#### 4.4.11 Trade-In Teaser (REQ-032)

##### Description

An embedded Basic Teaser Feature App for trade-in offers, rendered via the `<Spawn>` component from `@oneaudi/falcon-tools`.

##### Business Rules

- Used cars: shown only if `tradeInUc === true`
- New cars: shown only if `tradeInNc === true`
- URL placeholders (e.g., `{{sc_vehicle_id}}`) replaced with vehicle data
- Content model must be configured by content authors

##### Acceptance Criteria (from REQ-032)

- Trade-in teaser rendered as an embedded Feature App
- Conditionally shown based on car type and trade-in flags
- URL placeholders replaced with vehicle data

#### 4.4.12 Back Navigation Breadcrumb (REQ-033)

##### Description

A breadcrumb link back to the search page, with the URL configurable via the `searchLink` content field.

##### Acceptance Criteria (from REQ-033)

- A breadcrumb displays a link back to the search page
- The back URL is content-configurable

#### 4.4.13 Next Best Actions (REQ-034)

##### Description

Quick action buttons displayed below the image gallery stage.

##### Business Rules

- Available actions: Favorite (toggle), Share (copy URL to clipboard), Audi Code (show code in popover with copy)
- Content authors configure which NBAs to show and their order via the `nbas` field

##### Acceptance Criteria (from REQ-034)

- A row of action buttons displayed below the stage
- Available actions: Favorite, Share, Audi Code
- Content authors configure which NBAs appear and their order

---

### 4.5 Vehicle Image Gallery

#### 4.5.1 PLP Tile Image Gallery (REQ-035)

##### Description

Each tile shows a swipeable image slider with pagination arrows and a counter.

##### Business Rules

- Swipe and arrow navigation supported
- Images use responsive `srcSet` and WebP format via media service
- Lazy loading for off-screen images, eager loading for adjacent slides
- Clicking a gallery image navigates to the detail page

##### Acceptance Criteria (from REQ-035)

- Each tile shows an image slider with Previous/Next arrows and a counter
- Swipe and arrow navigation supported
- Images use responsive `srcSet` and WebP format
- Lazy loading with eager loading for adjacent slides
- Clicking a gallery image navigates to the detail page

#### 4.5.2 PLP Image Selection Logic (REQ-036)

##### Description

Image selection varies by vehicle type and business model.

##### Business Rules

| Condition | Image Behavior |
|-----------|---------------|
| New cars (type `N`) | Show only render images |
| Used cars (type `U`) | Show dealer photos alongside render images |
| `hideRenderImages` flag set | Render images suppressed |
| `nationWideSelling` vehicles | Never show dealer images |
| No images available | Fallback images used |

##### Acceptance Criteria (from REQ-036)

- New cars show only render images
- Used cars show dealer photos alongside render images
- `hideRenderImages` suppresses render images
- NWS vehicles never show dealer images
- Fallback images used when no other images available

#### 4.5.3 PDP Full Image Gallery (REQ-037)

##### Description

A large, swipeable image gallery on the detail page with fullscreen view capability.

##### Business Rules

- Swipe (touch) and prev/next arrow navigation
- Fullscreen expand button opens fullscreen gallery layer
- Responsive sizes: 400/600/900/1000/1440px with WebP format
- Lazy loading for non-first images
- Dynamic alt text generated via `useDynamicAltText` for accessibility
- Fallback images shown when no other images available

##### Acceptance Criteria (from REQ-037)

- Gallery supports swipe and prev/next navigation
- Fullscreen expand button opens a fullscreen gallery layer
- Responsive image sizes served with WebP format
- Lazy loading for non-first images
- Dynamic alt text generated for accessibility
- Fallback images shown when none available

#### 4.5.4 3D Webstream / AVP Integration (REQ-038)

##### Description

Real-time 3D rendering of new vehicles via the Audi Visualization Platform (AVP) web component.

##### Business Rules

- 3D view available only when ALL conditions met:
  - Vehicle is new car (type `N`)
  - `use3dWebstreaming` is true
  - `avpRessourceUrl` is set
  - `configId` is set
- 3D viewer loads as `<avp-3dws-deck>` web component in a full overlay
- Error events from AVP stop the loading spinner
- Z-index management: webstream z-index=1 (below conversion bar), fullscreen z-index=101

##### Acceptance Criteria (from REQ-038)

- "3D view" button displayed (TextButton on desktop, IconButton on mobile)
- 3D view only available when all four conditions are met
- 3D viewer loads as a web component in a full overlay
- Error events stop the loading spinner

---

### 4.6 Favorites / Wishlist

#### 4.6.1 Local Favorites (REQ-039)

##### Description

Users can mark vehicles as favorites from PLP tiles and view them on a dedicated favorites page. Local favorites are stored in the Redux store (`FAVORITE_VEHICLES`).

##### User Flow

1. User clicks the heart icon on a vehicle tile
2. Vehicle is added to the local favorites store
3. Favorites count badge updates in the toolbar
4. User navigates to the favorites page to view saved vehicles
5. Removing a favorite from the favorites view shows a confirmation overlay
6. Removing a favorite from the results view is immediate (no confirmation)

##### Acceptance Criteria (from REQ-039)

- Favorite icon button (heart) on each vehicle tile
- Clicking adds/removes the vehicle from favorites
- Favorites link with count badge in the toolbar
- Favorites page shows saved vehicles using the same tile format
- Confirmation overlay for removal in favorites view
- Immediate removal in results view
- Empty state with headline, description, and "New search" link

#### 4.6.2 myAudi Cloud Wishlist (REQ-040)

##### Description

When `enableMyAudiWishlist` is true, favorites sync to the user's myAudi account via the Omnigraph GraphQL API.

##### User Flow

1. Unauthenticated user clicks favorite → login layer prompts authentication
2. Vehicle ID stored in localStorage before login redirect
3. After login, vehicle is auto-added to the cloud wishlist
4. Toast notifications confirm add/remove actions
5. Favorites sync across devices via myAudi

##### Business Rules

- Vehicle type (NEW/USED) passed to the wishlist API
- When enabled, the local favorites link in the toolbar is hidden
- Authentication via `@oneaudi/audi-auth-service` and `vw:authService` Feature Service

##### Acceptance Criteria (from REQ-040)

- Cloud sync via myAudi Wishlist API when enabled
- Login layer for unauthenticated users
- Auto-add after login redirect
- Vehicle type passed to API
- Toast notifications for add/remove
- Local favorites link hidden when cloud wishlist enabled

#### 4.6.3 PDP Favorite Button (REQ-041)

##### Description

Favorite action available from the PDP via Next Best Actions.

##### Acceptance Criteria (from REQ-041)

- Favorite action available via Next Best Actions on the PDP
- Behavior matches PLP: local storage or myAudi wishlist depending on configuration
- Post-login auto-add supported

---

### 4.7 Finance & Pricing

#### 4.7.1 Retail Price Display (REQ-042)

##### Description

Retail price displayed on PLP tiles and in the PDP conversion bar with market-specific currency formatting.

##### Business Rules

- Currency formatting via `currencyPattern` and `audi-number-formatter-service`
- Currency symbol position configurable (`shiftCurrencySymbolLeftToRight`)
- Price footnotes attached where configured (`priceFootnote`)

##### Acceptance Criteria (from REQ-042)

- Retail price displayed on PLP tiles and PDP conversion bar
- Market-specific currency formatting
- Configurable currency symbol position
- Price footnotes attached where configured

#### 4.7.2 Finance Rate Display (REQ-043)

##### Description

Monthly finance rate shown when financing is enabled and available for the vehicle.

##### Business Rules

- Shown when `scopes_financeEnabled` is true AND vehicle has financing data
- "Change rate" link allows recalculation (hidden if `scopes_hideRateChangeCTA` is true)
- Hidden for e-commerce vehicles when `scopes_hideFinanceForEcom` is true

##### Acceptance Criteria (from REQ-043)

- Finance rate per month shown when finance is enabled and vehicle has data
- "Change rate" link for recalculation with configurable visibility
- Finance hidden for e-commerce vehicles when configured

#### 4.7.3 Price Breakdown (REQ-044)

##### Description

Detailed price breakdown accessible from the conversion bar.

##### Business Rules

- Configuration defined per business model and availability status
- Rate can be included in summary breakdown (`includeRateInSummaryBreakdown`)
- Selected product options can be included (`showSelectedProductOptionsInBreakdown`)

##### Acceptance Criteria (from REQ-044)

- Price breakdown CTA available in the conversion bar
- Configuration defined per business model and availability
- Rate and selected product options optionally included

#### 4.7.4 Finance Option Modes (REQ-045)

##### Description

Three finance display modes support varying levels of detail per market requirements.

##### Data Model — Finance Option Types

| Mode | Description |
|------|-------------|
| `MINIMAL` | Minimal finance information |
| `FULL` | Complete finance details |
| `STATIC_WITH_DISCLAIMER` | Static display with disclaimer text |

##### Business Rules

- Finance layer disclaimer items configurable
- Finance table can be hidden (`financeLayer_hideFinanceTable`)
- Calculation disclaimer can be hidden (`scopes_hideCalculationDisclaimer`)

##### Acceptance Criteria (from REQ-045)

- Three finance option modes supported
- Finance layer disclaimer items configurable
- Finance table and calculation disclaimer can be hidden

#### 4.7.5 Dynamic Financing (REQ-046)

##### Description

Users can recalculate financing terms with different parameters via the CRS API.

##### Data Flow

```
SCS API (vehicle.basic.financing) → FinanceProvider context
                                     ↓
CRS API (FSAG products + default calculation) → useDynamicFinancing hook
                                     ↓
                              Session storage persistence (per vehicle type NC/UC)
                                     ↓
              Finance components (rate display, change rate, breakdown)
```

##### Business Rules

- FSAG products and default calculations fetched from CRS API
- Finance data persisted in session storage per vehicle type (NC/UC)
- Finance results cached to minimize API calls via `getCachedValue()`/`updateCachedValue()`

##### Acceptance Criteria (from REQ-046)

- Dynamic financing fetches from CRS API
- Data persisted in session storage per vehicle type
- Results cached to minimize API calls

#### 4.7.6 Market-Specific Price Templates (REQ-047)

##### Description

Price and finance display adapts to market-specific formats.

##### Business Rules

| Market | Template |
|--------|----------|
| Spain (ES) used cars | `PriceInformationSpain` |
| Japan (JP) used cars | `PriceInformationJapan` |
| All other markets | Default price/leasing/taxation template |

Currency, date, and number formatting follow market-specific patterns.

##### Acceptance Criteria (from REQ-047)

- Spain and Japan use dedicated price templates
- All other markets use the default template
- Formatting follows market-specific patterns

---

### 4.8 Consumption & Emissions

#### 4.8.1 Standard Consumption & Emission Display (REQ-048)

##### Description

Consumption and emission data for non-DE markets, with WLTP prioritized over NEDC.

##### Business Rules

- WLTP prioritized over NEDC; both can be shown if configured
- Content authors control test cycles via `consumptionEmission_consumptionEmissionRepresentation`
- Blacklist can hide certain emission types (`consumptionEmission_scsConsumptionEmissionBlacklist`)
- PLP tiles: inline summary with "Details" link to open a layer
- Detail layer: full values, efficiency class image, footnote references
- Efficiency class image controlled by `useEfficiencyImage` and `urls_eecImageUrl`
- PHEV vehicles receive special consumption label handling (multi-fuel)

##### Acceptance Criteria (from REQ-048)

- WLTP prioritized over NEDC
- Content authors control test cycle display
- Blacklist for hiding emission types
- PLP inline summary with details layer
- Detail layer with full values and efficiency image
- PHEV multi-fuel handling

#### 4.8.2 German EnVKV Compliance (REQ-049)

##### Description

German market uses the legally required EnVKV energy label instead of the standard consumption display.

##### Business Rules

- When `countryCode === 'DE'`, the `<ENVKV>` component renders instead of standard `<CEE>`
- ENVKV 2024 data uses SVG label URLs from `vehicle.basic.envkv2024`
- Applies on both PLP tiles and PDP consumption tab
- Tab shown when vehicle has `envkv2024` data with SVG label URLs

##### Acceptance Criteria (from REQ-049)

- EnVKV component renders for DE market
- ENVKV 2024 data with SVG label URLs
- Applies on both PLP and PDP
- Tab shown when envkv2024 data exists

#### 4.8.3 French Market Efficiency Label (REQ-050)

##### Description

French market uses `vehicle.eecLabel` instead of the computed SVG path for efficiency display.

##### Acceptance Criteria (from REQ-050)

- When `country === 'fr'`, efficiency display uses `vehicle.eecLabel`

---

### 4.9 Dealer Information

#### 4.9.1 Dealer Info on PLP Tiles (REQ-051)

##### Description

Dealer name displayed on each vehicle tile via the `DealerInfoRethink` shared component.

##### Acceptance Criteria (from REQ-051)

- Dealer name displayed on each vehicle tile

#### 4.9.2 Dealer Info Feature App (REQ-052)

##### Description

The `fa-vtp-dealer-info` Feature App embeds the `fa-partner-business-card` Feature App to render dealer information on the PDP. Configuration is passed including partner ID, BEV agency flags, investor-shared status, and display toggles.

##### Business Rules

- Display variant: "Show dealer name and address only"
- Toggleable options: official name, phone, email, Google Map link
- Returns null if no vehicle data available

##### Acceptance Criteria (from REQ-052)

- Dealer info displayed by embedding Partner Business Card FA
- Configuration includes partner ID, BEV agency flags, NWS flag, imprint, locale
- Display options for name, phone, email, map link
- Returns null when no vehicle data

#### 4.9.3 Chain Dealer Display (REQ-053)

##### Description

Chain dealers are fetched and displayed for investor-shared vehicles.

##### Business Rules

- Fetched via `useChainDealers()` hook
- List passed to Partner Business Card
- Empty array used on fetch error (graceful degradation)

##### Acceptance Criteria (from REQ-053)

- Chain dealers fetched for investor-shared vehicles
- Chain dealer list passed to Partner Business Card
- Empty array on error

---

### 4.10 Contact / CTA System

#### 4.10.1 Configurable CTA Buttons on PLP Tiles (REQ-054)

##### Description

Up to 2 CTA buttons per tile (primary + secondary) plus a favorite icon button, with CTAs loaded from configuration.

##### Business Rules

- CTAs loaded from `vtp-configuration-service`
- Fallback "Go to Details" button when no CTAs configured
- Supported types: `details`, `contact`, `nws`, `bevAgency`, `ecom`, `phone`, `central-customer-hotline`
- Phone CTA can show number directly (`phoneWithNumber` flag)
- Phone CTA can be forced as primary (`scopes_forcePhoneAsPrimary`)

##### Acceptance Criteria (from REQ-054)

- Up to 2 CTA buttons per tile plus favorite icon
- CTAs from configuration service
- Fallback "Go to Details" when no CTAs configured
- All listed CTA types supported
- Phone number display and primary forcing options

#### 4.10.2 Conversion Bar / Sticky CTA (REQ-055)

##### Description

Sticky bottom bar on the PDP with vehicle name, price, and action buttons, portaled to `<main>` via `ReactDOM.createPortal`.

##### Business Rules

- Displays: vehicle name (carline + model code), retail price, finance rate (if applicable), CTA buttons
- ≤2 buttons: all visible; >2: primary visible, secondary in "I am interested" popover
- Mobile: primary full-width, secondary as icon buttons
- Bar moves up when footer scrolls into view

##### Acceptance Criteria (from REQ-055)

- Sticky bottom bar portaled to `<main>`
- Displays vehicle name, retail price, finance rate, CTA buttons
- Button overflow handled with popover
- Mobile-optimized layout
- Footer-aware positioning

#### 4.10.3 CTA Button Types — Full Catalog (REQ-056)

##### Description

A comprehensive catalog of CTA types available for content author configuration.

##### Data Model — CTA Types

| Type | Description |
|------|-------------|
| `availability-notification` | Notify when available (toggles with `reserve` based on `vehicle.reservation`) |
| `contact` | Standard dealer contact |
| `bevAgency` | BEV agency model contact |
| `custom` | Custom URL action |
| `central-customer-hotline` | Central hotline contact |
| `ecom` | E-commerce flow |
| `finance-checkout` | Finance checkout |
| `cash-checkout` | Cash purchase checkout |
| `leasing` | Leasing flow |
| `nws` | Nationwide selling contact |
| `reserve` | Reserve vehicle |
| `liteReservation` | Lightweight reservation |
| `phone` | Phone CTA |
| `details` | Navigate to details |
| `dealer` | Dealer page link |
| `whatsApp` | WhatsApp contact |
| `financeOptions` | Finance options display |
| `financeInfo` | Finance information |
| `glc-reservation` | GLC reservation checkout |
| `glc-cash-checkout` | GLC cash checkout |
| `glc-financing-checkout` | GLC financing checkout |
| `aoz` | Accessories contact |

##### Business Rules

- Each CTA has configurable: label, URL, method (GET/POST), target (same-window/new-window/open-in-layer), display option (tiles/carinfo/both/none)
- Reserve/availability-notification toggle based on `vehicle.reservation`
- Finance checkout buttons only shown when finance enabled and vehicle has financing data
- CTAs can be filtered by `buyableOnline` and `paymentOptions`

##### Acceptance Criteria (from REQ-056)

- All 22 CTA types supported
- Each CTA configurable with label, URL, method, target, display option
- Conditional visibility based on vehicle state and configuration

#### 4.10.4 CTA Dealer ID Filtering (REQ-057)

##### Description

CTA buttons can be shown or hidden based on the vehicle's dealer.

##### Business Rules

- Include list: button only shown for listed dealers
- Exclude list: button hidden for listed dealers
- Controlled via `options_filterByDealerId_filterType` and `options_filterByDealerId_filterList`

##### Acceptance Criteria (from REQ-057)

- CTA buttons support include/exclude dealer ID lists
- Include: button only shown for those dealers
- Exclude: button hidden for those dealers

#### 4.10.5 CTA POST Data Submission (REQ-058)

##### Description

POST CTAs submit hidden forms with comprehensive vehicle and context data.

##### Business Rules

- POST data includes: vehicle data, audicode, financing data, AOZ selected products, dealer filter IDs
- Data profiles: `generic` (full), `finance-checkout-only`, `no-finance-data`
- URL placeholders replaced (e.g., `{{sc_vehicle_id}}`, `{{sc_audicode}}`, `{{sc_dealer_id}}`)
- Target options: layer (iframe), new window, same window
- URL template replacement supports 20+ vehicle data placeholders via `formatUrl()`

##### Acceptance Criteria (from REQ-058)

- POST CTAs submit hidden forms with vehicle data
- Data profile controls what data is included
- URL placeholders replaced with actual vehicle data
- Multiple target options supported

#### 4.10.6 GLC Lean Checkout (REQ-059)

##### Description

E-commerce checkout initiated from the PDP via the `eCommerceService.startCheckout()` method.

##### Business Rules

- `glc-reservation`, `glc-cash-checkout`, `glc-financing-checkout` CTAs trigger checkout
- Payload includes vehicle data, financing information, and selected accessories
- GLC financing product IDs configurable per CTA
- Checkout dispatched via the `@volkswagen-onehub/e-commerce-service` Feature Service

##### Acceptance Criteria (from REQ-059)

- GLC CTAs start checkout via e-commerce service
- Payload includes vehicle data, financing, and accessories
- Financing product IDs configurable per CTA

#### 4.10.7 Lite Reservation (REQ-060)

##### Description

Lightweight reservation requiring valid dealer email and PDF base URL.

##### Acceptance Criteria (from REQ-060)

- Lite reservation requires valid dealer email and PDF base URL
- CTA only shown when vehicle supports reservation

---

### 4.11 Vehicle Sold Out Experience

#### 4.11.1 Sold Out Page (REQ-061)

##### Description

The `fa-vtp-soldout` Feature App sets the HTTP status to 404 when a vehicle is no longer available. The FA itself renders no visible UI — the 404 page content is rendered by AEM.

##### Business Rules

- Uses `page-info-service` to set `response.status: 404` and `response.message: 'No Content Found'`
- Existing SEO description is preserved
- i18n messages define: headline, copy text, "Start a new search" button, "Back to search page" link

##### Acceptance Criteria (from REQ-061)

- HTTP status set to 404
- Existing SEO description preserved
- FA renders no visible UI
- i18n messages available for headline, copy, and navigation

#### 4.11.2 Vehicle Not Found Redirect (REQ-062)

##### Description

When the SCS API returns 404 for a vehicle, the system redirects or returns an error status.

##### Business Rules

- 404 from SCS API → redirect to `notFound` URL (301) or return 404 status
- Non-404 API errors → Feature App silently hides (no vehicle data = no render)

##### Acceptance Criteria (from REQ-062)

- 404 redirects to configured not-found URL or returns 404
- Non-404 errors cause silent Feature App hiding

---

### 4.12 Configuration System

#### 4.12.1 Headless Configuration Injection (REQ-063)

##### Description

The `fa-vtp-configuration` Feature App reads AEM Content Fragments, maps them from flat field keys to a nested `VTPConfiguration` object, and broadcasts them to all VTP Feature Apps.

##### Data Flow

```
AEM Content Authors (Content Fragments)
           ↓ audi-content-service getContent()
fa-vtp-configuration (mapContent: flat keys → nested object)
           ↓ configService.setConfiguration()
vtp-configuration-service (pub/sub store)
           ↓ subscribeConfiguration()
PLP / PDP / dealer-info
```

##### Business Rules

- Field key mapping: e.g., `scopes_financeEnabled` → `{ scopes: { financeEnabled: true } }`
- The FA renders no visible UI (returns `null`)

##### Acceptance Criteria (from REQ-063)

- Configuration FA reads AEM Content Fragments
- Content mapped from flat keys to nested structure
- Configuration broadcast via service
- FA renders no visible UI

#### 4.12.2 Configuration Service Pub/Sub (REQ-064)

##### Description

The `vtp-configuration-service` implements a publisher/subscriber pattern for reactive configuration distribution.

##### API

| Method | Purpose |
|--------|---------|
| `getConfiguration()` | Read current configuration |
| `setConfiguration()` | Set/update configuration (triggers subscribers) |
| `subscribeConfiguration()` | Register callback for configuration changes |
| `unsubscribe()` | Remove subscription |

##### Business Rules

- Subscribers immediately notified if configuration already set at subscription time
- During SSR, state serialized for CSR hydration
- Singleton shared across all VTP Feature Apps on the page

##### Acceptance Criteria (from REQ-064)

- Pub/sub pattern implemented
- All API methods available
- Immediate notification on subscribe if config exists
- SSR serialization for hydration
- Singleton across Feature Apps

---

### 4.13 Content Author Configuration

#### 4.13.1 PLP Content Author Controls (REQ-065)

##### Description

Content Fragment Model fields that control PLP behavior per market.

##### Configurable Fields

| Category | Fields |
|----------|--------|
| **Core** | App context (results/favorites), details page URL pattern, filter start page path, favorites page path, new search URL, isNewCarsPage, other cars page path |
| **Units** | Power unit (PS/HP), mileage unit (km/miles) |
| **Filters** | Up to 7 filter categories (25 groups each), up to 12 quick filter slots, carline photo overrides, filter info layers, equipment filter categories |
| **Location** | Google Maps auth params, default radius, radius options |
| **Modes** | filterHandlerOnly toggle |
| **Campaigns** | Campaign configuration |
| **Reference** | Central VTP configuration reference |

##### Acceptance Criteria (from REQ-065)

- All listed fields configurable via AEM Content Fragment Models

#### 4.13.2 PDP Content Author Controls (REQ-066)

##### Description

Content Fragment Model fields that control PDP behavior per market.

##### Configurable Fields

| Category | Fields |
|----------|--------|
| **Navigation** | Search link (back breadcrumb URL) |
| **3D View** | 3D webstreaming toggle, AVP resource URL |
| **NBAs** | Selection and order (favorite/share/audiCode) |
| **Warranties** | Info layer mappings per warranty type |
| **Technical Data** | Initial fields, extended fields (65+ available) |
| **Trade-In** | Teaser content configuration |
| **Pricing** | Currency symbol position, rate/price breakdown toggles |
| **Reference** | Central VTP configuration reference |

##### Acceptance Criteria (from REQ-066)

- All listed fields configurable via AEM Content Fragment Models

#### 4.13.3 VTP Configuration Content Author Controls (REQ-067)

##### Description

Shared VTP configuration fields consumed by all Feature Apps.

##### Configurable Fields

| Category | Fields |
|----------|--------|
| **CTAs** | Type, label, URL, method, target, display option, reservation filter, payment option filter, dealer ID filtering, GLC product IDs |
| **Finance** | Enablement toggle, option type, disclaimer items, e-commerce hiding |
| **Sorting** | Options array, default option, conditional overrides |
| **Consumption/Emission** | NEDC/WLTP cycle selection, blacklist, info links, efficiency image |
| **SCS** | Market path for API routing |
| **Formatting** | Currency/date patterns, mileage unit |
| **Features** | Mandatory area search, Google cookie consent, vehicle identification type |
| **URLs** | Not-found redirect, warranty logo URL |
| **Scopes** | 15+ toggleable scope flags |

##### Acceptance Criteria (from REQ-067)

- All listed fields configurable for consistent behavior across PLP and PDP

#### 4.13.4 Price Configuration Model (REQ-068)

##### Description

Price display configured per business model and availability status.

##### Data Model

```
mainPriceConfiguration
├── business model: dealer_stock | nsc_stock | agency_model | agency_model_dealer
│   └── availability: now | soon | date
│       └── items[]
│           ├── type: retail | regular | rate | custom
│           ├── path: data path to price value
│           └── label: display label
└── price groups
    ├── bold styling flag
    ├── VAT reclaimable flag
    ├── suffix text
    ├── footnote reference
    └── disclaimer text
```

##### Acceptance Criteria (from REQ-068)

- Price configuration per business model and availability
- Nested price fragments with type, path, and label
- Price groups with styling, VAT, suffix, footnote, and disclaimer

---

### 4.14 Location Filter & Google Maps

#### 4.14.1 Location Search with Autocomplete (REQ-077)

##### Description

Google Maps-powered location search with configurable radius for proximity filtering.

##### Business Rules

- Google Maps autocomplete on location input
- Radius options content-configurable (default: 10, 20, 50, 100, 200)
- Default radius configurable (default: 10)
- Location state persisted in localStorage with 30-day expiry

##### Acceptance Criteria (from REQ-077)

- Location search input with Google Maps autocomplete
- Configurable radius options and default
- Location state persisted in localStorage with 30-day expiry

#### 4.14.2 Browser Geolocation (REQ-078)

##### Description

"Locate me" button triggers browser geolocation for automatic location detection.

##### Acceptance Criteria (from REQ-078)

- "Locate me" icon button triggers browser geolocation
- On success, location filter applied with user's coordinates
- Geolocation errors logged to console

#### 4.14.3 Google Maps GDPR Consent (REQ-079)

##### Description

Google Maps features require explicit GDPR consent via a two-click flow.

##### Business Rules

- Consent modal with toggle switch
- Consent can be persistent or per-session
- Controlled by `enableGoogleCookieConsent`

##### Acceptance Criteria (from REQ-079)

- Two-click consent before Google Maps activation
- Consent modal with toggle switch
- Persistent or per-session consent
- Configurable enablement

---

### 4.15 Market-Specific Behaviors

#### 4.15.1 Germany (DE) Market Variations (REQ-080)

##### Description

German-specific regulatory compliance and conventions.

##### Business Rules

- EnVKV-compliant consumption display using `<ENVKV>` component when `countryCode === 'DE'` (see §4.8.2)
- ENVKV 2024 SVG label URLs from `vehicle.basic.envkv2024`
- Fallback sorting explanation text in German if i18n key is empty

##### Acceptance Criteria (from REQ-080)

- EnVKV component for consumption display
- German fallback text for sorting explanation

#### 4.15.2 France (FR) Market Variations (REQ-081)

##### Description

French-specific efficiency label display.

##### Business Rules

- When `country === 'fr'`, efficiency display uses `vehicle.eecLabel` instead of computed SVG path (see §4.8.3)

##### Acceptance Criteria (from REQ-081)

- French efficiency label uses `vehicle.eecLabel`

#### 4.15.3 Spain (ES) Market Variations (REQ-082)

##### Description

Spanish-specific sorting behavior and pricing.

##### Business Rules

- ES market does NOT auto-switch to distance sorting when location filter is set (exception to REQ-020)
- Spain used cars use a dedicated finance/price template (`PriceInformationSpain`)

##### Acceptance Criteria (from REQ-082)

- No auto-switch to distance sorting
- Dedicated price template for used cars

#### 4.15.4 Configurable Units and Formatting (REQ-083)

##### Description

Content-configurable units, currency, and date formatting per market.

##### Data Model — Pattern Options

| Setting | Options |
|---------|---------|
| Power unit | PS, HP |
| Mileage unit | km, miles |
| Currency pattern | 6+ patterns (e.g., `#.###,## €`, `€#,###.##`, etc.) |
| Date pattern | 10 options |
| SCS market path | Per-market segment (e.g., `de/nc`, `es/uc`) |

##### Acceptance Criteria (from REQ-083)

- All listed units and formatting options configurable per market

#### 4.15.5 Business Model Differentiation (REQ-084)

##### Description

Vehicle behavior adapts based on business model.

##### Data Model — Business Models

| Model | Detection | Effects |
|-------|-----------|---------|
| `dealer_stock` | Default | Standard dealer flow |
| `nsc_stock` | NWS check via `isNationWideSellingVehicle()` | No dealer images, NWS-specific CTA, NWS dealer comment |
| `agency_model` | BEV Agency check via `isBevAgencyVehicle()` | BEV Agency CTA, BEV Agency dealer comment, AOZ contact type |
| `agency_model_dealer` | Investor-shared flag | Chain dealers fetched and displayed |

##### Acceptance Criteria (from REQ-084)

- All four business models supported
- NWS, BEV Agency, and investor-shared detection and differentiation

---

### 4.16 Design System

#### 4.16.1 Classic and Alpha Design System Support (REQ-085)

##### Description

The VTP supports two concurrent Audi design systems, producing four bundles per visual Feature App.

##### Build Artifacts

| Bundle | Design | Rendering |
|--------|--------|-----------|
| `fh/app.js` | Classic | CSR |
| `fh/app-ssr.js` | Classic | SSR |
| `fh/app-alpha.js` | Alpha | CSR |
| `fh/app-alpha-ssr.js` | Alpha | SSR |

##### Implementation Patterns

| Pattern | Purpose |
|---------|---------|
| Webpack aliases | Alpha builds remap `@oneaudi/unified-web-components` → `@oneaudi/unified-web-alpha-components` |
| `DynamicComponent` | Renders different implementations based on active design system |
| `dynamicStylingHelper(legacyValue, alphaValue)` | Returns appropriate value for active design system |
| `DynamicThemeProviderWrapper` | Wraps app in appropriate theme provider |

##### Design System Libraries

| System | Components Package | Common Package |
|--------|-------------------|----------------|
| Classic | `@oneaudi/unified-web-components` ^1.45.0 | `@oneaudi/unified-web-common` ^1.13.0 |
| Alpha | `@oneaudi/unified-web-alpha-components` 1.22.0 | `@oneaudi/unified-web-alpha-common` 1.11.0 |

##### Acceptance Criteria (from REQ-085)

- Four bundles built per visual FA
- `dynamicStylingHelper` and `DynamicComponent` select correct values
- Both design system libraries supported
- Webpack aliases for alpha builds

---

### 4.17 Vehicle Order Status & Availability

#### 4.17.1 Vehicle Order Status Badge (REQ-086)

##### Description

Delivery status badges based on order status codes.

##### Business Rules

| Status Range | Badge |
|-------------|-------|
| 7–10 | "In delivery" |
| 11–12 | "At dealer" |
| Other | No badge |

Only numeric states in valid ranges are shown.

##### Acceptance Criteria (from REQ-086)

- Order status 7–10 displays "in-delivery" badge
- Order status 11–12 displays "at-dealer" badge
- Only valid numeric states shown

#### 4.17.2 Availability Badge (REQ-087)

##### Description

Availability status badge computed from multiple vehicle properties.

##### Business Rules

Availability determined by: car type (NC/UC), business model, `availableFrom`, `availableFromCode`, `reservation` flag, and dealer city.

##### Acceptance Criteria (from REQ-087)

- Availability computed from listed properties
- Badge renders appropriate status text

---

### 4.18 Internationalization (i18n)

#### 4.18.1 Full Internationalization Support (REQ-088)

##### Description

All user-visible text sourced from i18n translation keys.

##### Business Rules

- i18n via `@oneaudi/i18n-service` (^2.2.0) + `@oneaudi/i18n-context` (^5.1.0)
- Key patterns: `stockcars.*`, `nemo.ui.sc.*`
- Locale detection via `gfa:locale-service`
- 40+ key patterns covering all user-facing text
- Number/currency formatting via `@oneaudi/number-formatter-service`

##### Acceptance Criteria (from REQ-088)

- All user-visible labels use i18n keys
- Locale detection via locale service
- All feature areas fully i18n-managed

---

### 4.19 Toolbar

#### 4.19.1 PLP Toolbar (REQ-089)

##### Description

Top bar with filter access, favorites link, and navigation controls.

##### Business Rules

- Filter button with active filter count badge shown in `results` context (mobile)
- Favorites link with count (hidden when `enableMyAudiWishlist` is true)
- Back button in `favorites` context; uses `window.history.back()` or redirects to `/`
- SSR guard: logs info instead of navigating when running server-side

##### Acceptance Criteria (from REQ-089)

- Filter button with count badge on mobile
- Favorites link with count (hidden with cloud wishlist)
- Back button in favorites context
- SSR-safe navigation

---

### 4.20 Vehicle Identification

#### 4.20.1 Vehicle Identification Methods (REQ-090)

##### Description

Multiple vehicle identification methods supported for market-specific requirements.

##### Data Model — Identification Types

| Type | Description |
|------|-------------|
| `commissionNumber` | Commission number |
| `vin` | Full VIN |
| `croppedVin` | Shortened VIN |
| `numberplate` | License plate number |

##### Business Rules

- Vehicle ID extracted from URL via `getVehicleIdFromUrl()` supporting:
  - `sc_detail` parameter
  - SEO URLs with `/id…/` pattern
  - `?vehicleId=` query parameter
- SEO-friendly PDP URLs generated from model year + description + ID

##### Acceptance Criteria (from REQ-090)

- All four identification types supported
- URL extraction from three URL patterns
- SEO-friendly URL generation

---

### 4.21 Audi Code

#### 4.21.1 Audi Code Display (REQ-091)

##### Description

Audi Code shown via a Next Best Action button on the PDP with copy-to-clipboard functionality.

##### Acceptance Criteria (from REQ-091)

- Audi Code displayed via NBA button
- Popover with copy-to-clipboard
- Analytics event tracked on access

---

### 4.22 Share

#### 4.22.1 Share Vehicle from PDP (REQ-092)

##### Description

Share action in PDP Next Best Actions copies the vehicle URL with filter hash parameters to the clipboard.

##### Acceptance Criteria (from REQ-092)

- Share action in PDP NBAs
- Copies vehicle URL with filter hash to clipboard
- Tracking event fired on share

---

### 4.23 Footnotes & Legal Disclaimers

#### 4.23.1 Footnote and Disclaimer System (REQ-093)

##### Description

Inline footnote references and disclaimers rendered alongside prices, consumption, and other legal data.

##### Business Rules

- Footnote references via `audi-footnote-reference-service`
- Footnote text via `audi-footnote-service`
- Disclaimer types: `Global`, `Product`, `Calculation`
- Text style interpretation toggleable via `scopes_interpretDisclaimersTextStyle`

##### Acceptance Criteria (from REQ-093)

- Footnote references rendered inline
- Footnote text managed by service
- Three disclaimer types supported
- Text style interpretation toggleable

---

## 5. Non-Functional Requirements

### 5.1 SSR / Performance

#### 5.1.1 Server-Side Rendering Support (REQ-069)

##### Description

All VTP Feature Apps support SSR via Renderman. The rendering pipeline delivers pre-rendered HTML to the browser, which is then hydrated by the CSR bundles.

##### SSR Rendering Pipeline

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
         │             │             │
         └─────────────┼─────────────┘
                       ↓
              Pre-rendered HTML → Browser → CSR hydration
```

##### SSR Strategy by Package

| Package | SSR Approach |
|---------|-------------|
| PLP | Renders skeleton placeholders only (not real vehicle data). Vehicle data fetched client-side. |
| PDP | Fetches real vehicle data. Three-path init: SSR (fetch + serialize), CSR-after-SSR (deserialize), CSR-only (fetch). |
| Configuration | Serializes VTPConfiguration state for hydration. CSR-after-SSR deserializes without re-fetching. |
| Configuration Service | Serializes/deserializes config state across SSR boundary. |
| Dealer-info | SSR support via optional SSR services. |
| Soldout | Sets 404 page info during SSR via `page-info-service`. |

##### SSR Feature Services

| Service | ID | Role |
|---------|----|------|
| `s2:async-ssr-manager` | `@feature-hub/async-ssr-manager` | Schedules async rerenders during SSR |
| `s2:server-request` | `@feature-hub/server-request` | Provides HTTP request context |
| `s2:serialized-state-manager` | `@feature-hub/serialized-state-manager` | Serializes state for CSR hydration |

##### SSR Safety Patterns

- All `window`/`document` accesses guarded with `typeof window !== 'undefined'`
- Fallback dimensions for SSR (e.g., window width defaults to 400)
- Interactive elements gated by `isClient` from `useClientServerUtils()`
- Redux loading state triggers skeleton display during hydration

##### Acceptance Criteria (from REQ-069)

- PLP and PDP support SSR with appropriate strategies
- SSR-safe guards on all browser API accesses
- State serialization for hydration
- SSR services are optional dependencies
- Both classic and alpha SSR bundles produced

#### 5.1.2 Skeleton Loading States (REQ-070)

##### Description

Skeleton placeholders shown during SSR and initial data loading.

##### Skeleton Components

- Toolbar skeleton
- Tile grid skeleton
- Individual tile skeleton
- Quick filters skeleton

##### Acceptance Criteria (from REQ-070)

- PLP shows skeletons for toolbar, tiles, individual tile, and quick filters
- Skeletons display during SSR and Redux loading state
- Real content replaces skeletons after data fetch

#### 5.1.3 Image Performance Optimization (REQ-071)

##### Description

Image optimization patterns for fast page loads.

##### Optimization Techniques

| Technique | Implementation |
|-----------|---------------|
| Lazy loading | Off-screen images use lazy loading |
| Eager loading | Adjacent slide positions use eager loading |
| WebP format | Preferred when available via media service |
| Responsive sizes | 400/600/900/1000/1440px based on viewport |
| srcSet | Responsive `srcSet` attributes for resolution optimization |

##### Acceptance Criteria (from REQ-071)

- Lazy loading with eager for adjacent slides
- WebP format preferred
- Responsive sizes served
- srcSet attributes used

#### 5.1.4 Lighthouse Performance Thresholds (REQ-072)

##### Description

Minimum Lighthouse CI scores for quality gating.

##### Thresholds

| Metric | Threshold | Severity |
|--------|-----------|----------|
| Accessibility | ≥ 0.3 (30%) | Error (blocks CI) |
| Performance | ≥ 0.75 (75%) | Warning only |

Note: The accessibility threshold of 0.3 is low relative to WCAG 2.0 AA compliance requirements.

##### Acceptance Criteria (from REQ-072)

- PLP Lighthouse accessibility minimum: 0.3 (error threshold)
- PLP Lighthouse performance minimum: 0.75 (warning threshold)

---

### 5.2 Accessibility (REQ-073)

#### Description

Keyboard navigation, screen reader support, and semantic HTML throughout the VTP.

#### Implementation Details

| Aspect | Implementation |
|--------|---------------|
| ARIA labels | `aria-label` on all interactive elements |
| Focus after Load More | Focus moves to first newly loaded tile |
| Focus on PDP scroll | Focus moves to first visible focusable element |
| Layer focus trapping | `primaryAriaLabel` and `secondaryAriaLabel` for gallery/warranty layers |
| Tab navigation | `role="tablist"`, `role="tab"`, keyboard `onKeyNavigate` |
| External links | `rel="noopener"` with `target="_blank"` |
| Heading hierarchy | h2 for sections, h3 for subsections, h3 for carline name in tiles |
| WCAG target | WCAG 2.0 AA (per Audi platform requirements) |

#### Acceptance Criteria (from REQ-073)

- All interactive elements have `aria-label` attributes
- Focus management after Load More and on PDP scroll
- Layer focus trapping with ARIA labels
- Tab navigation with proper roles and keyboard support
- External links use `rel="noopener"`
- Semantic heading hierarchy maintained

---

### 5.3 Tracking / Analytics

#### 5.3.1 PLP Tracking Events (REQ-074)

##### Description

Key PLP interactions tracked via `audi-tracking-service` (OneSight V2).

##### Tracked Events

| Event | Data |
|-------|------|
| `feature_app_ready` | Search name, results count, filter state, sort option |
| Gallery image change | Image index |
| Sort change | New sort option |
| Go to detail page | Vehicle ID |
| Add/remove favorites | Vehicle ID |
| Load more | Page number |
| Tab click | Tab name |
| CTA clicks | CTA type, vehicle ID |
| Modal layer open/close | Layer type |
| NC/UC switch | Target type |
| Advanced filter open | — |
| Quick filter open/close | Filter name |
| Share | URL |

##### Component Update Data

- `implementer: 2`
- Available filter categories
- Sorting option
- Search name (new/used)
- Result count
- Active filters
- viewType

##### Acceptance Criteria (from REQ-074)

- All listed events tracked
- Component update data includes all listed fields
- All tracking uses OneSight V2

#### 5.3.2 PDP Tracking Events (REQ-075)

##### Description

Key PDP interactions tracked via `audi-tracking-service` (OneSight V2).

##### Tracked Events

| Event | Data |
|-------|------|
| `feature_app_ready` | Product data |
| Gallery open/close | — |
| Image change | Image index |
| Thumbnail toggle | — |
| Favorite add/remove | Vehicle ID |
| Share | URL |
| Audi code | Code value |
| Equipment impression/info click/tyre label | Equipment ID |
| CTA button clicks | CTA type |
| Warranty display | Warranty type |
| Tech data drawer toggle | — |
| "I am interested" click | — |
| GLC checkout | Checkout type |

##### Product Tracking Data

- `productId`: Vehicle ID
- `productName`: Vehicle name
- `manufacturer`: `'Audi'`
- `primaryCategory`: Carline group
- `subCategory1`: Carline
- `productType`: new/used car

##### Acceptance Criteria (from REQ-075)

- All listed events tracked
- Product data includes all listed fields
- All tracking uses OneSight V2

#### 5.3.3 Filter Tracking Events (REQ-076)

##### Description

Filter interactions tracked for user behavior analysis.

##### Tracked Events

- Filter overlay open/close
- Individual filter clicks
- Results count after filter
- Applied filter list

##### Tracking Data

- `componentName: 'vtp-filter'`
- Search name (used/new)
- Applied filters array
- Available categories

##### Acceptance Criteria (from REQ-076)

- All listed filter events tracked
- Tracking data includes component name, search name, filters, and categories

---

### 5.4 SEO

#### Description

SEO considerations for the VTP, derived from the technical analysis.

| Aspect | Implementation |
|--------|---------------|
| SEO-friendly URLs | PDP URLs generated from model year + description + ID |
| Page metadata | PDP sets page metadata via `page-info-service`; Soldout sets 404 status |
| SSR limitation | PLP renders skeletons during SSR — vehicle listing content not available for crawlers in initial HTML |
| SEO filter resolution | `createFilterSeoResolver` resolves SEO-friendly filter slugs before app initialization |
| Carline SEO resolver | `createCarlineSeoResolver` provides carline-specific SEO metadata |

### 5.5 Security

#### Description

Security measures implemented in the VTP.

| Aspect | Implementation |
|--------|---------------|
| HTML sanitization | DOMPurify (^3.2.4) for dealer remarks, product deviations |
| GDPR consent | Google Maps: two-click consent with persistent/per-session option |
| Auth proxy | Authentication routed through server-side proxy (`/qa-userinfo-emea/v2`) |
| API auth | SCS API uses token-based authentication via header |
| External link safety | `rel="noopener"` on `target="_blank"` links |

---

## 6. Technical Design Notes

### Architecture Decisions

1. **Headless Configuration FA Pattern:** Configuration is separated into a dedicated headless Feature App that reads AEM content and broadcasts it via a pub/sub service. This decouples content reading from consuming Feature Apps and ensures consistent configuration across all VTP FAs on a page.

2. **Singleton React Contexts via `Symbol.for()`:** `ServicesContext` and `StockContext` use `Symbol.for()` to create singleton context instances across multiple Feature Apps that share the same React runtime. This enables cross-FA state sharing without requiring a shared context provider at the Feature Hub level.

3. **Dual Design System Support:** Four build artifacts per visual FA support gradual migration from the classic to alpha design system. Webpack aliases remap imports at build time, and runtime helpers (`DynamicComponent`, `dynamicStylingHelper`) select appropriate implementations.

4. **Content-Driven Feature Toggling:** All feature flags are managed through AEM Content Fragments (`scopes_*` fields) rather than a runtime feature flag service. This gives content authors full control but limits rollback speed and percentage-based rollouts.

5. **SSR Strategy Divergence:** PLP renders skeleton-only during SSR (vehicle data fetched client-side) while PDP performs full SSR with real vehicle data. This trade-off prioritizes PDP SEO value and perceived performance, at the cost of PLP crawlability.

6. **Monolithic Shared Library:** `vtp-shared` (v2.7.0) contains all shared components, hooks, types, utils, and contexts as a single monolithically versioned package. Changes affect all consuming Feature Apps simultaneously.

### External Service Dependencies

| Service | Type | Purpose |
|---------|------|---------|
| SCS (Stock Car Service) | REST API | Vehicle data, search, filters, dealers |
| CRS (Car Rating Service) | REST API | Financial products, rate calculations |
| OneGraph | GraphQL | Carline names/groups for model filter |
| Omnigraph | GraphQL (auth proxy) | myAudi wishlist CRUD |
| Google Maps API | JavaScript API | Location autocomplete, geocoding, dealer map |
| AVP (Audi Visualization Platform) | Web Component | 3D vehicle webstreaming |
| Falcon Content API | REST | Trade-In teaser content |
| Partner Business Card FA | Feature App | Dealer information rendering |
| myAudi Auth Service | OAuth/Feature Service | Authentication, token management |
| Audi Tracking Service | Feature Service | OneSight V2 analytics |
| Footnote Services | Feature Service | Legal footnote rendering |
| Notification Display Service | Feature Service | Toast notifications |
| E-Commerce Service | Feature Service | GLC lean checkout |
| Navigation Service | Feature Service | Page navigation for checkout |
| Page Info Service | Feature Service | SEO metadata, HTTP status |
| Env Config Service | Feature Service | Environment-specific config |

### Package Registry

| Package | Type | App Store ID | Version |
|---------|------|-------------|---------|
| `fa-vtp-plp` | Feature App | `1894ccb5-dbea-4a6a-9fd8-068d635f0d66` | 4.19.1 |
| `fa-vtp-pdp` | Feature App | `9877d64f-c7e8-42b7-80eb-d597ba12b311` | 4.30.1 |
| `fa-vtp-configuration` | Feature App (headless) | `b84607fc-e63a-4531-a0f5-92ee1ce147a1` | 5.14.0 |
| `fa-vtp-dealer-info` | Feature App | `9fea9015-4853-4c93-851e-fe338f9c1c19` | 1.17.0 |
| `fa-vtp-soldout` | Feature App (headless) | `852f355b-09dc-4797-a432-a06e6a66bff1` | 1.1.0 |
| `vtp-configuration-service` | Feature Service | N/A | 0.0.1 |
| `vtp-shared` | Library | N/A | 2.7.0 |

---

## 7. Open Issues

The following questions were identified during requirements extraction and remain unresolved:

| # | Question | Impact |
|---|----------|--------|
| 1 | How does the `content-overrides/` directory modify per-market behavior? What specific overrides exist? | May affect market-specific configuration documentation |
| 2 | What PLP and PDP scenarios are covered by the monorepo-level Cypress tests? | Test coverage validation |
| 3 | What is the full user flow for the dynamic finance calculator/layer? | REQ-046 may be incomplete |
| 4 | Is the OneGraph integration (PLP demo component) used in production for vehicle data queries? | REQ-012 dependency clarity |
| 5 | Is the Instavid 360° spin integration actively used in production? | May require additional requirement |
| 6 | Are there plans to introduce LaunchDarkly feature flags? | Affects REQ-063/REQ-064 scope |
| 7 | Is the EU VTP deployed in Japan? The JP-specific price template is referenced in code. | REQ-047 scope clarification |
| 8 | Is a vehicle compare feature in scope? Compare APIs exist in shared. | May require additional requirements |
| 9 | What is the full flow for `availability-notification` CTA type? | REQ-056 completeness |
| 10 | What is the implementation for the `whatsApp` CTA type? | REQ-056 completeness |
| 11 | What events are tracked for the mandatory area search flow? | REQ-017/REQ-074 completeness |
| 12 | Since the soldout FA is headless, how is the visible "sold out" page content rendered? | REQ-061 architecture clarity |
| 13 | Can PLP and PDP exist on the same AEM page, or are they always separate pages? | System composition model |
| 14 | Under what conditions is e-commerce fully disabled (`hideEcom` scope)? | REQ-056/REQ-059 conditions |
| 15 | Are there special display or access rules for employee vehicles (`employeeVehicle` field)? | May require additional requirement |

---

## 8. Appendices

### Appendix A: Feature Service Dependency Matrix

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

### Appendix B: Technology Stack Summary

| Layer | Technology | Version |
|-------|-----------|---------|
| Language | TypeScript | ^5.4.5 |
| UI Framework | React | ^18.2.0 |
| State Management | Redux (`react-redux` ^8.0.2) | Via `@oneaudi/stck-store` 3.4.2 |
| Styling | styled-components | ^5 |
| Microfrontend | Feature Hub (`@feature-hub/react` ^3.6.0) | |
| CMS | AEM Headless | Content Fragment Models |
| GraphQL | `@oneaudi/onegraph-client` ^4.9.2 | |
| Build | webpack + SWC | `@oneaudi/oneaudi-os-build-scripts` ^7.1.2 |
| Unit Testing | Jest 29.7.0 + React Testing Library ^14.3.1 | |
| E2E Testing | Cypress + Cucumber | |
| Maps | Google Maps (`@googlemaps/react-wrapper` ^1.1.42) | |
| HTML Sanitization | DOMPurify ^3.2.4 | |
| Auth | `@oneaudi/audi-auth-service` ^6.1.2 | |
| Node Runtime | v18.20.4 | |
| Infrastructure | AWS CDK (`@oneaudi/oneaudi-os-infrastructure` ^10.0.0) | |

### Appendix C: Inter-App Communication Mechanisms

| Mechanism | Used Between | Purpose |
|-----------|-------------|---------|
| `vtp-configuration-service` (pub/sub) | Configuration → PLP/PDP/dealer-info | Broadcast VTPConfiguration |
| URL hash | PLP → PDP (and back) | Filter state persistence |
| Custom DOM events | External → PDP | `pdp:open-warranties-tab` programmatic navigation |
| localStorage | PLP ↔ PDP | Selected vehicle, filter state, location, wishlist pending add |
| sessionStorage | Within PDP | AOZ selections, finance data |
| Feature Services (DI) | Feature Hub → all FAs | Locale, i18n, auth, tracking, content, footnotes, notifications, e-commerce |
| URL navigation | PLP → PDP | PDP URL from configurable pattern with placeholders |

### Appendix D: Content Fragment Model Hierarchy

```
AEM Content Fragments
├── VTP Configuration (shared, referenced by PLP + PDP)
│   ├── scopes_*           → Feature toggles
│   ├── urls_*             → External links
│   ├── assets_*           → Content reference assets
│   ├── cta[]              → CTA button definitions
│   │   └── options_*      → Per-CTA options
│   ├── scs_*              → SCS API path
│   ├── sortParams_*       → Sort options + default
│   ├── consumptionEmission_* → WLTP/NEDC display settings
│   ├── financeLayer_*     → Finance layer settings
│   ├── mainPriceConfiguration → Price type mappings
│   │   └── items[]        → Price fragment definitions
│   └── priceConfiguration → Price breakdown configuration
│
├── Feature App - PLP (per PLP instance)
│   ├── appContext          → results | favorites
│   ├── isNewCarsPage       → NC/UC toggle
│   ├── detailsPageUrlPattern → PDP URL template
│   ├── quickFilters        → Up to 12 quick filter slots
│   ├── filterCategories    → Up to 7 filter tab categories
│   ├── carlinePhotos       → Override carline thumbnail images
│   ├── filterInfoLayers    → Info button URL mappings
│   ├── equipmentFilter_*   → Equipment filter configuration
│   ├── locationFilterConfig_* → Google Maps configuration
│   ├── campaigns           → Campaign configuration
│   └── vtpConfiguration    → Reference to central configuration
│
├── Feature App - PDP (per PDP instance)
│   ├── searchLink          → Back navigation URL
│   ├── use3dWebstreaming   → 3D view toggle
│   ├── avpRessourceUrl     → 3D script URL
│   ├── nbas               → Next Best Actions selection
│   ├── warrantyInfoLayer   → Warranty type info URL mappings
│   ├── scsTechdataInitial/Extended → Tech data field selection
│   ├── tradeInTeaser       → Trade-in editorial content
│   └── vtpConfiguration    → Reference to central configuration
│
└── Feature App - Dealer Info
    ├── appVersion          → Partner Business Card FA version
    └── vtpDealerResultsURL → Dealer results link pattern
```
