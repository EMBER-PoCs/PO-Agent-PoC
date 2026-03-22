# PO-Agent-PoC: Automated VTP Requirements Extraction

## The Mission

Produce a **combined, unified set of requirements** for Audi's Vehicle Transaction Pages (VTP) across two markets:

| Market | Codebase | Architecture | Status |
|--------|----------|-------------|--------|
| **European VTP** | `oneaudi/vtp` — single monorepo | Monolithic Feature App with packages (PLP, PDP, configuration, dealer-info, soldout, etc.) | Source added as submodule at `source-repos/audi-eu-vtp/` |
| **American VTP** | Multiple repos (TBD) | Multiple smaller Feature Apps composed together; additional per-state legal requirements | Not yet added |

The end goal is a set of requirements documents and functional specifications that capture **what both VTPs do today** — extracted directly from code — and a **gap analysis** showing where they diverge.

## The Approach: Agents That Do the Work

Rather than manually reading thousands of source files and writing requirements by hand, we've built a **family of coordinated AI agents** that automate the extraction pipeline. The agents are the tool; the requirements are the product.

This is a dual-purpose effort:

1. **The deliverable:** A combined requirements set for EU and US VTP, with gap analysis
2. **The capability:** A reusable, automated pipeline that can extract requirements from any Feature App codebase

## The Agent Family

Six specialized agents work together in a pipeline, orchestrated by a coordinator:

```
 You
  │
  ▼
┌─────────────────────────────────────────────────────┐
│           Product Delivery Coordinator               │  ← The only agent you talk to
│   Orchestrates the full pipeline, delegates to all   │
│   workers, synthesizes results, iterates on quality  │
└──────────┬──────────────────────────────────────────┘
           │
           │  Phase 0 (Mode B: from code)
           ▼
    ┌──────────────┐
    │    Code       │  Reads source code and produces raw findings:
    │ Archaeologist │  features, data models, APIs, business rules,
    │               │  market variations, feature flags, edge cases
    └──────┬───────┘
           │
           │  Phase 1 (parallel)
           ├────────────────────────┐
           ▼                        ▼
    ┌──────────────┐        ┌──────────────┐
    │   Business   │        │   Solution   │
    │   Analyst    │        │   Architect  │
    │              │        │              │
    │  Structures  │        │  Maps reqs   │
    │  raw findings│        │  to technical│
    │  into formal │        │  components, │
    │  requirements│        │  risks, and  │
    │  (REQ-001…)  │        │  feasibility │
    └──────┬───────┘        └──────┬───────┘
           │                        │
           │  Phase 2 (synthesis)   │
           ├────────────────────────┘
           ▼
    ┌──────────────┐
    │  Spec Writer │  Produces the functional specification,
    │              │  merging requirements + technical analysis
    └──────┬───────┘
           │
           │  Phase 3 (review loop)
           ▼
    ┌──────────────┐
    │ Spec Reviewer│  Quality-checks the spec; if issues found,
    │              │  sends it back for revision
    └──────────────┘
```

### Agent Details

| Agent | Role | Tools | Key Capability |
|-------|------|-------|----------------|
| **Product Delivery** | Coordinator / Project Manager | Subagent invocation, read, search, edit, todo | Orchestrates pipeline; detects Mode A (from docs) vs Mode B (from code) |
| **Code Archaeologist** | Reverse-engineering specialist | Read, search (read-only) | Extracts features, business rules, data models, APIs, feature flags, market variations, and test insights from source code |
| **Business Analyst** | Requirements structuring | Read, search (read-only) | Transforms raw findings into formal user stories with acceptance criteria, MoSCoW prioritization |
| **Solution Architect** | Technical analysis | Read, search, RED Docs MCP | Maps requirements to technical components; has access to Audi platform documentation via MCP |
| **Spec Writer** | Technical documentation | Read, search, edit | Writes functional specifications with full requirement traceability |
| **Spec Reviewer** | Quality assurance | Read, search (read-only) | Reviews specs against requirements and domain-specific checklist (market variations, SSR, WCAG 2.0 AA, content author config) |

### Design Principles

- **Context isolation**: Each agent runs in its own context window — the Business Analyst isn't distracted by architecture concerns
- **Least privilege**: Only the Spec Writer can edit files; analysts and reviewers are read-only
- **Parallelization**: Business Analyst and Solution Architect run simultaneously
- **Iterative quality**: Writer ↔ Reviewer loop continues until the spec passes review
- **Traceability**: Requirement IDs (REQ-NNN) flow from Archaeologist → Analyst → Writer → Reviewer

## Supporting Knowledge

Two skills provide on-demand domain knowledge to agents that need it:

| Skill | Purpose |
|-------|---------|
| **feature-app-knowledge** | Architecture reference: AEM, Feature Hub, CSR/SSR, Feature Services, deployment, App Store |
| **red-docs-research** | How to query Audi's RED documentation portal via MCP tools |

## Workspace Structure

```
.github/
├── agents/                          ← Agent definitions (6 agents)
├── instructions/                    ← Shared instruction files
│   └── spec-standards.instructions.md
├── skills/                          ← On-demand knowledge
│   ├── feature-app-knowledge/
│   └── red-docs-research/
└── copilot-instructions.md          ← Always-on workspace rules

source-repos/
└── audi-eu-vtp/                     ← EU VTP monorepo (git submodule, READ-ONLY)

requirements/                        ← Output: extracted requirements
specs/                               ← Output: functional specifications
reviews/                             ← Output: review reports
planning/                            ← Planning and reference docs
```

## Pipeline Outputs

For each VTP package, the pipeline produces:

1. **Code Archaeology Report** — raw inventory of what the code does (features, rules, APIs, flags)
2. **Structured Requirements** — formal user stories with acceptance criteria (REQ-NNN format)
3. **Technical Analysis** — architecture mapping, risks, feasibility assessment
4. **Functional Specification** — complete spec with traceability matrix
5. **Review Report** — quality verdict (APPROVED / REVISION REQUIRED)

Once both VTPs are extracted, a **gap analysis** compares EU vs US requirements to identify shared functionality and market-specific divergences.

## Branch Strategy

| Branch | Purpose |
|--------|---------|
| `main` | Clean template — no source repos, reusable for future projects |
| `audi-eu-vtp` | Active working branch for EU VTP extraction |

A `pre-extraction` tag marks the checkpoint before any pipeline output, enabling safe rollback.

## Safety Rules

- **NEVER** modify files inside `source-repos/` — they are read-only references
- **NEVER** create branches or PRs against source repositories
- All agent output stays in this repo only
