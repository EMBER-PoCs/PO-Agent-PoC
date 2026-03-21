# Repository Info

## PO-Agent-PoC
- **URL:** https://github.com/EMBER-PoCs/PO-Agent-PoC
- **Visibility:** Public (open to EMBER-PoCs org members)
- **Purpose:** Product Owner Agent PoC — coordinated agent family for extracting requirements and generating functional specifications

## Branches
- `main` — clean template (no source repos, reusable for new projects)
- `audi-eu-vtp` — Audi EU VTP work (submodule reference to source repo)

## Source Repos

### Audi EU VTP
- **URL:** https://github.com/oneaudi/vtp
- **Location:** `source-repos/audi-eu-vtp/` (git submodule, read-only)
- **Description:** European market Vehicle Transaction Pages (vehicle listing page, vehicle details page). Monolithic app.
- **Structure:** Monorepo with `packages/`, `shared/`, `content-overrides/`, Cypress tests

## Future Scope
- **Audi USA / Audi Canada VTP:** American market equivalent. Not monolithic — composed of many smaller apps. Has additional per-state legal requirements. Source repo TBD.

## Safety Rules
- NEVER create branches on `oneaudi/vtp`
- NEVER create PRs against `oneaudi/vtp`
- All agent output (requirements, specs, reviews) stays in this repo only
