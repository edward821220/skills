---
name: refine-notion-feature
description: Review a Notion feature with its primary spec, collect PM clarifications asynchronously, and publish approved tracer-bullet engineering tickets.
---

# Refine Notion Feature

Turn one PM feature and its primary spec into approved, published engineering tickets. End when clarification is waiting on the PM or when the tickets are published; implementation belongs to the assigned engineers.

## Inputs

Require a Notion feature-ticket URL. Treat the feature, its linked primary spec, its explicitly designated UI/UX spec when present, and explicit PM replies on the feature as the product source of truth. Use the current user as the engineering manager who approves the breakdown.

## Notion writing gate

Before every Notion write, read the installed [speak-human-tw skill](../speak-human-tw/SKILL.md) completely and apply it in embedded mode to all author-written prose. This includes clarification comments, task titles and bodies, blocker fallbacks, and any other text sent to Notion. Run its draft, audit, and final pass internally; send only the final text to Notion.

Preserve required templates and section labels, `OQ-###` identifiers, code identifiers, API paths, schema names, enum values, product names, links, and exact Notion property values. Refine the prose around them using `speak-human-tw` (Taiwan Traditional Chinese, full-width punctuation, natural phrasing, no AI patterns or Chinese mainland vocabulary) without changing facts, decisions, requirements, or acceptance criteria. Before writing, confirm that the final text contains no em dash or en dash, invented detail, AI attribution, chatbot filler, or promotional language.

## Process

### 1. Load the source

Fetch the feature with discussions and all of its comments. From the feature, fetch exactly the complete primary spec and any explicitly designated UI/UX spec. Do not fetch other linked Notion pages or tickets, including historical tasks, related features, dependency tickets, prior implementations, or links found inside the allowed specs. Treat this Notion source set as a whitelist for the entire run.

Allow database/data-source schema reads and related-task existence queries only as operational metadata needed to publish safely; do not treat their task rows as requirement context.

Read prior refinement comments before generating new questions. Explore the current codebase—source code, tests, migrations and schema, configuration, generated contracts, and repository documentation—to settle facts about implemented behavior, feasibility, redundancy, technical dependencies, and ticket boundaries.

If the primary spec is missing or ambiguous, treat identification of the authoritative spec as a product clarification.

**Complete when:** the feature and its comments, the primary spec, the designated UI/UX spec when present, and every prior PM answer are loaded; every existing `OQ-###` has a known status; and no other Notion ticket or page has been fetched.

### 2. Audit the feature

Read [the clarification audit](references/clarification-audit.md) completely and apply every dimension. Resolve repository facts through legwork. Route product decisions to the PM. Carry engineering decisions into the relevant ticket for its assignee without deciding architecture in this refinement.

**Complete when:** every discovered gap is classified as a resolved fact, an open product decision, or an engineering consideration, and every requirement has a testable product outcome.

### 3. Run the asynchronous PM gate

When product decisions remain, read [the clarification comment template](references/clarification-comment.md) completely. Assign stable `OQ-###` identifiers across the whole feature, sort the currently visible questions by identifier, and split them into consecutive batches of at most three questions. Post one Notion comment per batch. Every comment must contain only one `Questions for PM` section; do not add batch labels, AI attribution, established facts, audit summaries, repository findings, next-step prose, or other sections.

Before posting, build one global OQ registry from every prior refinement comment and PM reply on the feature. Never recycle or duplicate an identifier across batches. Post batches in ascending identifier order. If a comment create partially succeeds or returns an ambiguous failure, reload all feature comments, compare the persisted identifiers with the intended batch, and post only the missing questions; never retry an already-persisted batch blindly.

Stop after all currently visible batches have been posted. Create no tickets while any product decision remains open.

On a later invocation, reload every feature comment, merge questions and PM replies into the global OQ registry, map replies to the existing identifiers, and preserve answered questions. When following up on an ambiguous answer, use the discussion containing that identifier when possible. Batch only newly surfaced or still-ambiguous questions, again with at most three questions per comment. A PM answer that materially changes the primary spec remains open until the authoritative spec records the decision or the PM explicitly designates the reply as authoritative.

**Complete when:** zero product decisions remain open and the authoritative product sources contain testable outcomes for every in-scope requirement.

### 4. Draft tracer-bullet tickets

Draft each ticket as a vertical slice using the slicing semantics below. Tickets live only in Notion, and the frontier communicates what is immediately takeable; do not produce local plan artifacts or separate checkpoint tickets. Do not record specific file paths in ticket bodies, because file paths go stale between refinement and implementation. Read [the Notion ticket template](references/ticket-template.md) completely and follow its fields for every ticket.

Keep the Step 1 Notion whitelist while drafting tickets; derive implementation facts and blocking edges from the allowed product sources plus the current codebase, not from linked historical or dependency tickets.

Write every task title, heading, description, estimate, and verification statement in Taiwan Traditional Chinese (`zh-TW`). Preserve code identifiers, API paths, schema names, enum values, and product names in their source language.

Every breakdown must preserve these slicing semantics:

- Cut narrow, complete vertical slices through every necessary layer.
- Make each slice independently demoable or verifiable in one fresh implementation context.
- Declare genuine blocking edges and expose the frontier.
- Order tickets so every blocking edge lands before its dependents, and each landed ticket leaves the system in a working state.
- Separate sequential dependencies (database migrations, shared state changes, shared API contracts) into prerequisite tickets; independent slices can be parallelized once prerequisites land.
- Schedule high-risk or unknown-heavy slices early so integration surprises surface before the bulk of the effort.
- Aim for a verification checkpoint every two to three tickets — an ordering signal for the reviewer, not a published ticket.
- Use expand–contract only for a wide mechanical refactor that cannot land green as vertical slices.
- Present the numbered draft and iterate on granularity, edges, merges, and splits until the manager approves it.

Grade every draft ticket against this sizing table before presenting it:

| Size | Approx. files | Scope | Example |
|------|---------------|-------|---------|
| **XS** | 1 | Single function or config change | Add a validation rule |
| **S** | 1–2 | One component or endpoint | Add a new API endpoint |
| **M** | 3–5 | One feature slice | User registration flow |
| **L** | 5–8 | Multi-component feature | Search with filtering and pagination |
| **XL** | 8+ | Too large — split it further | — |

Target S- and M-sized tickets; split anything graded L or larger before the manager sees the draft. Split a ticket further if it touches two or more independent subsystems (e.g. auth and billing) or cannot be verified in a single focused implementation context. A ticket whose title needs an "and" is usually two tickets. The grade is a drafting heuristic only — publish no size field on the ticket; `理想工程工時` remains the sole recorded estimate.

Assign the responsible engineer only through Notion's owner property when known; keep ownership out of the ticket body. Preserve the vertical slice regardless of the assignee's specialization.

Reference the applicable feature-level acceptance-criteria identifiers instead of copying their business prose. Do not define prescriptive task-level acceptance criteria or completion evidence, leaving technical design and verification strategy to the responsible engineer. Add `任務邊界` only when adjacent ticket ownership would otherwise be ambiguous.

Fill one initial `理想工程工時` estimate for every ticket during refinement. Express every estimate as `N 人日`, using `1 人日 = 8 小時`; half-day values such as `2.5 人日` are allowed, but hour-based estimates are not. Treat it as uninterrupted engineering effort rather than a delivery commitment; if evaluation or implementation shows the estimate is unlikely to hold, the implementer updates it dynamically through a task comment or direct message to the manager.

Append the blank `Tech Spec（工程師填寫）` template from the ticket template to the bottom of every task. Leave it unfilled during refinement; before implementation, the assigned engineer may use the template or document the technical plan in their preferred format.

**Complete when:** the manager explicitly approves every ticket boundary and blocking edge, every source requirement and feature-level acceptance criterion maps to at least one ticket, and each ticket has an initial ideal-effort estimate in person-days, linked feature-level acceptance criteria references, Prototype URLs copied from the feature 影響頁面 when that table lists them, and the blank engineer-owned Tech Spec template.

### 5. Publish to Notion

Create tickets in dependency order, blockers first, so every blocking edge references a real task identifier. Fetch the feature database and engineering task database schemas, identify the exact writable relation from a task to its feature, and create every ticket in the task database with that relation set to the feature ticket. Confirm the reciprocal feature-to-task relation exposes the created tickets. Use this database relation for publication even when a native parent/sub-item property also exists. If the schemas cannot express a writable task-to-feature relation, pause before publishing and report the limitation.

Use exact existing property names and values; set status, assignee, estimate, and blockers only where the schema supports them. Use native blocking relationships when available and the body fallback otherwise.

Before retrying a failed create, query for already-created tasks related to the feature so retries cannot duplicate tickets. Preserve the feature and primary spec; add relations and comments without rewriting their product content.

Return the created ticket links, their blocking graph, and the initial frontier. Stop there.

**Complete when:** every approved ticket exists once in the task database, is related to the feature ticket through the configured task-to-feature relation, declares its blockers, includes the blank Tech Spec template at the bottom, appears through the reciprocal feature-to-task relation, and the manager can see which tickets are immediately takeable.

## Failure boundary

If Notion access, the primary spec, the task database, the task-to-feature relation mapping, or another required schema mapping is unavailable, report the exact missing input and pause at the current step. Preserve all resolved `OQ-###` identifiers on resumption.
