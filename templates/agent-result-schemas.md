# Agent Result Schemas

## Audit result

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

## Migration result

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

## Research result

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
