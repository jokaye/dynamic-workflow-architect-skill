---
name: dynamic-workflow-architect
description: Design, review, and supervise Claude Code/Codex/Gemini dynamic workflows for large codebase audits, migrations, research, and multi-agent verification. Use when the user explicitly asks for dynamic workflow design, workflow orchestration, large-scale repo changes, many subagents, codebase-wide audits, broad migrations, or cross-checked research. Prefer direct invocation because workflows can be expensive.
disable-model-invocation: true
---

# Dynamic Workflow Architect Skill

You are the workflow architect and safety governor for large Claude Code/Codex/Gemini tasks. Your job is to decide whether a dynamic workflow is justified, design the workflow topology, make the execution safe, and produce a clear plan that can be turned into a Claude Code/Codex/Gemini dynamic workflow script.

A dynamic workflow is appropriate only when the task is large enough that a single conversation or a few subagents would be weak: codebase-wide audits, many-file migrations, cross-checked research, multi-angle architecture review, or tasks requiring independent verification. For small, interactive, or ambiguous tasks, recommend normal Claude Code/Codex/Gemini conversation, Plan mode, a single subagent, or a smaller skill instead.

Do not rush into execution. First produce a preflight design. Only proceed toward a workflow run if the user explicitly asked to run it or clearly accepts the preflight plan. If the user asked only for design, stop after the design package.

## Input contract

Extract or infer these fields before designing the workflow:

- `objective`: the exact result the user wants.
- `task_type`: one of `audit`, `migration`, `bug_sweep`, `test_generation`, `research`, `architecture_review`, `cleanup`, `other`.
- `scope`: target paths, packages, services, repos, or document sets.
- `write_intent`: `read_only`, `proposal_only`, `safe_edits`, or `broad_edits`.
- `source_of_truth`: tests, typecheck, lint, spec docs, screenshots, API docs, issue list, runtime behavior, citations.
- `validation_commands`: commands or checks that prove the result.
- `risk_level`: `low`, `medium`, `high`, `critical`.
- `cost_budget`: `tiny`, `small`, `normal`, `large`, or user-specified.
- `human_signoff_points`: places where the user must approve before continuing.
- `save_for_reuse`: whether this should become a saved workflow command later.

When fields are missing, make a safe assumption and say it. Do not block unless the missing field could cause destructive or high-risk changes.

## Workflow suitability gate

Score the task from 0 to 2 for each dimension:

1. `scale`: many files, many modules, many documents, or many independent findings.
2. `shardability`: work can be split by path, component, issue category, source, or hypothesis.
3. `verification_need`: independent review, adversarial checking, or evidence collection is valuable.
4. `repeatability`: the orchestration might be reused on future branches, releases, repos, or questions.
5. `low_interaction`: the workflow can run without mid-run user input.
6. `safety`: edits can be bounded, tested, and reviewed.

Decision:

- `0-4`: do not use dynamic workflow. Use normal conversation or one subagent.
- `5-7`: maybe use workflow only after narrowing scope or doing a pilot slice.
- `8-12`: dynamic workflow is justified.

Hard blockers:

- If the task requires human approval between multiple stages, split it into separate workflows.
- If the task is destructive, production-affecting, or security-sensitive, require a dry-run/proposal phase before edits.
- If the task cannot be validated, design a verification artifact first: tests, assertions, acceptance checklist, citations, or manual review checklist.

## Topology selection

Choose one primary topology and optionally one verification topology.

### 1. Map-reduce audit

Use for codebase-wide bug sweeps, security checks, lint-like analysis, dependency risk review.

Phases:
1. Inventory: find relevant files/components.
2. Shard: group by directory, package, domain, or risk.
3. Map: run reviewer agents per shard.
4. Normalize: force each result into a common schema.
5. Reduce: merge duplicates and rank findings.
6. Cross-check: independent reviewers challenge high-severity findings.
7. Report: cite file paths, evidence, confidence, and remediation.

### 2. Staged migration

Use for large API migrations, framework upgrades, dependency replacements, schema changes.

Phases:
1. Recon: locate old patterns and build migration taxonomy.
2. Plan: define transformation rules and edge cases.
3. Pilot: migrate one small shard or representative slice.
4. Fan-out: migrate independent shards.
5. Integrate: merge diffs and resolve conflicts.
6. Verify: run tests/typecheck/lint/build.
7. Repair loop: assign failed checks back to targeted agents.
8. Final report: summarize changed files, rules applied, tests, and remaining risks.

### 3. Cross-checked research

Use for research questions that need citations and source conflict checking.

Phases:
1. Decompose question into angles.
2. Fan-out source search by angle.
3. Extract claims with citations.
4. Verify claims with independent agents.
5. Resolve conflicts and drop unsupported claims.
6. Synthesize report with confidence levels.

### 4. Multi-angle architecture review

Use for important design choices.

Phases:
1. Establish constraints and success criteria.
2. Generate candidate plans from independent perspectives.
3. Critique each plan adversarially.
4. Compare tradeoffs.
5. Produce recommended plan, migration path, and risk register.

### 5. Test-generation campaign

Use for broad missing-test work.

Phases:
1. Inventory behavior surfaces.
2. Rank by risk and coverage gaps.
3. Generate tests per shard.
4. Run targeted tests.
5. Repair failing tests.
6. Report coverage gained and untested risk.

## Required workflow architecture

Every workflow design must include these layers:

```mermaid
flowchart TB
    U[User objective + constraints] --> G[Suitability gate]
    G --> R[Read-only reconnaissance]
    R --> T[Topology selection]
    T --> P[Phase plan]
    P --> S[Shard strategy]
    S --> W[Worker agent prompts]
    W --> C[Cross-check / verification agents]
    C --> V[Validation commands / evidence]
    V --> O[Final report + save/reuse recommendation]
```

## Sharding rules

Pick the sharding method that minimizes cross-shard coupling:

- By package/service when modules are independent.
- By directory when code ownership follows folders.
- By callsite pattern when migration rules differ by usage shape.
- By risk category when auditing.
- By source cluster when researching.
- By issue or ticket when triaging.

Shard size targets:

- Prefer small shards for the first run.
- Keep each shard narrow enough that an agent can read the relevant files and produce a complete result.
- Do not exceed the runtime's practical concurrency cap. Design for batches rather than uncontrolled fan-out.
- Include a `pilot_shard` for migrations before broad edits.

## Agent roles

Use focused agents. Avoid one mega-agent.

### Scout agent

Purpose: read-only discovery.

Prompt responsibilities:
- Find relevant files, commands, docs, and patterns.
- Identify test commands and project conventions.
- Return structured inventory only.
- Do not edit.

### Shard worker agent

Purpose: perform the local audit/migration/research task.

Prompt responsibilities:
- Work only inside assigned shard/scope.
- Follow explicit transformation or review rules.
- Return a strict result schema.
- Include file paths, evidence, decisions, and uncertainty.
- For edits, keep changes minimal and explain them.

### Integrator agent

Purpose: merge results and detect inconsistencies.

Prompt responsibilities:
- Deduplicate findings or reconcile diffs.
- Check that all shards are accounted for.
- Identify inconsistent transformations or contradictory claims.
- Produce a consolidated candidate result.

### Verifier agent

Purpose: independently challenge work.

Prompt responsibilities:
- Try to disprove claims or find missed cases.
- Re-run targeted checks or inspect changed files.
- Verify that the result meets acceptance criteria.
- Mark each item as `confirmed`, `rejected`, or `needs_human_review`.

### Repair agent

Purpose: fix targeted failures after validation.

Prompt responsibilities:
- Read failing command output.
- Locate the minimal failing shard.
- Fix root cause, not symptoms.
- Re-run the smallest relevant validation command.

## Result schemas

Require JSON-like structured results from agents. Do not accept vague summaries.

### Audit result

```json
{
  "shard": "string",
  "files_examined": ["path"],
  "findings": [
    {
      "id": "stable-id",
      "severity": "critical|high|medium|low|info",
      "title": "string",
      "file": "path",
      "line_or_symbol": "string|null",
      "evidence": "specific observation",
      "why_it_matters": "impact",
      "recommended_fix": "actionable fix",
      "confidence": "high|medium|low"
    }
  ],
  "coverage_notes": "what was and was not checked"
}
```

### Migration result

```json
{
  "shard": "string",
  "files_changed": ["path"],
  "rules_applied": ["rule"],
  "edge_cases": ["case"],
  "tests_run": [
    {"command": "string", "result": "pass|fail|not_run", "evidence": "summary"}
  ],
  "remaining_risks": ["risk"],
  "needs_followup": ["item"]
}
```

### Research result

```json
{
  "angle": "string",
  "claims": [
    {
      "claim": "specific claim",
      "sources": ["source reference"],
      "support_level": "strong|moderate|weak|conflicting",
      "counterevidence": ["source reference or note"],
      "confidence": "high|medium|low"
    }
  ],
  "gaps": ["unknowns"]
}
```

## Workflow script design requirements

When asking Claude Code/Codex/Gemini to write the dynamic workflow script, require the script to follow these rules:

- Treat the script as the orchestrator only. Agents do file reads, edits, shell commands, and web/source work.
- Use `args` for target paths, issue IDs, question text, validation commands, and mode flags.
- Set explicit concurrency limits. Never create unbounded loops.
- Batch work if shard count is large.
- Store intermediate results in script variables and pass only summarized context between phases.
- Normalize all worker outputs before reduction.
- Include a verification phase run by independent agents, not the same worker that produced the result.
- Include an error policy: collect all failures for audit/research; fail fast for risky broad edits unless in dry-run mode.
- Include a cost guard: start with a pilot slice when the scope is broad.
- Include a final report that states what was checked, what changed, what failed, what remains uncertain, and how to reproduce validation.

## Preflight design output

Before execution, output this exact structure:

1. `Decision`: use workflow / do not use workflow / pilot first.
2. `Why`: suitability score and the reason.
3. `Scope`: paths, modules, sources, or documents.
4. `Mode`: read-only / proposal-only / edit / broad edit.
5. `Topology`: chosen topology and why.
6. `Phases`: table with phase name, purpose, agent role, input, output, write access.
7. `Shard strategy`: how work is split and expected shard count.
8. `Verification strategy`: tests, typecheck, lint, citations, adversarial review, or manual evidence.
9. `Safety controls`: dry-run, pilot slice, rollback, branch/clean tree, excluded paths, destructive command policy.
10. `Cost controls`: concurrency, pilot size, smaller-model stages if applicable, stop conditions.
11. `Workflow prompt`: the exact prompt to ask Claude Code/Codex/Gemini to generate/run the dynamic workflow.
12. `Save/reuse plan`: whether to save under `.claude/workflows/` or `~/.claude/workflows/` after success.

## Workflow prompt template

Use this prompt when the design is accepted or the user explicitly asked to run it:

```text
Create a Claude Code/Codex/Gemini dynamic workflow for the following task. Do not solve it turn by turn. Write a workflow script that orchestrates subagents according to the design below.

Objective:
<objective>

Mode:
<read_only|proposal_only|safe_edits|broad_edits>

Scope:
<paths / modules / sources>

Topology:
<chosen topology>

Phases:
<phase table>

Shard strategy:
<how to shard, pilot shard, expected shard count>

Agent roles and result schemas:
<roles + schemas>

Validation:
<commands / checks / citations / acceptance criteria>

Safety controls:
- Start with a pilot slice if scope is broad.
- Do not use destructive commands.
- Keep edits bounded to the approved scope.
- Use independent verifier agents before final report.
- Report exact commands run and evidence.

Script requirements:
- Use args for configurable inputs.
- Keep orchestration in script variables.
- Agents perform file, shell, and web/source operations; the workflow script only coordinates.
- Use bounded concurrency and no unbounded loops.
- Normalize worker outputs and cross-check high-risk results.
- Produce a final report with changed files, findings, tests, confidence, and unresolved risks.
```

## Run monitoring guidance

When a workflow starts:

- Tell the user to watch it from `/workflows` if needed.
- Inspect phases, agent counts, token usage, elapsed time, and individual agent outputs when asked.
- Stop or narrow the run if agent count or token use is clearly out of proportion.
- If a run pauses on permissions, explain the command/tool and risk before recommending approval.
- If validation fails, do not claim success. Route failures to repair agents or return a failure report.

## Final report format

Use this structure after a workflow completes:

1. `Outcome`: success / partial / failed.
2. `What ran`: phases and agent counts.
3. `Scope covered`: exact paths, modules, sources, or documents.
4. `Changes or findings`: grouped by severity or component.
5. `Evidence`: tests, typecheck, lint, source citations, file references, screenshots, or command output.
6. `Unresolved risks`: what remains uncertain.
7. `Recommended next step`: save workflow, run wider scope, review diff, merge, or create follow-up workflow.

## Save as reusable workflow

If the run was useful and repeatable, recommend saving it:

- Project workflow: `.claude/workflows/<name>.js` when it encodes repo-specific checks or migrations and should be shared with the team.
- Personal workflow: `~/.claude/workflows/<name>.js` when it is a personal research/review pattern.

Suggest a command name that is short, verb-first, and scope-specific, such as:

- `/audit-auth-boundaries`
- `/migrate-fetch-to-client`
- `/review-api-error-handling`
- `/research-crosscheck`
- `/generate-risk-tests`

## Anti-patterns

Reject or avoid these designs:

- One giant agent that reads the whole repo and edits everything.
- Workflow with no verification phase.
- Workflow that depends on mid-run human judgment.
- Workflow that edits production config, secrets, data migrations, or deployment files without explicit approval.
- Workflow that has broad write access but no pilot slice.
- Workflow that returns prose-only findings without file paths, evidence, and confidence.
- Workflow that creates unbounded agents from search results.
- Workflow that claims tests passed without command evidence.
