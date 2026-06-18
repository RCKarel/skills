---
name: hydrate-issues
description: Review generated implementation issues against a PRD, CONTEXT.md, the current repository, and relevant project docs, then enrich each issue with source-backed implementation context, readiness, dependencies, acceptance criteria, tests, non-goals, and open questions. Use when issues generated from a PRD, planning conversation, feature brief, or implementation breakdown are too thin, lack source references, assume prior context, have vague acceptance criteria, or may have drifted from the source requirements.
---

# Hydrate Issues

## Purpose

Use this skill to review generated implementation issues against the PRD, `CONTEXT.md`, the current codebase, and any relevant project documentation, then enrich each issue so that a developer or coding agent can implement it from a fresh context window.

The goal is to add missing implementation context, not to expand the product scope.

This skill may identify:

* Missing requirement coverage
* Unsupported scope in an issue
* Conflicts between sources
* Overlap between issues
* Missing dependencies
* Blocking implementation decisions

It must not silently fix these problems by expanding, splitting, merging, or creating issues unless explicitly instructed to do so.

## Inputs required

Use as many of the following as are available:

* The generated issues, including their existing descriptions and acceptance criteria
* Existing issue metadata, such as IDs, labels, milestones, dependencies, and parent issues
* The PRD
* `CONTEXT.md`
* The current repository or relevant source files
* Relevant docs, architecture notes, API docs, schema docs, design notes, and operational runbooks
* The original conversation or planning notes, where available
* Related existing or completed issues
* The branch, commit, release, or document version being reviewed, where relevant

Do not assume unavailable inputs exist.

When a required fact cannot be verified because a source is unavailable, state that explicitly. Mark the issue `Not ready` only when the missing information blocks implementation.

## Core instruction

Review the generated issues against the PRD, `CONTEXT.md`, the current codebase, and relevant docs.

For each issue, enrich it so that a developer or coding agent can implement it from a fresh context window without needing to re-read the full conversation.

Do not increase scope.

Do not invent requirements.

Only add clarifying context supported by:

* The PRD
* `CONTEXT.md`
* Relevant project documentation
* Repository evidence
* Existing related issues
* Explicit decisions from the original conversation

If something is unclear, unsupported, or contradictory, capture it as an open question instead of guessing.

## Source discipline

### Distinguish intended and current behaviour

Use product and design sources to establish intended behaviour.

Use repository evidence, deployed-system evidence, or explicit current-state documentation to establish current behaviour.

Do not describe a proposed design as current behaviour merely because it appears in the PRD.

Where useful, distinguish between:

* `Observed in code`
* `Documented behaviour`
* `Inferred but not verified`
* `Unknown`

Do not present an inference as a verified fact.

### Use specific source references

Prefer precise references such as:

* PRD: `Permissions / Role behaviour`
* `CONTEXT.md`: `NotificationService`
* `docs/api/webhooks.md`: `Retry policy`
* `src/billing/subscriptions.ts`: `cancelSubscription`
* Schema: `subscriptions.status`
* Related issue: `#123`
* Conversation decision: `Use the existing account-level feature flag rather than adding a user setting`

Include a commit, branch, document version, or decision date when it materially affects the interpretation.

Avoid references such as `the PRD` or `the codebase` without identifying the relevant section or component.

### Handle conflicting sources explicitly

When sources conflict:

1. Do not silently choose the interpretation that seems most likely.
2. Cite each conflicting source.
3. Determine whether a clearly documented later decision supersedes an earlier source.
4. State the resolution only when the override is explicit.
5. Otherwise, add a blocking open question and mark the issue `Not ready` if implementation depends on the answer.

Repository behaviour does not automatically override product requirements. It may represent the behaviour that must be changed.

### Handle unsupported issue content

If the original issue contains requirements that cannot be traced to a source:

* Do not convert them into verified target behaviour.
* Do not silently remove them.
* Identify them as unsupported or ambiguous.
* Add an open question requesting confirmation.
* Mark the issue `Not ready` when the unsupported requirement materially changes implementation.

## Issue integrity

Preserve the following unless explicitly instructed otherwise:

* Issue order
* Issue IDs
* Existing titles
* Labels
* Milestones
* Parent-child relationships
* Existing dependencies
* Existing valid constraints
* Existing valid acceptance criteria

Do not:

* Merge issues
* Split issues
* Create replacement issues
* Move scope from one issue to another
* Delete unsupported scope without noting it
* Assign ownership or estimates without evidence

When a title has a clear typo or is too vague to identify the work, it may be improved. Preserve the original title as:

`Original title: <original title>`

## Process

### 0. Find the issue destination

Before hydrating issues, determine where the issues live and how they can be updated.

Prefer an existing issue tracker integration over loose markdown output:

1. Look for configured MCP tools or resources for GitHub, GitLab, Jira, Linear, or the project's issue tracker.
2. Look for installed and authenticated CLIs such as `gh`, `glab`, or `jira`.
3. Use repository evidence to identify the tracker, such as git remotes, `.github/`, `.gitlab/`, Jira project keys in docs, or issue links in planning files.
4. If a tracker integration is available and the user asked to update issues, update the existing issues in place.
5. If no tracker integration is available, look for markdown issue files in the repository, such as `issues/`, `docs/issues/`, `tickets/`, or planning docs, and update those files when they clearly correspond to the supplied issues.
6. If neither a tracker integration nor matching markdown files are available, return the hydrated issues as markdown.

Do not install new CLIs or create new tracker integrations as part of hydration unless explicitly asked.

### 1. Inventory the available sources

Identify:

* Which source documents are available
* Their versions or dates, where known
* Whether the repository is available
* Which branch or commit is being inspected
* Which issue tracker integration, CLI, MCP server, or markdown issue files are available
* Whether conversation decisions may supersede written documents
* Any sources that appear stale or contradictory

Do not treat an apparently stale document as authoritative without noting the discrepancy.

### 2. Read the PRD and identify

* The feature goal
* Target users or personas
* Core user problems
* Functional requirements
* Non-functional requirements
* Explicit non-goals
* Success criteria
* Important terminology
* User-visible states and failure cases
* Permissions or role-specific behaviour
* Compatibility, migration, or rollout requirements
* Requirements explicitly deferred to later work

Only extract items actually stated or directly implied by the source.

### 3. Read `CONTEXT.md` and identify

* Existing architecture
* Important services, modules, packages, APIs, jobs, tables, screens, or workflows
* Naming conventions
* Domain terms
* Existing constraints
* Established implementation patterns
* Test frameworks and test locations
* Feature-flag or rollout conventions
* Error-handling and observability conventions
* Known technical debt or areas that should not be modified

### 4. Inspect the repository where available

Use the codebase to verify implementation-relevant context, including:

* Current user flow
* Relevant entry points
* Existing files and modules
* Existing API and schema contracts
* State transitions and domain invariants
* Authorization checks
* Validation and error handling
* Existing tests and fixtures
* Background jobs or asynchronous processing
* Logging, metrics, and audit behaviour
* Feature flags and configuration
* Migration and backward-compatibility patterns

Repository inspection should locate the implementation area, not design the solution in advance.

Do not list a file merely because its name appears relevant. Verify its relationship to the issue.

When repository access is unavailable, say that the current implementation area was not verified.

### 5. Review relevant docs and conversation notes

Look for:

* Clarifying decisions
* Superseding decisions
* Prior implementation details
* Edge cases
* Dependencies between issues
* Known risks
* Ambiguous areas
* Explicitly rejected approaches
* Rollout or migration constraints
* Decisions about ownership between frontend, backend, infrastructure, or other systems

### 6. Build a requirement-to-issue coverage map

Before rewriting issues, map each relevant requirement to the issue or issues intended to implement it.

Use this pass to detect:

* PRD requirements not represented by any issue
* Requirements represented by multiple issues without a clear ownership boundary
* Issues that contain scope unsupported by the source material
* Requirements assigned to the wrong implementation layer
* Dependencies that are missing from issue descriptions
* Potential sequencing problems

Do not add an uncovered requirement to the nearest issue merely to make the coverage complete.

Report uncovered requirements in the hydration summary unless explicitly asked to create additional issues.

### 7. Hydrate each issue

For each issue:

1. Preserve its original intent.
2. Identify the exact requirement it owns.
3. Add the minimum context needed for implementation.
4. Identify the verified implementation surface.
5. Clarify observable target behaviour.
6. Strengthen acceptance criteria without broadening them.
7. Add relevant tests.
8. Add dependencies and sequencing information.
9. Add explicit non-goals.
10. Capture unresolved questions.
11. Assign readiness based on the readiness rules below.

### 8. Perform a cross-issue consistency pass

After hydrating all issues, verify that:

* Terminology is consistent
* Shared APIs and schemas are described consistently
* Acceptance criteria do not contradict one another
* Responsibility is not unintentionally duplicated
* Dependencies point in the correct direction
* No issue relies on behaviour that another issue explicitly excludes
* Sequencing is viable
* Blocking questions are consistently represented
* The same requirement has not been broadened differently across issues
* Every referenced related issue exists in the supplied issue set or is clearly external

Report overlaps, gaps, contradictions, or dependency cycles in the hydration summary. Do not silently redesign the issue breakdown.

## Readiness rules

### Ready

Mark an issue `Ready` when:

* Its requirement is supported by an identifiable source.
* Its scope and non-goals are clear.
* Its target behaviour is concrete enough to implement.
* Relevant dependencies are available or clearly sequenced.
* Required API, schema, UX, permission, and state decisions have been made.
* Acceptance criteria are testable.
* There are no unresolved questions that materially affect implementation.

An implementation detail that can be resolved through normal repository exploration does not automatically make an issue `Not ready`.

### Not ready

Mark an issue `Not ready` when implementation is blocked by matters such as:

* Conflicting product requirements
* An unresolved API or schema contract
* Missing UX behaviour required to implement the flow
* Undefined permissions or authorization behaviour
* An unavailable prerequisite
* Unsupported scope that materially changes the work
* An unclear ownership boundary between issues
* A missing migration or compatibility decision
* A required dependency whose contract is not yet known

Non-blocking questions do not make an issue `Not ready`.

Include a concise readiness reason whenever an issue is `Not ready`.

## Output format

Each hydrated issue must include the following sections.

---

# Issue: `<issue title>`

`Original title: <original title>`  
Include only when the title was changed.

Preserve existing issue metadata above or below the title where supplied.

## Readiness

`Ready` or `Not ready`

For `Not ready` issues, include:

`Reason: <brief explanation of the blocking decision or dependency>`

## Context capsule

Briefly explain:

* Why this issue exists
* Which user or problem it serves
* Which requirement this issue owns
* How it fits into the larger feature
* What neighbouring issues are responsible for, where that boundary matters

Keep this concise, but make it useful for someone opening the issue with no prior context.

Do not reproduce the full PRD.

## Source references

Include all relevant source references that directly support the issue.

Use references such as:

* PRD section: `<section name>`
* `CONTEXT.md` terms: `<term/module/service>`
* Repository: `<path / symbol / schema>`
* Relevant docs: `<doc name / section>`
* Related previous issues: `#123`, `#124`
* Conversation decision: `<brief summary of decision>`
* Design reference: `<screen, component, flow, or state>`

Do not cite sources that were not actually used.

Where sources conflict, cite each source and explain the conflict under `Open questions`.

## Current behaviour

Describe what exists today.

Include known relevant:

* Files
* Modules
* APIs
* Screens
* Jobs
* Tables
* Services
* Config
* Existing user flows
* State transitions
* Authorization behaviour
* Error behaviour
* Existing tests
* Existing limitations

Clearly identify whether the behaviour was:

* Observed in the repository
* Described by current-state documentation
* Not verified

If the current behaviour is unknown, say so explicitly.

Do not infer current behaviour from target-state requirements.

## Implementation context

Include implementation-relevant facts already established by the sources.

Depending on the issue, this may include:

* Components likely to be changed
* Existing extension points
* Existing domain models or invariants
* API contracts
* Schema fields and relationships
* Events, queues, or jobs
* Configuration or feature flags
* Authentication and authorization boundaries
* Error and retry semantics
* Idempotency requirements
* Backward-compatibility constraints
* Migration or backfill requirements
* Accessibility requirements
* Logging, metrics, audit, or observability requirements
* Existing test utilities or fixtures

Only include subsections relevant to the issue.

Do not prescribe a new architecture unless the source material already made that decision.

Do not present a guessed implementation location as verified. Label an uncertain location as a candidate and explain the evidence.

## Target behaviour

Describe what should be true after this issue is completed.

Focus on:

* Observable user or system behaviour
* State changes
* Contract changes
* Required failure behaviour
* The implementation outcome owned by this issue

Distinguish product behaviour from implementation constraints.

Do not add features beyond the original issue.

## Dependencies and sequencing

List relevant dependencies using the supplied issue IDs where available.

Include:

* `Depends on`: Work that must be completed first
* `Blocks`: Work that cannot proceed until this issue is complete
* `Related`: Connected work without a strict ordering requirement
* `External dependency`: A service, decision, team, or artifact outside the issue set

For each strict dependency, briefly explain why the ordering is required.

If there are no known dependencies, write:

`None identified.`

Do not invent dependencies solely because two issues concern the same feature.

## Acceptance criteria

Add concrete, testable bullet points.

Preserve valid existing acceptance criteria. Clarify or strengthen them without changing their meaning.

Acceptance criteria should be specific enough that a developer, reviewer, or coding agent can verify completion.

Include known edge cases, error states, empty states, permission cases, and compatibility requirements only when supported by the source material.

Use this style:

* Given `<context>`, when `<action>`, then `<expected result>`
* The implementation must `<specific requirement>`
* The system must not `<explicitly excluded behaviour>`
* Existing `<contract or behaviour>` must remain unchanged when `<condition>`

Avoid:

* Vague criteria such as `works correctly`
* Unverifiable criteria such as `is user-friendly`
* Internal implementation choices unless they are explicit constraints
* Repeating the issue description without adding a testable outcome
* Turning speculative best practices into requirements

## Test plan

Provide practical guidance using the project's existing test patterns where known.

Do not require a class of test that is irrelevant to the issue.

When the source issue already includes a strong testing strategy, preserve and sharpen it instead of replacing it with generic test advice. Capture:

* The smallest failing tests expected before implementation, especially when TDD is required.
* A coverage matrix for each meaningful dispatch path, overload, mode, role, state, or data shape.
* Boundary and error cases with exact inputs and expected outputs or error codes.
* Regression cases that must stay green, including named historical bugs, corpus IDs, passlist entries, snapshots, goldens, or fixture names.
* The required test layers, such as unit, integration, end-to-end, migration, contract, or drift tests, only when supported by the source or existing project practice.
* Exact verification commands and quality gates when known, such as typecheck, lint, format, test, clippy, or build commands.
* Explicit exclusions, such as tests that should not be added because that scenario remains intentionally unsupported.

### Unit tests

List expected unit-level coverage, including relevant:

* Success cases
* Boundary conditions
* Validation failures
* State transitions
* Permission checks
* Retry or idempotency logic
* Serialization or mapping behaviour

Identify existing test files, fixtures, factories, or utilities where verified.

### Integration tests

List expected integration coverage for relevant boundaries, such as:

* API to service
* Service to database
* Event producer to consumer
* Frontend to API
* External-service adapters
* Migrations against representative data

### End-to-end tests

Include end-to-end coverage when the issue changes a critical user flow or when the project already tests that flow at this level.

Otherwise write:

`Not required for this issue based on the available context.`

### Regression tests

Identify existing behaviour that must remain unchanged.

Include this section when the work modifies an established flow or shared component.

### Manual verification

List clear manual steps to verify the issue.

Include, where relevant:

* Required account role or test data
* Configuration or feature-flag state
* Actions to perform
* Expected visible result
* Expected persisted state
* Expected logs, events, or side effects
* Negative or permission case
* Cleanup steps

Include exact test commands only when they are known from the repository or documentation.

## Non-goals

List things this issue must not attempt to solve.

Use this section to prevent scope creep and clarify ownership boundaries.

Examples:

* Do not redesign the full flow.
* Do not change unrelated APIs.
* Do not introduce new user-facing settings.
* Do not refactor unrelated modules.
* Do not migrate historical data unless explicitly required.
* Do not change authorization behaviour outside this flow.
* Do not implement work owned by `#123`.
* Do not solve open questions inside this issue unless explicitly required.

Non-goals must be supported by explicit exclusions, the issue boundary, or the ownership of related issues. Do not invent arbitrary restrictions.

## Open questions

List anything still ambiguous, conflicting, or unverified.

Use this format:

* `[Blocking]` `<question>` - `<decision or information needed>`
* `[Non-blocking]` `<question>` - `<why implementation can proceed without it>`

Where known, identify the appropriate decision owner, such as Product, Design, Platform, Security, or the API owner.

Do not include questions that can be answered through ordinary repository inspection.

If there are no open questions, write:

`None.`

---

## Hydration rules

### Do

* Ground every addition in the PRD, `CONTEXT.md`, repository, docs, existing issues, or conversation.
* Make the issue understandable from a fresh context window.
* Add implementation-relevant context where it is already known.
* Use exact domain, API, schema, and module names.
* Add specific source references so the developer can trace requirements.
* Preserve issue scope and ownership boundaries.
* Preserve valid original acceptance criteria.
* Add testable acceptance criteria.
* Preserve source-backed testing strategy details, including exact examples, edge cases, regression IDs, and required quality gates.
* Identify dependencies and sequencing.
* Distinguish observed behaviour from intended behaviour.
* Mark blocked issues as `Not ready`.
* Record missing source access and unverified assumptions.
* Report coverage gaps and overlaps separately from individual issue hydration.
* Keep repeated feature-level context concise.

### Do not

* Do not invent product requirements.
* Do not add nice-to-have features.
* Do not broaden the issue.
* Do not silently resolve ambiguity.
* Do not assume implementation details unsupported by evidence.
* Do not rewrite the issue into a different feature.
* Do not remove important constraints from the original issue.
* Do not accept unsupported original scope as verified.
* Do not duplicate the full PRD inside each issue.
* Do not create, merge, split, or reorder issues unless explicitly instructed.
* Do not assign estimates, owners, or priorities without evidence.
* Do not turn generic best practices into acceptance criteria.
* Do not replace precise source-backed test requirements with generic unit/integration/e2e boilerplate.
* Do not claim that a file or component is affected without verifying the connection.
* Do not mark an issue `Not ready` for questions that normal implementation exploration can answer.
* Do not conceal conflicts between documentation and code.

## Quality bar

A hydrated issue is successful when:

* A developer or coding agent can understand and implement it without the full original conversation.
* The issue clearly explains why it exists and which requirement it owns.
* The issue has enough verified technical context to find the implementation area.
* Current behaviour and target behaviour are clearly separated.
* Source references are precise and traceable.
* Dependencies and issue boundaries are explicit.
* Acceptance criteria are concrete and reviewable.
* Tests are clear enough to guide implementation.
* Existing behaviour that must not regress is identified.
* Scope boundaries are explicit.
* Remaining ambiguity is captured as open questions.
* Readiness reflects actual implementation blockers.
* The issue does not contain requirements invented during hydration.

## Final consistency checks

Before returning the result, confirm that:

* Every hydrated statement can be traced to a source or is clearly marked unknown.
* Every `Ready` issue has no blocking open questions.
* Every `Not ready` issue identifies the specific blocker.
* No unsupported requirement has been silently accepted or removed.
* No issue was unintentionally broadened.
* Existing valid acceptance criteria were not weakened.
* Cross-issue terminology is consistent.
* Dependencies do not form an unexplained cycle.
* Shared contracts are described consistently.
* Missing PRD coverage was not hidden inside an unrelated issue.

## Final output

Return the hydrated issues in the same order as the input issues.

Preserve original issue titles unless there is a clear typo or the title is too vague to identify the work.

If changing a title, keep the original title visible as:

`Original title: <original title>`

Do not create additional implementation issues unless explicitly requested.

If issues were updated in a tracker or markdown files, state where they were written. If no update path was available, state that the hydrated issues are returned as markdown only.

At the end, include:

## Hydration summary

* Total issues reviewed: `<number>`
* Ready: `<number>`
* Not ready: `<number>`
* Requirements with no issue coverage: `<short bullets or None>`
* Duplicate or overlapping ownership: `<short bullets or None>`
* Source conflicts or stale documentation: `<short bullets or None>`
* Common missing context added: `<short bullets>`
* Dependencies or sequencing added: `<short bullets or None>`
* Blocking questions across issues: `<short bullets or None>`
* Current behaviour not verified: `<short bullets or None>`
