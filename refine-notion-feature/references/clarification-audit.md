# Clarification Audit

Apply every dimension to the feature, primary spec, designated UI/UX spec, and PM replies on the feature. Record traceability by requirement heading or identifier instead of copying product prose.

## Classification

### Repository fact

Find the answer in the current codebase: source code, tests, migrations and schema, configuration, generated contracts, or repository documentation. Record the evidence for the manager; do not ask the PM and do not consult historical tickets.

Examples: an endpoint already exists, a schema supports a field, or tests demonstrate the current behavior.

### Product decision

Ask the PM when the answer changes externally observable behavior, business meaning, scope, priority, actor permissions, or acceptance. Include a recommended answer and its consequence.

Examples: what a terminal state means, who may retry, which error the user sees, whether historical records are in scope.

### Engineering consideration

Place the unresolved consideration in the ticket whose engineer owns the outcome. Keep product acceptance independent of the eventual implementation choice.

Examples: module boundary, persistence strategy, internal concurrency mechanism, test seam, whether a durable architectural trade-off deserves an ADR.

## Audit dimensions

- **Outcome and scope:** actor, goal, business value, in-scope cases, explicit exclusions.
- **Behavior:** happy path, alternate paths, empty states, invalid actions, cancellation and recovery.
- **State:** initial state, legal transitions, terminal states, retry semantics, repeated and concurrent actions.
- **Authorization:** actor permissions, separation of duties, visibility, disabled or ineligible actors.
- **Contracts:** inputs, outputs, validation, errors, compatibility, events, third-party behavior.
- **Data:** source of truth, history, migration, backfill, retention, audit evidence.
- **Experience:** UI states, copy that carries business meaning, accessibility-relevant behavior, operator workflows.
- **Failure:** timeout, partial success, dependency outage, reconciliation, user-visible recovery.
- **Operations:** rollout constraints, observability outcomes, support or manual intervention.
- **Acceptance:** measurable assertions for happy paths, boundaries, failures, permissions, and concurrency where relevant.
- **Dependencies:** prerequisite product decisions, external teams, provider readiness, and other features.

## Question quality gate

Every PM question must include:

1. A stable `OQ-###` identifier.
2. The requirement source and current interpretation.
3. One concrete scenario that exposes the gap.
4. One decision-shaped question.
5. A recommended answer.
6. The behavior, scope, estimate, or ticket boundary affected.

Split a question when it asks for more than one independent decision. Omit questions whose answers are repository facts or engineering choices.

## Ready-to-split gate

Pass only when:

- zero product decisions remain open;
- every in-scope behavior has a testable outcome;
- actors, permissions, state meanings, failure behavior, and scope boundaries are unambiguous where relevant;
- PM replies and the primary spec do not conflict;
- dependencies are identifiable well enough to draft blocking edges;
- engineering considerations can be owned inside tickets without changing product acceptance.
