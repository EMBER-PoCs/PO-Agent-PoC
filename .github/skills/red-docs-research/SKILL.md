---
name: red-docs-research
description: "Search and retrieve Audi RED documentation via the red-docs-port MCP server. Use when researching Audi platform architecture, feature app development patterns, backend services, deployment processes, or any Audi-specific technical documentation."
---

# RED Docs Portal Research

## What Is the RED Docs Portal?

The Audi RED (Retail Experience & Digital) documentation portal is a Docusaurus-based knowledge base containing all technical documentation for the Audi digital platform. It is accessible via the `mcp_red-docs-port` MCP server.

## Available MCP Tools

| Tool | Purpose | When to Use |
|------|---------|-------------|
| `mcp_red-docs-port_search_docs` | Semantic search across all docs | Finding docs by topic, keyword, or concept |
| `mcp_red-docs-port_get_document` | Fetch full document by ID | Deep-reading a specific document |
| `mcp_red-docs-port_get_related_docs` | Find related documents | Expanding research from a known document |
| `mcp_red-docs-port_get_category_docs` | List docs in a category | Browsing a documentation section |
| `mcp_red-docs-port_get_backend_service_guidelines` | Backend service patterns | NestJS, GraphQL, API design |
| `mcp_red-docs-port_create_feature_app` | Generate feature app scaffold | Creating new feature apps |

## Search Strategy

### Effective Search Queries
- Use **domain-specific terms**: "feature app", "feature hub", "renderman", "AEM", "content fragment"
- Combine **concept + context**: "SSR feature app preparations rendering"
- Include **role keywords**: "content author", "developer", "architect"
- Use `top: 10` for broad discovery, `top: 3` for focused retrieval

### Key Document Collections
| Collection | Path Pattern | Content |
|-----------|-------------|---------|
| Feature App Development | `development/tools/fawi_docs/**` | FA creation, setup, services, SSR, deployment |
| Backend Services | `development/backend-service/**` | NestJS, GraphQL, feature flags |
| Getting Started | `getting-started/**` | Onboarding, prerequisites |
| Product Readiness | `product_readiness/**` | Support checklists, handoff criteria |
| Content Author Guides | `development/tools/fawi_docs/content_author_guide/**` | AEM content management |

### Document Metadata Tags
Search results include structured metadata useful for filtering:
- `discipline`: development, product, enablement
- `lifecycle_stage`: discover, build, operate
- `artifact_type`: reference, guide, checklist, guidelines
- `systems`: feature-app-setup, engineering-tooling, backend-services, aem
- `roles`: frontend-engineer, backend-engineer, product-manager, solution-architect

## Usage Procedure

1. **Start broad** — search with conceptual terms to discover what exists
2. **Note doc_ids** — from search results, capture `doc_id` values for full retrieval
3. **Fetch full docs** — use `get_document` with the `id` parameter (use the `doc_id` from search)
4. **Expand** — use `get_related_docs` with `documentId` to find connected content
5. **Synthesize** — combine findings into structured knowledge

## Constraints
- The `get_document` tool requires an `id` parameter (use `doc_id` from search results)
- The `get_related_docs` tool requires a `documentId` parameter
- Document IDs with spaces or special characters may need URL encoding
- Search is semantic — natural language queries work well
