---
name: feature-app-knowledge
description: "Domain knowledge about Audi Feature Apps — microfrontend architecture, Feature Hub, AEM integration, feature services, SSR/CSR rendering, deployment, and the App Store. Use when analyzing feature app codebases, extracting requirements from feature apps, or understanding Audi's component architecture."
---

# Audi Feature App Knowledge Base

## What Is a Feature App?

A **Feature App** (FA) is a **microfrontend** — a self-contained, independently deployable UI component that renders within an Audi website page. Feature Apps are the building blocks of Audi's web presence across all markets.

### Core Architecture

```
┌─────────────────────────────────────────────────┐
│                  AEM (CMS)                      │  ← Adobe Experience Manager hosts pages
│         Page skeleton + content                 │
└────────────────┬────────────────────────────────┘
                 │
    ┌────────────┴────────────────┐
    ▼                             ▼
┌──────────────────┐   ┌──────────────────────┐
│  CSR Integrator  │   │  Renderman (SSR)     │
│  (Browser)       │   │  (Server-side proxy) │
└───────┬──────────┘   └───────┬──────────────┘
        │                      │
        ▼                      ▼
┌──────────────────────────────────────────┐
│         Feature Hub                       │
│  (Manages FA lifecycle, dependencies,     │
│   feature services, shared state)         │
└───────┬──────────┬───────────┬───────────┘
        ▼          ▼           ▼
   ┌────────┐ ┌────────┐ ┌────────┐
   │  FA 1  │ │  FA 2  │ │  FA 3  │   ← Independent microfrontends
   └────────┘ └────────┘ └────────┘
```

### Key Components

| Component | What It Is | Role |
|-----------|-----------|------|
| **AEM** | Adobe Experience Manager | CMS that hosts Audi country websites, provides page skeleton and content |
| **Feature Hub** | Microfrontend orchestrator | Manages Feature App lifecycle, dependency injection, shared services |
| **Feature Hub Integrator (CSR)** | Client-side renderer | Renders Feature Apps in the user's browser |
| **Renderman (SSR)** | Server-side rendering proxy | Reads page skeleton from AEM, generates Feature App markup before delivery to client. Critical for performance. |
| **Feature Services** | Shared capabilities | Services provided by the Feature Hub to Feature Apps (logging, locale, auth, tracking, config) |
| **App Store** | Deployment registry | Where Feature Apps are registered, versioned, and made available to AEM pages |
| **FAWI** | Feature App Wizard | Tool at https://fawi.one.audi for generating Feature App boilerplate |

### Rendering Modes

- **CSR (Client-Side Rendering):** Feature Apps render in the browser via the client-side integrator
- **SSR (Server-Side Rendering):** Renderman pre-renders Feature App markup on the server before page delivery. **Required for performance goals.** All FAs should support SSR.
- The CSR and SSR integrators provide **different** sets of feature services (e.g., tracking and auth are CSR-only)

### Feature Services

Feature Services are shared capabilities injected into Feature Apps by the Feature Hub:

| Type | Prefix | Notes |
|------|--------|-------|
| **GFA Feature Services** | `gfa:` | Group Frontend Architecture — shared across all VW brands. Very rigid, slow to change. |
| **Audi Feature Services** | `s2:` or Audi-specific | Audi-only versions with additional capabilities. Easier to extend. |

**Rule:** If a Feature App is NOT a whitelabel/cross-brand app, use the Audi-specific service versions. Use GFA services only if the app must port to other VW brands.

Key repos:
- Feature services: `oneaudi/audi-feature-services`
- CSR integrator: `oneaudi/audi-feature-hub-integrator-csr`
- SSR integrator: `oneaudi/audi-feature-hub-integrator-ssr`

## Feature App Lifecycle

### 1. Creation
- Generated via **FAWI** (Feature App Wizard) at https://fawi.one.audi
- Boilerplate includes: build scripts, webpack config, CI jobs, SSR setup, Cypress/Lighthouse tests
- Updates via **Bilbo** tool
- Uses `@volkswagen-onehub/oneaudi-os-build-scripts` for build configuration

### 2. Development
- React-based microfrontends
- Content from AEM via **Content Fragment Models** (CFM)
- Local dev: `npm run serve` (CSR) or `npm run dev:ssr` (SSR)
- Testing: Jest + Enzyme/RTL (unit), Cypress (E2E/acceptance)
- Conventional commits enforced

### 3. Content Management
- Content authored in **AEM Headless** via Content Fragment Models
- Edited in the **Universal Editor** (UE)
- Content Fragments define the structured data a Feature App consumes
- Models follow naming convention: "My Feature App" (root), "My Feature App: Item" (child)

### 4. Deployment
- Feature Apps deployed to **App Store** (replaces deprecated AEM Registry and Marketplace)
- Available in AEM via "Generic Feature App Include" component
- Version mappings: **global** (all markets) or **market-specific** (overrides global)
- Environments: Test → Pre-Live → Live
- Live deployments managed by Agnosco team

### 5. Support Readiness
Required before handoff:
- [ ] Complete documentation (project summary, handbooks, ADL)
- [ ] C4 architecture diagrams (Context, Container, Component, Code)
- [ ] API specifications (GraphQL/REST)
- [ ] 80%+ Jest unit test coverage
- [ ] 80%+ Cypress E2E automation
- [ ] WCAG 2.0 AA accessibility compliance
- [ ] No open security findings (dependabot/npm audit)
- [ ] App Store registration complete
- [ ] Figma designs exported as flat files

## VTP Context (Vehicle Transaction Pages)

The **VTP** (Vehicle Transaction Pages) is a set of Feature Apps that power Audi's vehicle shopping experience:

- **Vehicle Listing Page (VLP):** Browse/search available vehicles with filters
- **Vehicle Details Page (VDP):** Detailed view of a specific vehicle

### EU VTP vs US/Canada VTP

| Aspect | EU VTP | US/Canada VTP |
|--------|--------|---------------|
| **Architecture** | Monolithic app (`oneaudi/vtp`) | Multiple smaller Feature Apps composed together |
| **Scope** | Single monorepo with packages/ | Distributed across many repos |
| **Legal requirements** | EU-wide regulations | Per-state legal requirements (50 states + provinces) |
| **Complexity driver** | Single codebase, many markets | Many apps, many jurisdictions |

## Open Questions for Future Research

1. What specific Feature Apps compose the US/Canada VTP? (repo names, app IDs)
2. What are the per-state legal requirements that drive feature differences?
3. How does the content-overrides system work in the EU VTP monorepo?
4. What Feature Services does the VTP specifically consume?
5. How does the VTP handle market-specific vehicle data APIs?
6. What is the relationship between VTP and the broader Audi configurator flow?
7. How does LaunchDarkly feature flagging integrate with VTP deployments?
8. What are the EMEA vs NAR AEM environment differences?

## Key Document Slugs (RED Docs Portal)

For deeper research, query these documents:
- `/development/tools/fawi_docs/Basic Development/what-is` — Architecture overview
- `/development/tools/fawi_docs/Basic Development/Initial Setup/Create your FeatureApp Boilerplate` — FA creation
- `/development/tools/fawi_docs/Basic Development/FeatureApp development/Available FeatureServices/index` — Feature services
- `/development/tools/fawi_docs/Basic Development/FeatureApp development/Available FeatureServices/Audi vs GFA FeatureServices` — Audi vs GFA services
- `/development/tools/fawi_docs/Basic Development/FeatureApp development/Server Side Rendering/SSR - FeatureApp preparations` — SSR
- `/development/tools/fawi_docs/Basic Development/Deployment/index` — Deployment & App Store
- `/product_readiness/ProductReadiness` — Support readiness checklist
- `/getting-started/README` — Onboarding overview
