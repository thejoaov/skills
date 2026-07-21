---
name: feature-management
description: Full lifecycle for planning and documenting a new feature. Use when starting a new feature, writing a PRD from scratch with an interview, creating a technical spike for unknowns, or generating an implementation task checklist. Covers the spike → PRD → tasks workflow with versioned, non-destructive documents under docs/[feature-name]/. Trigger when the user says "plan a feature", "let's start a new feature", "write a spike", "create tasks for this", or when entering Plan mode.
---

# Feature Management & Planning

This skill guides the full lifecycle of a new feature: research, requirements, and execution tracking. It produces versioned documents under `docs/[feature-name]/` that serve as the persistent record of every decision made.

> If you already have a conversation to capture and just need a PRD written quickly, use `to-prd` instead. This skill is for the full planning workflow including the interview.

## Document Structure

Every feature gets its own directory:

```text
docs/
  [feature-name]/
    spike.md      Optional — technical research, library evaluation, and feasibility
    prd.md        Required — product requirements and architecture impact
    tasks.md      Optional — implementation checklist
```

All files use YAML frontmatter with `created_at` and `last_modified_at` timestamps:

```yaml
---
created_at: "YYYY-MM-DD HH:MM:SS"
last_modified_at: "YYYY-MM-DD HH:MM:SS"
---
```

## Versioning Rules

- `spike.md` and `tasks.md` are **immutable once written** — do not modify them for new phases. Create a new directory (`[feature-name]-v2/`) instead.
- `prd.md` is a **living document**, but updates are non-destructive: wrap deprecated text in slashes (`/old text/`) and append to a `## Rectifications` section with the date and rationale. Always update `last_modified_at`.

## Dependency Philosophy (Build vs. Buy)

To keep the project lean, maintainable, and free of unnecessary dependency bloat, adhere to the following principles when planning features:

- **Default Stance (Zero-Dependency Bias):** Prefer manual/in-house implementation using existing project stack primitives, standard libraries, or native capabilities whenever feasible. Avoid adding third-party packages for low-to-medium complexity features to prevent bundle bloat, supply chain security risks, and breaking API churn.
- **When to Consider External Libraries:**
  1. **High Domain Complexity or Security-Critical Features:** Domains where custom implementation is notoriously error-prone, security-sensitive, or requires reinventing massive infrastructure wheels. Examples include:
     - Authentication, OAuth 2.0 / OIDC & Session Management
     - Fine-Grained Authorization (RBAC / ABAC / ReBAC)
     - In-App Purchases (IAP), Subscriptions & Payment Gateway Integrations
     - Cryptography, Key Management & Data Encryption
     - Complex State Machines, Schema Validation & Form Engine
     - Data Synchronization, CRDTs, Local-First & Offline Storage
     - Rich Text / WYSIWYG Editors & Complex PDF Generation / Parsing
     - Native Hardware Abstractions, Push Notifications & Background Job Queues
  2. **High-Quality / Battle-Tested Libraries:** An external package may be recommended if it scores exceptionally high across core quality metrics:
     - **Maintainability:** Active commits, frequent releases, clear breaking-change policies, rapid security patch response.
     - **Package Weight / Footprint:** Tree-shakable, minimal bundle size impact, zero or low transitive dependency tree.
     - **Documentation & DX:** Clear guides, first-class TypeScript support, clean API design.
     - **Community & Governance:** Widespread adoption (GitHub stars, monthly downloads), active issue triage, backed by reputable organizations or strong OSS maintainers.
- **Decision Hand-off to PRD:** The Spike investigates options, researches top libraries, and assesses manual build complexity. The Spike **must not unilaterally lock in a library** — it presents the research, metrics, and trade-offs so the PRD (and stakeholders) can make an informed decision on whether to use a library OR build in-house.

## Step 1 — Spike (optional)

Before writing requirements, ask: are there significant technical unknowns, API constraints, database migration risks, architectural questions, or library choices?

If yes, create `docs/[feature-name]/spike.md` first.

A spike is a time-boxed investigation. Its job is to produce concrete research, evaluate libraries vs. manual implementation, and provide recommendations that directly inform the PRD.

### Spike Research Protocol:

1. **Investigate Existing Capabilities:** Check if the codebase, current stack, or native APIs already have primitives to solve the problem without adding new packages.
2. **Library Research & Search:** If the domain is complex (Auth, RBAC, IAP, Crypto, Sync, etc.) or candidate libraries exist, search for top solutions and evaluate them using the core metrics (Maintainability, Weight, Docs/DX, Community).
3. **In-House / Manual Build Assessment:** Evaluate what it would take to build from scratch manually:
   - Estimate **Implementation Difficulty** (Low, Medium, High, Critical).
   - Identify technical risks, maintenance overhead, and security/edge-case pitfalls.
4. **Compare & Recommend:** Present a clear side-by-side comparison of manual build vs. top candidate libraries, leaving the final choice to be ratified in the PRD.

```markdown
---
created_at: "YYYY-MM-DD HH:MM:SS"
last_modified_at: "YYYY-MM-DD HH:MM:SS"
---

# Spike — [Feature Name]

## Goal

What question, technical unknown, or architecture choice is this spike trying to answer?

## Technical Findings

What was discovered? Include API constraints, schema considerations, third-party limitations, or native platform behavior.

## Library Research & Build vs. Buy Analysis

### In-House / Manual Build Assessment
- **Estimated Complexity:** [Low | Medium | High | Critical]
- **Implementation Overhead:** Summary of work required to build manually from scratch.
- **Risks & Edge Cases:** Potential pitfalls (e.g., security vulnerabilities, cross-platform inconsistencies, high maintenance burden).

### Library Evaluation Matrix
| Library | Maintainability | Bundle Weight | Docs & DX | Community & Adoption | Key Pros & Cons |
|---|---|---|---|---|---|
| `[lib-a]` | Active (vX.Y, updated 2d ago) | Lightweight (X kB, tree-shakable) | Excellent, TS native | High (Xk stars, Y downloads/mo) | **Pros:** ... <br> **Cons:** ... |
| `[lib-b]` | ... | ... | ... | ... | ... |

### Options & Recommendations for PRD
- **Option 1 (Manual Build):** Pros/cons and complexity implications.
- **Option 2 (Library X):** Pros/cons and architectural impact.
- **Recommendation:** Clear suggestion based on complexity vs. bloat trade-offs, leaving final decision to the PRD.

## Conclusion

Summary of findings and concrete input for the PRD.
```

## Step 2 — PRD (required)

Run an interview with the user before writing the PRD. Ask targeted questions about:

- Who the user is (actor) and what problem they have
- The happy path and the most important edge cases
- Auth and permission requirements
- Which packages and layers will be affected
- **Dependency / Library Decisions:** Based on spike findings or domain complexity, decide whether to use an external library or build in-house manually (and confirm the manual implementation difficulty if building in-house).
- Any hard constraints (performance, offline support, compatibility)

After the interview, write `docs/[feature-name]/prd.md` using the template:

```markdown
---
created_at: "YYYY-MM-DD HH:MM:SS"
last_modified_at: "YYYY-MM-DD HH:MM:SS"
---

# PRD — [Feature Name]

## 1. Overview & Objectives

Why are we building this? What problem does it solve and what is the success criteria?

## 2. Functional Requirements

- **FR-01:** [Detailed description]
- **FR-02:** [Detailed description]

## 3. Architecture, Package Impact & Dependencies

- **`packages/db`:** Schema additions or modifications
- **`packages/api`:** New procedures, routers, service methods
- **`packages/auth`:** Auth guards, roles, session requirements
- **`apps/web`:** UI components, routes, pages
- **`apps/native`:** Native screens, navigation
- **`packages/env`:** New environment variables needed

### Dependency & Library Decisions
- **Approach Chosen:** [Manual Implementation OR Library Name (e.g., `better-auth`, `reanimated`)]
- **Rationale:** Why this choice was made (e.g., "Manual implementation chosen to avoid bundle bloat; difficulty rated Medium; edge cases handled via custom service layer" OR "Adopted `[library]` due to high domain complexity in RBAC/OAuth and excellent maintainability/bundle-weight metrics").
- **Manual Build Difficulty & Edge Cases (if manual):** Estimated complexity [Low | Medium | High | Critical] and key risks to mitigate during implementation.

## 4. User Journeys

Step-by-step walk-through of the main flows (happy path + key alternatives).

## 5. Open Questions

- [Unresolved product decision]
- [Technical constraint requiring clarification]

## 6. Rectifications

- [No rectifications yet]
```

**Rules:**
- Be exhaustive on business logic — include happy paths, alternative flows, and edge cases
- Explicitly document dependency choices (manual vs. library) and justify them against bloat vs. complexity
- Do not include file paths or code snippets unless a specific type or schema shape was decided
- Do not include time estimates
- Always end with `## Open Questions` — never leave this section empty if there are unresolved items
- After writing, ask the user to resolve any open questions using structured multiple-choice where possible

## Step 3 — Tasks (optional)

Create `docs/[feature-name]/tasks.md` when the implementation needs to be tracked in detail — especially for features touching multiple packages or requiring coordination.

```markdown
---
created_at: "YYYY-MM-DD HH:MM:SS"
last_modified_at: "YYYY-MM-DD HH:MM:SS"
---

# Implementation Checklist — [Feature Name]

## Summary of Completion

0% complete — initial planning phase.

## Tasks

### 1. [Task Group] - (done when finished)

Brief description of what this group covers.

- (done) - [Completed subtask]
- [ ] [Pending subtask — include exact files, services, and methods affected]
```

**Rules:**
- Start with `## Summary of Completion` — update it as work progresses
- Group tasks by package or domain boundary (`packages/db`, `packages/api`, `apps/web`, etc.)
- Each subtask must name the exact files, service classes, routers, or methods involved — enough detail for another developer or agent to implement with no ambiguity
- Mark completed main tasks with ` - (done)` appended to the heading
- Mark completed subtasks with `(done) - ` prepended to the list item
- No time estimates
