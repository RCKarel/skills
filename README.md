# Skills

Local Codex skills for recurring engineering workflows.

## Included skills

### `complexity-analyzer`

Reviews implementation tickets for scope, risk, dependencies, and reviewability, then proposes or applies decompositions before rerunning `$hydrate-issues` on affected tickets.

Use it after issues are generated or imported and before assigning them to a developer or coding agent.

The skill is defined in [`complexity-analyzer/SKILL.md`](complexity-analyzer/SKILL.md). The OpenAI agent display metadata lives in [`complexity-analyzer/agents/openai.yaml`](complexity-analyzer/agents/openai.yaml).

### `hydrate-issues`

Reviews generated implementation issues against available source material, then enriches them with implementation context, source references, readiness, dependencies, acceptance criteria, tests, non-goals, and open questions.

Use it when issues from a PRD, feature brief, planning conversation, or implementation breakdown are too thin for a developer or coding agent to implement from a fresh context window.

The skill is defined in [`hydrate-issues/SKILL.md`](hydrate-issues/SKILL.md). The OpenAI agent display metadata lives in [`hydrate-issues/agents/openai.yaml`](hydrate-issues/agents/openai.yaml).

## Usage

Ask Codex to use `$hydrate-issues` with the generated issues and any available source material, such as:

- PRD or feature brief
- `CONTEXT.md`
- relevant repository files
- architecture or API docs
- planning notes or prior decisions
- related issues

The skill preserves issue scope. It should clarify and source what already exists, not invent requirements or expand product scope.

## Repository layout

```text
complexity-analyzer/
  SKILL.md
  agents/
    openai.yaml

hydrate-issues/
  SKILL.md
  agents/
    openai.yaml
```
