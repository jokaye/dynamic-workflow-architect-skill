# Dynamic Workflow Architect Skill

A reusable agent skill for designing and governing dynamic multi-agent workflows
for Claude Code, Codex, and Gemini-style coding agents.

The skill helps decide whether a task is large enough for dynamic workflow
orchestration, then produces a preflight design with topology, sharding,
agent roles, result schemas, validation commands, and safety controls.

## When To Use

Use this skill for work that is too broad or risky for a single turn-by-turn
agent session, such as:

- Codebase-wide audits and bug sweeps
- Large API or framework migrations
- Cross-checked research with citations
- Multi-angle architecture reviews
- Broad test-generation campaigns
- Multi-agent verification and repair loops

For small, interactive, or ambiguous tasks, use a normal agent conversation,
Plan mode, or a single focused subagent instead.

## What It Produces

The skill is intentionally designed as a workflow architect, not as an
automatic executor. It first creates a preflight package that includes:

- Suitability score and decision
- Read/write mode and risk level
- Chosen topology
- Phase plan
- Shard strategy and pilot scope
- Worker, integrator, verifier, and repair-agent responsibilities
- Structured result schemas
- Validation and evidence requirements
- Cost and safety controls

## Installation

Install as a project skill:

```bash
mkdir -p .claude/skills/dynamic-workflow-architect
cp SKILL.md .claude/skills/dynamic-workflow-architect/SKILL.md
cp -R templates .claude/skills/dynamic-workflow-architect/templates
```

Install as a personal skill:

```bash
mkdir -p ~/.claude/skills/dynamic-workflow-architect
cp SKILL.md ~/.claude/skills/dynamic-workflow-architect/SKILL.md
cp -R templates ~/.claude/skills/dynamic-workflow-architect/templates
```

## Example

```text
/dynamic-workflow-architect Migrate all internal fetch() calls to the apiClient.
First design a pilot workflow for src/services only. Validate with npm test and
npm run typecheck.
```

Expected behavior:

1. Extract the objective, scope, write intent, validation commands, and risks.
2. Score whether a dynamic workflow is justified.
3. Recommend a workflow topology, such as staged migration or map-reduce audit.
4. Define agent roles, shard boundaries, result schemas, and stop conditions.
5. Produce a preflight report before any broad execution.

## Repository Layout

```text
.
├── README.md
├── SKILL.md
└── templates/
    ├── agent-result-schemas.md
    ├── preflight-report.md
    ├── topologies.md
    └── workflow-prompt-template.md
```

## Design Notes

`SKILL.md` uses `disable-model-invocation: true` because dynamic workflows can
spawn many agents and consume substantially more tokens than a normal session.
Invoke it deliberately when workflow design, supervision, and verification are
worth the extra overhead.

The templates are reference material for consistent outputs. The canonical
behavior contract lives in `SKILL.md`.

## Contributors

- [jokaye](https://github.com/jokaye)
