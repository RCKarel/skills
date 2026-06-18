---
name: complexity-analyzer
description: Analyze implementation tickets in GitHub, Jira, Linear, GitLab, markdown issue files, or another tracker to decide whether each ticket is safely scoped for a developer or coding agent. Use after issues are generated or imported and before assignment, especially when tickets span multiple implementation areas, have broad acceptance criteria, unclear ownership, unresolved decisions, risky migrations, large test surfaces, dependency graph problems, or prior failed implementation attempts. Decompose oversized tickets, update dependency plans, and rerun hydrate-issues on affected tickets without expanding product scope.
---

# Complexity Analyzer

## Purpose

Analyze implementation tickets and decide whether each is appropriately scoped for implementation.

If a ticket is too complex, broad, ambiguous, risky, or overloaded, produce a cleaner implementation-ready ticket set. Decomposition may narrow the original ticket, convert it to a parent/coordinator, archive and replace it, or mark it `Not ready`.

After decomposition, rerun `$hydrate-issues` on the affected ticket set so old, edited, and new tickets have enough context for a fresh implementation context.

Do not expand product scope.

## Inputs

Use as many as are available:

* Issue list from GitHub, Jira, Linear, GitLab, markdown files, or another tracker
* Issue titles, descriptions, acceptance criteria, labels, status, priority, assignee, milestone, sprint, epic, parent, dependencies, and comments
* PRD, `CONTEXT.md`, architecture docs, API docs, schema docs, design notes, and decision notes
* Repository evidence and implementation context
* Permission model for whether tickets may be created, edited, linked, closed, or archived
* Existing `$hydrate-issues` skill

Do not assume unavailable inputs exist. If tracker write access is not explicitly granted, operate in dry-run mode and return a proposed decomposition plan only.

## Core Rules

Classify each ticket as:

* Small enough to implement as-is
* Complex but still implementable as one ticket
* Too complex and should be decomposed
* Not ready because complexity cannot be resolved without a decision

Decompose only when the work contains separable implementation responsibilities, risks, states, dependencies, or review boundaries. Do not decompose merely because a ticket is long.

Golden rule: a decomposition is only good when the resulting tickets are easier to implement, easier to review, and safer to test. Optimize for lower implementation risk, not more tickets.

Do not invent requirements, add nice-to-have work, create new product scope, silently remove acceptance criteria, blindly copy dependencies, or archive/close an original ticket before replacements are created and linked.

## Complexity Rubric

Score each dimension from `0` to `3`.

| Dimension | 0 | 1 | 2 | 3 |
| --- | --- | --- | --- | --- |
| Scope breadth | One narrow behaviour | Few related behaviours | Multiple behaviours in one flow | Multiple flows, actors, or responsibilities |
| Implementation surface | One file/module/service | Small related cluster | Multiple layers | Multiple systems, jobs, schemas, or integrations |
| Requirement clarity | Explicit and testable | Minor ambiguity | Several assumptions needed | Key behaviour or contract unclear |
| Dependency complexity | None | One clear prerequisite | Several sequencing concerns | External, cross-team, or unresolved prerequisite |
| Test complexity | Simple unit coverage | Unit plus small integration | Multiple integration/regression/manual paths | Broad e2e, migration, compatibility, or failure-mode testing |
| Data, migration, or contract risk | None | Small internal mapping | Schema, API, event, or persisted state | Migration, backfill, compatibility, or external contract |
| Rollout and regression risk | Isolated | Limited blast radius | Shared component/API or important flow | Critical path, production job, money, permissions, security, or irreversible action |

Bands:

* `0-5`: Low complexity. Keep as one ticket.
* `6-10`: Medium complexity. Keep unless there is a clean decomposition.
* `11-15`: High complexity. Decomposition recommended.
* `16+`: Very high complexity. Decomposition required unless explicitly rejected.

Override the score when there is an obvious atomicity problem, such as "build the new billing flow".

## Hard Triggers

Decompose, or mark `Not ready`, when a ticket includes:

* Multiple independently deliverable behaviours
* Multiple unrelated implementation areas
* Discovery, design, implementation, and migration in one ticket
* Backend contract work plus complex frontend behaviour
* Data model changes plus business logic plus reporting
* Happy path plus complex operational failure handling
* Acceptance criteria that cannot be verified in one coherent review
* Different blockers for different parts of the ticket
* Parts that should be implemented by different specialists or agents
* Work too large for a coding agent to complete with high confidence in one context

Do not decompose when:

* Work is tightly coupled and cannot be verified independently
* Replacement tickets would be artificial or too small to test/review
* Decomposition would obscure user value
* Decomposition requires inventing requirements
* The ticket is long only because it is well documented
* Complexity is only normal repository exploration

## Original Ticket Treatments

Choose one:

* `Keep unchanged`: Already appropriately scoped.
* `Hydrate only`: Scope is fine but context is thin.
* `Narrow original`: Keep one obvious core slice and move remaining scope into new tickets.
* `Convert to parent/coordinator`: Original is an umbrella, epic, or larger feature and should stop being an implementation ticket.
* `Archive and replace`: Original is broad, messy, duplicated, misleading, or would remain confusing after edits.
* `Mark Not ready`: A blocking decision prevents safe decomposition.

Prefer `Archive and replace` when the title no longer describes the work, the description is misleading, acceptance criteria cannot be cleanly edited, keeping the original would confuse implementers, the tracker lacks good parent-child decomposition, or clean replacements are simpler.

Archived or superseded tickets must keep an audit trail:

`This ticket has been superseded by <new ticket links>. It was decomposed because the original scope was too complex to implement and review safely as one issue. No product scope was intentionally added. See replacement tickets for the implementation-ready work.`

## Decomposition Patterns

Choose the cleanest boundary:

* By implementation layer: schema/migration, backend API/service, frontend integration, observability/ops
* By user flow stage: draft, review, submit, failed submission
* By system boundary: internal model, external provider adapter, webhook handling, reconciliation job
* By dependency sequence: database support, API contract, UI consumer, reporting/audit
* By risk: read-only display, write path, migration/backfill, permission enforcement
* By known versus unknown: ready implementation work plus a decision ticket for a real blocker

Only create discovery tickets when implementation is genuinely blocked by missing information.

## Replacement Ticket Rules

Each replacement ticket must:

* Preserve traceability to the original ticket
* Own one clear implementation responsibility
* Be independently understandable, implementable, reviewable, and testable
* Have clear acceptance criteria, explicit non-goals, and sibling dependencies where relevant
* Avoid duplicated acceptance criteria owned by another ticket
* Avoid product scope not present in the original ticket or source material

Each replacement ticket should include enough structure for `$hydrate-issues`:

* Superseded or parent ticket link
* Source references
* Context capsule
* Current behaviour
* Target behaviour
* Dependencies
* Acceptance criteria
* Test plan
* Non-goals
* Open questions

## Source Preservation

For every requirement moved from an original ticket to a replacement ticket, track:

* Original ticket ID
* Original requirement or acceptance criterion
* Supporting PRD, `CONTEXT.md`, docs, repository, issue, or conversation reference
* Destination replacement ticket
* Whether the requirement was preserved, narrowed, or removed as unsupported

Do not silently remove requirements. If a requirement is unsupported, do not carry it forward as verified scope. Record it as unsupported, add an open question, and mark the relevant ticket `Not ready` if it materially affects implementation.

## Dependency Graph

Review old dependency links and transfer them deliberately.

Incoming dependencies are tickets the original depended on:

* Transfer each dependency only to replacement tickets that genuinely need it.
* If only the first slice needs the dependency, attach it only there.
* If unclear, add an open question instead of guessing.

Outgoing dependencies are tickets that depended on the original:

* Transfer each downstream dependency to the replacement ticket that delivers the required capability.
* If several replacements are required, link all required tickets.
* If unclear, keep the downstream ticket linked to the archived original and add a dependency-review note.

Related links should be preserved only where still meaningful: relevant replacement, coordinator ticket, or archived original for audit.

When using `Archive and replace`, add supersession links:

* Original ticket `is superseded by` replacement tickets
* Replacement tickets `supersede` the original
* Replacement tickets link to each other with `depends on` or `blocks` where sequencing matters

If dependency transfer cannot be determined safely, include:

`Dependency review required: some links from the original ticket could not be safely mapped to replacement tickets without additional context.`

## Write Policy

Default to dry-run mode unless the user explicitly asks to update the tracker.

Dry-run mode returns analysis, proposed decomposition, proposed original-ticket edits, proposed replacement tickets, proposed dependency graph changes, and a hydration plan. Do not create, edit, close, archive, link, label, or comment on tickets.

Apply mode is allowed only with explicit permission and available write access. It may update descriptions, create replacements, convert originals to parent/coordinator tickets, archive/close/supersede originals, link tickets, add dependency links, add labels, add comments, rerun `$hydrate-issues`, and update affected descriptions with hydrated content.

Do not change status, assignee, sprint, estimate, or priority unless explicitly instructed. When archiving or closing, use the tracker's convention; if none exists, add a clear `Superseded` label or comment if supported.

## Subagents

For large issue sets, PRDs, docs, or repositories, use subagents when available. Useful roles:

* Issue reviewer: scope, acceptance criteria, overlap, atomicity
* PRD reviewer: product requirements, non-goals, terminology, edge cases
* Context reviewer: `CONTEXT.md`, architecture, schemas, APIs, patterns
* Codebase reviewer: relevant files, modules, tests, configs, jobs, current behaviour
* Dependency reviewer: links and safe dependency transfers
* Test reviewer: unit, integration, regression, manual verification coverage

Each subagent returns findings, source references, complexity drivers, decomposition boundaries, unverified assumptions, blocking questions, and dependency implications.

The main agent remains responsible for final judgement. Do not use subagents for small issue sets. Do not let subagents expand scope, create tickets, archive tickets, or make final readiness decisions.

## Process

1. Load tickets: ID, title, description, acceptance criteria, labels, status, parent/epic, dependencies, comments, estimates, and hydration state.
2. Load source context: PRD, `CONTEXT.md`, docs, decisions, repository evidence. Use sources only to understand scope and boundaries.
3. Build a requirement-to-ticket map. Report uncovered, duplicated, unsupported, misplaced, or hidden requirements. Do not silently attach uncovered requirements to the nearest ticket.
4. Score each ticket with the rubric. Identify complexity drivers, atomicity, blockers, and whether to keep, hydrate, decompose, or mark not ready.
5. Classify each ticket as `Keep as-is`, `Keep but hydrate`, `Edit scope and hydrate`, `Decompose recommended`, `Decompose required`, `Archive and replace recommended`, `Archive and replace required`, or `Not ready`.
6. For decompositions, propose original ticket treatment, replacement titles and owned scopes, moved acceptance criteria, dependencies, non-goals, blocking questions, source references, and dependency graph changes.
7. Check quality: no source-backed requirement lost, no new requirement invented, no overlap, replacements independently testable, dependencies explicit, original no longer duplicates replacement scope, complexity meaningfully reduced, audit trail preserved.
8. Apply only if permitted: update original, create replacements, link, add supersession links, add dependency links, add a short decomposition comment, preserve labels/milestone/epic where appropriate, and avoid changing priority/status/sprint/estimate/assignee unless instructed.
9. Rerun `$hydrate-issues` on affected tickets only: original edited ticket, replacements, and directly linked tickets whose dependency or scope changed.
10. Final consistency pass: no duplicate scope, dependencies clear, acceptance criteria testable, source references preserved, readiness accurate, blockers visible, scope preserved, changed tickets included in hydration target set, no unrelated backlog changes.

## Output Format

Start with:

```markdown
# Complexity analysis summary

* Total tickets analyzed: <number>
* Keep as-is: <number>
* Keep but hydrate: <number>
* Edit scope and hydrate: <number>
* Decompose recommended: <number>
* Decompose required: <number>
* Archive and replace recommended: <number>
* Archive and replace required: <number>
* Not ready: <number>
```

For each ticket:

```markdown
# Ticket: <ticket id/title>

## Current classification

<classification>

## Complexity score

<score>/21

| Dimension | Score | Reason |
| --- | ---: | --- |
| Scope breadth | <0-3> | <reason> |
| Implementation surface | <0-3> | <reason> |
| Requirement clarity | <0-3> | <reason> |
| Dependency complexity | <0-3> | <reason> |
| Test complexity | <0-3> | <reason> |
| Data, migration, or contract risk | <0-3> | <reason> |
| Rollout and regression risk | <0-3> | <reason> |

## Complexity drivers

* <driver>

## Recommendation

<keep/edit/decompose/archive/block recommendation>

## Proposed original ticket treatment

<Keep unchanged | Hydrate only | Narrow original | Convert to parent/coordinator | Archive and replace | Mark Not ready>

## Proposed replacement tickets

Only include when decomposition is recommended or required.

### Replacement ticket 1: <title>

**Owned scope**

* <scope>

**Moved from original ticket**

* <original requirement or acceptance criterion>

**Source references**

* <source reference>

**Dependencies**

* <dependency or None>

**Blocks**

* <blocked ticket or None>

**Non-goals**

* <non-goal>

**Readiness**

Ready or Not ready

**Open questions**

* [Blocking] <question>
* [Non-blocking] <question>

## Dependency graph changes

### Transfer from original ticket

* <dependency> -> <replacement ticket> because <reason>

### Transfer to downstream tickets

* <downstream ticket> should depend on <replacement ticket> because <reason>

### Supersession links

* <original ticket> is superseded by <replacement ticket>
* <replacement ticket> supersedes <original ticket>

### Requires manual review

* <dependency or link> - <reason>

If there are no dependency changes, write: None identified.

## Scope preservation check

* Requirements preserved: <yes/no>
* Requirements moved to replacement tickets: <list>
* Requirements removed as unsupported: <list or None>
* New requirements introduced: None

## Hydration action

Rerun /hydrate-issues on <affected tickets>.
```

If any decomposition is needed, include `## Final decomposition plan` with original ticket, treatment, replacement tickets, required links, dependency graph changes, and hydration target.

If apply mode made changes, include `## Final output after apply mode` with tickets updated, created, archived/superseded, links added, labels added, dependency changes made, tickets passed to `$hydrate-issues`, and tickets still blocked.

Always include:

```markdown
## Final summary

* Total tickets analyzed: <number>
* Tickets kept: <number>
* Tickets decomposed: <number>
* Tickets archived and replaced: <number>
* Replacement tickets proposed or created: <number>
* Tickets requiring hydration: <ids or count>
* Blocking questions: <short bullets or None>
* Dependency links requiring manual review: <short bullets or None>
* Requirements with no ticket coverage: <short bullets or None>
* Unsupported scope found: <short bullets or None>
```

## Final Checks

Before returning or applying changes, confirm:

* Every decomposition has a clear reason.
* Every replacement ticket owns distinct scope and is independently testable.
* The original ticket no longer competes with replacement tickets.
* Supersession links are explicit when archive-and-replace is used.
* Dependency graph changes are documented.
* Unsafe dependency transfers are marked for manual review.
* Every changed or new ticket is included in the `$hydrate-issues` target set.
* No unrelated backlog items were changed.
* No ticket was archived without an audit trail.
* No tracker write action was taken without explicit permission.
