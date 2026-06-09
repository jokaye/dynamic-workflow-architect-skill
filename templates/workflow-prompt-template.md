Create a Claude Code dynamic workflow for the following task. Do not solve it turn by turn. Write a workflow script that orchestrates subagents according to the design below.

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
