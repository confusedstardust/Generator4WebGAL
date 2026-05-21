# Phase 4: Validator — Runtime Prompt Template

Orchestrator fills `{{PLACEHOLDERS}}` then sends as Agent prompt.

---

TASK: Validate the complete WebGAL game. Report errors ONLY.

PROJECT DIR: {{PROJECT_DIR}}
STATE DIR: {{STATE_DIR}}

## Inputs to Read
- All scene files: `{{PROJECT_DIR}}/public/game/scene/*.txt`
- `{{STATE_DIR}}/characters.json` — verify speaker names
- `{{STATE_DIR}}/variables.json` — verify variable names/types
- `{{STATE_DIR}}/scene_graph.json` — verify all scenes exist
- `{{STATE_DIR}}/branch_map.json` — verify branches implemented
- `{{STATE_DIR}}/ending_matrix.json` — verify ending reachability
- Asset files: `{{PROJECT_DIR}}/public/game/background/`, `{{PROJECT_DIR}}/public/game/figure/`
- `{{PROJECT_DIR}}/assets_manifest.json` — verify every asset is used in at least one scene

## Validation Checklist

### Syntax
- [ ] Every `callScene` has `;` on next line
- [ ] `end;` only in start.txt and ending files
- [ ] All `jumpLabel` targets exist in same file
- [ ] No multi-condition `-when=`
- [ ] Boolean values are 0/1, never true/false

### References
- [ ] All `changeBg:*.webp` exist on disk
- [ ] All `changeFigure:*.webp` exist on disk
- [ ] All `miniAvatar:*.webp` exist on disk
- [ ] All `callScene:*.txt` targets exist
- [ ] All speaker names match characters.json

### Variables
- [ ] All `setVar` names declared in variables.json
- [ ] All `-when=` names declared in variables.json
- [ ] ≤ 12 globals, attitude vars use numeric range

### Endings
- [ ] Exactly {{ENDING_COUNT}} endings, all reachable by path tracing
- [ ] Every ending file exists and has ≥30 lines

### Figures
- [ ] Every scene: open-clear → show before speak → close-clear before callScene

### Config
- [ ] `config.txt` has Game_name, Title_img, Game_Logo set and pointing to existing files

### Limits
- [ ] {{MIN_CHARS}}-{{MAX_CHARS}} characters, max depth {{MAX_DEPTH}}, 3-4 options/choice

## Output: {{STATE_DIR}}/validation_report.json
```json
{
  "summary": {"total_scenes":N,"total_lines":N,"errors":N,"warnings":N,"passed":bool},
  "checks": {"syntax":{},"references":{},"variables":{},"endings":{},"limits":{},"figures":{}},
  "errors": [{"file":"...","line":N,"message":"..."}],
  "warnings": [{"file":"...","line":N,"message":"..."}],
  "unreachable_endings":[],
  "missing_assets":[],
  "undefined_variables":[],
  "undefined_speakers":[],
  "figure_violations":[]
}
```

## Boundaries
- Report ONLY — never modify files
- Errors = game will crash. Warnings = best practice deviation.

## Output
Write validation_report.json. Report: total errors and warnings found.
