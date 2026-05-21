# Phase 5: Repair Agent — Runtime Prompt Template

Orchestrator fills `{{PLACEHOLDERS}}` per repair cycle, sends as Agent prompt.

---

TASK: Fix validation errors. Minimize changes.

PROJECT DIR: {{PROJECT_DIR}}
STATE DIR: {{STATE_DIR}}

## Input
- `{{STATE_DIR}}/validation_report.json` — errors to fix
- Affected scene files: `{{PROJECT_DIR}}/public/game/scene/*.txt`
- `{{STATE_DIR}}/characters.json` — for speaker name fixes
- `{{STATE_DIR}}/variables.json` — for variable name fixes
- `{{PROJECT_DIR}}/assets_manifest.json` — for asset reference fixes

## Errors to Fix
{{ERRORS_LIST}}

## Fix Strategies

| Error | Fix |
|-------|-----|
| `callScene` missing `;` | Add `;` after callScene |
| `end;` in sub-scene | Replace with `callScene:next.txt;` + `;` |
| `jumpLabel` to undefined target | Find or create the label |
| Boolean true/false | Replace with 1/0 |
| Missing asset reference | Search disk for closest match; if none, mark unfixable |
| Unknown speaker name | Map to closest character ID |
| Undefined variable | Add to variables.json or use closest match |
| Unreachable ending | Trace backward, add missing connection |
| Ending file too short | Add closure content to ≥30 lines |
| Unused asset in manifest | Match by filename keyword to scene, insert `changeBg` at emotional high point |

## Rules
1. Fix ONLY what the error report flags — no rewrites
2. Preserve original intent when guessing
3. Mark truly unfixable errors honestly
4. Do NOT modify shared/state/ planning artifacts

## Output
1. Fix affected .txt files in-place
2. Write `{{STATE_DIR}}/repair_log.json`:
```json
{"cycle":{{CYCLE_NUM}},"changes":[{"file":"...","line":N,"error":"...","fix":"..."}],"unfixable":[]}
```
Report: number of fixes applied and any unfixable errors.
