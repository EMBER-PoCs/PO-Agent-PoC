# Project: Product Delivery Pipeline

## Purpose
This workspace reverse-engineers requirements from existing Audi Feature App
codebases and transforms them into functional specifications through a
coordinated agent workflow.

## Domain Context
This project deals with Audi's **Vehicle Transaction Pages (VTP)** — the vehicle
listing and vehicle details experience. Feature Apps are React-based microfrontends
that render within AEM (Adobe Experience Manager) pages via the **Feature Hub**.
See the `feature-app-knowledge` skill for full architectural details.

## Source Repositories
- `source-repos/audi-eu-vtp/` — European VTP monorepo (git submodule, READ-ONLY)
  - Monorepo structure: `packages/`, `shared/`, `content-overrides/`
  - DO NOT modify, create branches, or create PRs against this repo

## Directory Conventions
- Raw/extracted requirements go in `requirements/`
- Functional specifications go in `specs/`
- Review reports go in `reviews/`
- Source codebases for analysis go in `source-repos/` (as submodules)
- Planning documents go in `planning/`

## Document Standards
- All documents use Markdown format
- Every requirement must have a unique ID (e.g., REQ-001)
- Specifications reference requirement IDs for traceability

## Safety Rules
- NEVER modify files inside `source-repos/` — they are read-only references
- NEVER create branches or PRs against source repositories
- All agent output (requirements, specs, reviews) stays in this repo only
