# Building a Product Delivery Agent Family in GitHub Copilot

A complete end-to-end guide for creating a family of coordinated AI sub-agents in VS Code that transform business requirements into functional specifications.

---

## Table of Contents

1. [Core Concepts](#part-1-core-concepts-you-need-to-understand)
2. [Architecture — The Product Delivery Agent Family](#part-2-architecture--the-product-delivery-agent-family)
3. [Step-by-Step Implementation](#part-3-step-by-step-implementation)
4. [How to Use It](#part-4-how-to-use-it)
5. [Key Design Principles](#part-5-key-design-principles)
6. [Extending the Family](#part-6-extending-the-family)
7. [Quick-Start Checklist](#quick-start-checklist)

---

## Part 1: Core Concepts You Need to Understand

### What Are Custom Agents?

Custom agents are `.agent.md` Markdown files that define an AI persona with specific instructions, tools, and behaviors. Each agent gets defined in `.github/agents/` in your workspace. When an agent invokes another agent as a **subagent**, that subagent gets **its own context window** (isolated from the parent) but **shares the same file system**.

### How Subagents Work (Context Isolation)

This is the key architectural feature:

1. A **coordinator agent** receives your request
2. It delegates subtasks to **worker subagents** using `runSubagent`
3. Each subagent runs in a **fresh, isolated context** — it doesn't see the parent's conversation history
4. The subagent works autonomously, then returns a **single summary** back to the coordinator
5. The coordinator synthesizes results and continues

This means each specialist can focus deeply without being polluted by unrelated context, while still reading/writing the same project files.

### What the User Sees

When a subagent runs, it appears in the chat as a collapsible tool call. By default, the subagent is collapsed and shows:

- The name of the custom agent (if you specify one)
- The currently running tool (for example, "Reading file..." or "Searching codebase...")

Select the subagent tool call to expand it and view the full details, including all tool calls the subagent made, the prompt passed to the subagent, and the returned result.

### The PROSE Framework

PROSE is a structured prompt engineering framework for designing effective agent instructions. The acronym stands for:

| Letter | Meaning | Application to Agent Design |
|--------|---------|---------------------------|
| **P** | **Purpose** | The single, clear mission of the agent — what problem does it solve? |
| **R** | **Role** | The persona and expertise the agent embodies — who is it? |
| **O** | **Output** | The exact format and structure of what the agent produces |
| **S** | **Scope** | The boundaries — what it should and should NOT do, which tools it has access to |
| **E** | **Evaluation** | How to verify quality — criteria, checklists, review standards |

Every agent file you create should address all five PROSE dimensions. This keeps each agent focused and effective.

### VS Code Customization Primitives

Before building agents, understand the building blocks available:

| Primitive | File Type | Location | When to Use |
|-----------|-----------|----------|-------------|
| Workspace Instructions | `copilot-instructions.md` or `AGENTS.md` | `.github/` or root | Always-on, applies everywhere in the project |
| File Instructions | `*.instructions.md` | `.github/instructions/` | Explicit via `applyTo` patterns, or on-demand via `description` |
| Custom Agents | `*.agent.md` | `.github/agents/` | Subagents for context isolation, or multi-stage workflows with tool restrictions |
| Prompts | `*.prompt.md` | `.github/prompts/` | Single focused task with parameterized inputs |
| Skills | `SKILL.md` | `.github/skills/<name>/` | On-demand workflow with bundled assets (scripts/templates) |

### Agent File Structure

Agent files are Markdown with optional YAML frontmatter:

```yaml
---
description: "<required>"    # For agent picker and subagent discovery
name: "Agent Name"           # Optional, defaults to filename
tools: [search, web]         # Optional: aliases, MCP (<server>/*), extension tools
model: "Claude Sonnet 4"     # Optional, uses picker default
agents: [agent1, agent2]     # Optional, restrict allowed subagents by name
user-invocable: true         # Optional, show in agent picker (default: true)
disable-model-invocation: false  # Optional, prevent subagent invocation (default: false)
handoffs: [...]              # Optional, transitions to other agents
---
```

#### Tool Aliases

| Alias | Purpose |
|-------|---------|
| `execute` | Run shell commands |
| `read` | Read file contents |
| `edit` | Edit files |
| `search` | Search files or text |
| `agent` | Invoke custom agents as subagents |
| `web` | Fetch URLs and web search |
| `todo` | Manage task lists |

#### Invocation Control

| Attribute | Default | Effect |
|-----------|---------|--------|
| `user-invocable: false` | `true` | Hide from agent picker, only accessible as subagent |
| `disable-model-invocation: true` | `false` | Prevent other agents from invoking as subagent |

#### Restricting Subagents

The `agents` property accepts:

- A list of agent names (e.g., `['Edit', 'Search']`) to allow only specific agents
- `*` to allow all available agents (default behavior)
- An empty array `[]` to prevent any subagent use

> **Important**: Explicitly listing an agent in the `agents` array overrides `disable-model-invocation: true`. This means you can create agents that are protected from general subagent use but still accessible to specific coordinator agents that explicitly allow them.

---

## Part 2: Architecture — The Product Delivery Agent Family

### Family Structure

```
┌──────────────────────────────────────────────┐
│        Product Delivery Coordinator          │  ← You talk to this one
│   (Orchestrates the full workflow)           │
└──────────────┬───────────────────────────────┘
               │
    ┌──────────┼──────────┬──────────────┐
    ▼          ▼          ▼              ▼
┌────────┐ ┌────────┐ ┌──────────┐ ┌──────────┐
│Business│ │Solution│ │  Spec    │ │  Spec    │
│Analyst │ │Archi-  │ │  Writer  │ │  Reviewer│
│        │ │tect    │ │          │ │          │
└────────┘ └────────┘ └──────────┘ └──────────┘
   Step 1    Step 2     Step 3       Step 4
```

**Parallelization opportunity**: Steps 1 and 2 can partially overlap — the Business Analyst gathers requirements while the Solution Architect researches existing codebase patterns. The coordinator can run them in parallel.

### Agent Roles (PROSE Applied)

| Agent | Purpose (P) | Role (R) | Output (O) | Scope (S) | Evaluation (E) |
|-------|-------------|----------|------------|-----------|----------------|
| **Coordinator** | Orchestrate the full BRD→spec pipeline | Project Manager | Final functional spec | Delegates, never writes spec directly | All sections complete, reviewed |
| **Business Analyst** | Parse & structure raw business requirements | BA expert | Structured requirements doc (user stories, acceptance criteria) | Read-only, research only | Every requirement has acceptance criteria |
| **Solution Architect** | Map requirements to technical approach | Technical architect | Technical design choices, component mapping | Read-only, research codebase | Feasibility validated against existing code |
| **Spec Writer** | Produce the functional specification | Technical writer | Complete functional spec document | Read + edit (writes the spec) | Covers all requirements, clear format |
| **Spec Reviewer** | Quality-check the spec | QA/Review specialist | Review report with issues and sign-off | Read-only | No gaps, no ambiguity, traceable to requirements |

### Orchestration Patterns

#### Coordinator and Worker Pattern

A coordinator agent manages the overall task and delegates subtasks to specialized subagents. Each worker agent can have a tailored set of tools. For example, planning and review agents need only read-only access, while the implementer needs edit capabilities.

This pattern keeps the coordinator's context focused on the high-level workflow while each worker agent has a clean context and appropriate permissions for its specific job.

#### Multi-Perspective Pattern (Parallelizable)

Multiple subagents can run in parallel when their tasks are independent. In this family, the Business Analyst and Solution Architect can run simultaneously since they operate on different concerns (business requirements vs. technical architecture) and both are read-only.

---

## Part 3: Step-by-Step Implementation

### Step 0: Create Your Workspace Structure

Create a project folder and set up the directory structure:

```
your-project/
├── .github/
│   ├── agents/                    ← Agent definitions go here
│   │   ├── product-delivery.agent.md
│   │   ├── business-analyst.agent.md
│   │   ├── solution-architect.agent.md
│   │   ├── spec-writer.agent.md
│   │   └── spec-reviewer.agent.md
│   ├── instructions/              ← Shared instructions
│   │   └── spec-standards.instructions.md
│   └── copilot-instructions.md   ← Project-wide defaults
├── requirements/                  ← Input: raw business requirements
├── specs/                         ← Output: functional specifications
└── reviews/                       ← Output: review reports
```

### Step 1: Create the Workspace Instructions

Create `.github/copilot-instructions.md` — this applies to ALL agents automatically:

```markdown
# Project: Product Delivery Pipeline

## Purpose
This workspace transforms business requirements into functional specifications
through a coordinated agent workflow.

## Directory Conventions
- Raw business requirements go in `requirements/`
- Functional specifications go in `specs/`
- Review reports go in `reviews/`

## Document Standards
- All documents use Markdown format
- Every requirement must have a unique ID (e.g., REQ-001)
- Specifications reference requirement IDs for traceability
```

### Step 2: Create Each Agent File

#### 2a. The Coordinator — `product-delivery.agent.md`

```markdown
---
description: "Use when transforming business requirements into functional specifications. Orchestrates the full product delivery pipeline from requirements gathering through spec review."
tools: ['agent', 'read', 'search', 'edit', 'todo']
agents: ['business-analyst', 'solution-architect', 'spec-writer', 'spec-reviewer']
---

You are a Product Delivery Coordinator. Your job is to orchestrate the transformation of business requirements into a complete, reviewed functional specification.

## Workflow

Follow these steps in order:

### Phase 1: Discovery (Parallelizable)
Run these two subagents IN PARALLEL:
1. Use the **business-analyst** agent to analyze the raw requirements and produce structured user stories with acceptance criteria. Pass the requirements content directly.
2. Use the **solution-architect** agent to research the existing codebase and identify relevant patterns, existing components, and technical constraints.

### Phase 2: Synthesis
After both Phase 1 agents complete:
- Review the Business Analyst's structured requirements and the Solution Architect's technical analysis
- Identify any gaps or conflicts between business needs and technical reality
- If gaps exist, re-run the business-analyst with clarification prompts

### Phase 3: Specification
- Use the **spec-writer** agent to produce the functional specification
- Pass it BOTH the structured requirements AND the technical analysis as input

### Phase 4: Review
- Use the **spec-reviewer** agent to review the specification
- If the reviewer identifies critical issues, send the spec back to the spec-writer with the review feedback for revision
- Iterate between spec-writer and spec-reviewer until the review passes

## Output
Save the final specification to `specs/` and the review report to `reviews/`.
Present a summary of what was produced.

## Constraints
- DO NOT write the specification yourself — always delegate to the spec-writer
- DO NOT skip the review phase
- ALWAYS run business-analyst and solution-architect in parallel when possible
```

#### 2b. Business Analyst — `business-analyst.agent.md`

```markdown
---
description: "Use when analyzing raw business requirements, extracting user stories, and defining acceptance criteria. Specialist in requirements elicitation and structuring."
tools: ['read', 'search']
user-invocable: false
---

You are a senior Business Analyst. Your job is to take raw, unstructured business requirements and transform them into clear, structured deliverables.

## Approach
1. Read the provided business requirements carefully
2. Identify all stakeholders mentioned or implied
3. Extract discrete user stories in standard format
4. Define measurable acceptance criteria for each story
5. Identify assumptions, dependencies, and open questions
6. Prioritize using MoSCoW (Must/Should/Could/Won't)

## Output Format

Return a structured requirements document in this exact format:

```
# Structured Requirements: [Project Name]

## Stakeholders
- [Role]: [Description of needs]

## User Stories

### REQ-001: [Title]
**As a** [role], **I want** [capability], **so that** [benefit].
**Priority:** Must/Should/Could/Won't
**Acceptance Criteria:**
- [ ] [Criterion 1]
- [ ] [Criterion 2]

## Assumptions
- [Assumption 1]

## Open Questions
- [Question 1]

## Dependencies
- [Dependency 1]
```

## Constraints
- DO NOT make technical design decisions — that's the architect's job
- DO NOT invent requirements not present or clearly implied in the source material
- ALWAYS flag ambiguity as an open question rather than assuming
```

#### 2c. Solution Architect — `solution-architect.agent.md`

```markdown
---
description: "Use when researching codebase patterns, evaluating technical feasibility, and mapping requirements to technical components. Specialist in technical architecture and design patterns."
tools: ['read', 'search']
user-invocable: false
---

You are a Solution Architect. Your job is to analyze the existing codebase and map business requirements to a viable technical approach.

## Approach
1. Search the codebase for existing patterns, components, and architecture
2. Identify which existing modules relate to the new requirements
3. Assess technical feasibility and complexity
4. Recommend build vs. reuse decisions
5. Flag technical risks and constraints
6. Propose a high-level component architecture

## Output Format

Return a technical analysis in this format:

```
# Technical Analysis: [Project Name]

## Existing Architecture Summary
- [Key components and their relationships]

## Relevant Existing Code
- [File/module]: [What it does and how it relates]

## Technical Approach
### Component Mapping
| Requirement | Proposed Approach | Complexity | Reuse Opportunity |
|-------------|-------------------|------------|-------------------|

## Technical Risks
- [Risk 1]: [Impact and mitigation]

## Non-Functional Considerations
- Performance: [Notes]
- Security: [Notes]
- Scalability: [Notes]

## Recommended Tech Stack / Libraries
- [Technology]: [Justification]
```

## Constraints
- DO NOT modify any code — you are read-only
- DO NOT make product decisions — only technical ones
- ALWAYS ground recommendations in what you find in the actual codebase
```

#### 2d. Spec Writer — `spec-writer.agent.md`

```markdown
---
description: "Use when writing functional specifications from structured requirements and technical analysis. Specialist in technical documentation and specification writing."
tools: ['read', 'search', 'edit']
user-invocable: false
---

You are a Functional Specification Writer. Your job is to synthesize structured requirements and technical analysis into a comprehensive functional specification document.

## Approach
1. Review the structured requirements (user stories, acceptance criteria)
2. Review the technical analysis (architecture, component mapping)
3. Write each section of the spec, ensuring full requirement traceability
4. Include wireframe descriptions where applicable
5. Define data models, API contracts, and state transitions as needed
6. Save the specification to the `specs/` directory

## Output Format

Write the specification as a Markdown file with this structure:

```
# Functional Specification: [Project Name]
**Version:** 1.0
**Date:** [Date]
**Status:** Draft / In Review / Approved

## 1. Overview
[Executive summary of what is being built and why]

## 2. Goals and Non-Goals
### Goals
### Non-Goals

## 3. Requirements Traceability
| Spec Section | Requirement ID | Status |
|-------------|----------------|--------|

## 4. Functional Requirements
### 4.1 [Feature Area]
#### Description
#### User Flow
#### Business Rules
#### Data Model
#### Acceptance Criteria (from REQ-XXX)

## 5. Non-Functional Requirements
### Performance
### Security
### Accessibility

## 6. Technical Design Notes
[Key architecture decisions from the technical analysis]

## 7. Open Issues
[Unresolved questions from requirements gathering]

## 8. Appendices
```

## Constraints
- DO NOT invent requirements — only specify what's in the structured requirements
- ALWAYS maintain traceability back to requirement IDs
- If the review report identifies issues, address every one explicitly
```

#### 2e. Spec Reviewer — `spec-reviewer.agent.md`

```markdown
---
description: "Use when reviewing functional specifications for completeness, clarity, and correctness. Specialist in quality assurance and specification review."
tools: ['read', 'search']
user-invocable: false
---

You are a Specification Reviewer. Your job is to critically review a functional specification and produce a structured review report.

## Approach
1. Read the specification thoroughly
2. Cross-reference against the structured requirements — is every REQ-XXX covered?
3. Check for ambiguity, contradictions, and missing edge cases
4. Evaluate technical feasibility of the proposed design
5. Assess document quality (clarity, format, completeness)
6. Produce a verdict: APPROVED, APPROVED WITH COMMENTS, or REVISION REQUIRED

## Review Checklist
- [ ] Every requirement ID is traceable in the spec
- [ ] No requirement is missing from the spec
- [ ] Acceptance criteria are testable and unambiguous
- [ ] Data models are consistent across sections
- [ ] Edge cases are addressed
- [ ] Non-functional requirements are specified
- [ ] No contradictions between sections

## Output Format

```
# Specification Review Report
**Spec Reviewed:** [filename]
**Verdict:** APPROVED / APPROVED WITH COMMENTS / REVISION REQUIRED

## Coverage Analysis
| Requirement ID | Covered? | Notes |
|---------------|----------|-------|

## Issues Found
### Critical (Must Fix)
1. [Issue]: [Location in spec] — [Why it's a problem]

### Major (Should Fix)
2. [Issue]: [Location in spec] — [Recommendation]

### Minor (Nice to Fix)
3. [Issue]: [Location in spec] — [Suggestion]

## Strengths
- [What the spec does well]

## Summary
[Overall assessment]
```

## Constraints
- DO NOT rewrite the spec — only identify issues
- DO NOT approve a spec that has missing requirement coverage
- ALWAYS be specific about where issues are (section numbers, requirement IDs)
```

### Step 3: (Optional) Add a Shared Instruction File

Create `.github/instructions/spec-standards.instructions.md`:

```markdown
---
description: "Use when writing or reviewing specifications, requirements documents, or technical analyses"
applyTo: "specs/**"
---

# Specification Document Standards

- Use ISO 8601 dates (YYYY-MM-DD)
- Requirement IDs follow the format REQ-NNN (zero-padded to 3 digits)
- All specs must include a version number and status
- Status values: Draft, In Review, Approved, Superseded
- Use tables for structured data (traceability matrices, comparison tables)
- Code examples use fenced code blocks with language identifiers
```

---

## Part 4: How to Use It

### First Run

1. **Open your project folder** in VS Code
2. **Place your raw business requirements** in `requirements/` as a markdown file (e.g., `requirements/project-x.md`)
3. **Open Copilot Chat** and select the **Product Delivery** agent from the agents dropdown
4. **Type your request**:

   > Take the business requirements in requirements/project-x.md and produce a complete functional specification.

5. **Watch the workflow**:
   - The coordinator launches Business Analyst and Solution Architect as parallel subagents
   - Each appears as a collapsible tool call in chat — click to expand and see their work
   - The coordinator then launches the Spec Writer with combined inputs
   - Finally the Spec Reviewer checks the output
   - If revision is needed, the coordinator iterates

### Using Handoffs (Alternative Sequential Approach)

If you prefer manual control over each step, you can add handoffs to make the coordinator pass control to you between phases. Add to the coordinator's frontmatter:

```yaml
handoffs:
  - label: "Review Requirements"
    agent: product-delivery
    prompt: "Requirements are structured. Review requirements/ and proceed to spec writing when ready."
    send: false
  - label: "Review Spec"
    agent: product-delivery
    prompt: "Spec is written. Run the reviewer to check quality."
    send: false
```

When users see the handoff button and select it, they switch to the target agent with the prompt pre-filled. If `send: true`, the prompt automatically submits to start the next workflow step.

### How Subagent Invocation Works

Subagents are typically agent-initiated, not directly invoked by users in chat. The pattern works like this:

1. You (or your custom agent's instructions) describe a complex task
2. The main agent recognizes the part of the task that benefits from isolated context
3. The agent starts a subagent, passing only the relevant subtask
4. The subagent works autonomously and returns a summary
5. The main agent incorporates the result and continues

You can hint that you want subagent delegation by phrasing your prompt to suggest isolated research or parallel analysis. The main agent will start a subagent, pass the task to it, and receive only the final result.

> **Tip**: For consistent subagent behavior, define when to use subagents in your custom agent's instructions rather than prompting for them manually each time.

---

## Part 5: Key Design Principles

### Why This Architecture Works

| Principle | How It's Applied |
|-----------|-----------------|
| **Context isolation** | Each subagent gets a clean context — the BA isn't distracted by architecture concerns |
| **Least privilege tools** | BA and Architect are read-only; only the Spec Writer can edit files |
| **Parallelization** | BA + Architect run simultaneously in Phase 1 |
| **Iterative quality** | Spec Writer ↔ Reviewer loop continues until quality passes |
| **PROSE compliance** | Every agent defines Purpose, Role, Output format, Scope boundaries, and Evaluation criteria |
| **Traceability** | Requirement IDs flow from BA → Writer → Reviewer, ensuring nothing is lost |

### Common Pitfalls to Avoid

1. **Don't make the coordinator too smart** — it should delegate, not do the work itself
2. **Don't give every agent all tools** — a reviewer with `edit` tools might "fix" the spec instead of reporting issues
3. **Keep descriptions keyword-rich** — the coordinator finds subagents by matching their `description` field against the task
4. **Don't skip `user-invocable: false`** on worker agents — you don't want 5 internal agents cluttering your agent picker
5. **Don't embed huge instructions** — keep each agent file focused; use `.instructions.md` for shared standards

### Anti-Patterns

| Anti-Pattern | Why It's Bad | Fix |
|-------------|--------------|-----|
| **Swiss-army agents** | Too many tools, tries to do everything | One role, minimal tools per agent |
| **Vague descriptions** | "A helpful agent" doesn't guide delegation | Use "Use when..." pattern with specific keywords |
| **Role confusion** | Description doesn't match body persona | Ensure description and body are aligned |
| **Circular handoffs** | A → B → A without progress criteria | Define clear convergence conditions |
| **Kitchen-sink instructions** | Everything instead of what matters most | Only what's relevant to every task |

### Agent File Best Practices

- **Single role**: One persona with focused responsibilities per agent
- **Minimal tools**: Only include what the role needs — excess tools dilute focus
- **Clear boundaries**: Define what the agent should NOT do
- **Keyword-rich description**: Include trigger words so parent agents know when to delegate
- **Show, don't tell**: Brief code examples over lengthy explanations in instructions

---

## Part 6: Extending the Family

Once you're comfortable, you can add more specialists:

| Agent | Purpose | When to Add |
|-------|---------|-------------|
| **UX Analyst** | Generate wireframe descriptions and user flow diagrams | When specs need UI details |
| **Data Modeler** | Design database schemas and entity relationships | When specs have significant data modeling |
| **Risk Analyst** | Identify and categorize project risks | For high-stakes projects |
| **Test Planner** | Generate test plans from acceptance criteria | To extend the pipeline into QA |

Each new agent follows the same pattern:

1. Define the PROSE dimensions (Purpose, Role, Output, Scope, Evaluation)
2. Restrict tools to the minimum needed
3. Set `user-invocable: false`
4. Add it to the coordinator's `agents` list

### Example: Adding a New Worker Agent

```markdown
---
description: "Use when generating test plans from acceptance criteria. Specialist in QA strategy and test case design."
tools: ['read', 'search', 'edit']
user-invocable: false
---

You are a Test Planner. Your job is to...
```

Then update the coordinator's frontmatter:

```yaml
agents: ['business-analyst', 'solution-architect', 'spec-writer', 'spec-reviewer', 'test-planner']
```

---

## Quick-Start Checklist

- [ ] Create `.github/agents/` in your workspace
- [ ] Create the 5 agent files (coordinator + 4 workers)
- [ ] Create `.github/copilot-instructions.md` with project conventions
- [ ] Create `requirements/`, `specs/`, and `reviews/` directories
- [ ] Put a raw requirements document in `requirements/`
- [ ] Select the **Product Delivery** agent in Copilot Chat
- [ ] Ask it to process your requirements
- [ ] Expand the subagent tool calls to watch each specialist work

---

## Reference Links

- [VS Code Custom Agents Documentation](https://code.visualstudio.com/docs/copilot/customization/custom-agents)
- [VS Code Subagents Documentation](https://code.visualstudio.com/docs/copilot/agents/subagents)
- [VS Code Custom Instructions Documentation](https://code.visualstudio.com/docs/copilot/customization/custom-instructions)
- [VS Code Agent Skills Documentation](https://code.visualstudio.com/docs/copilot/customization/agent-skills)
- [VS Code Prompt Files Documentation](https://code.visualstudio.com/docs/copilot/customization/prompt-files)
- [Awesome GitHub Copilot — Community Resources](https://github.com/github/awesome-copilot)
