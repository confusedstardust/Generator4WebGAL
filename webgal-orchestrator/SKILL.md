---
name: webgal-orchestrator
description: |
  Pipeline orchestrator for webgal-game. Coordinates the 5-phase game
  building pipeline as compiler-style passes. Each phase runs as an
  isolated Agent with minimal task-specific context — SKILL.md files
  are NEVER loaded at runtime. The orchestrator assembles runtime
  prompts from compact templates, dispatches agents, and enforces
  gate conditions between phases.
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - TodoWrite
  - Agent
  - Skill
---

# WebGAL Orchestrator (Compiler-Style Pipeline)

Pipeline coordinator for building WebGAL visual novels. Drives 5 phases as **isolated compiler passes**. The orchestrator reads compact prompt templates, fills placeholders from project state, spawns agents, and checks gate conditions. Each agent sees only what it needs — no global context dumps.

---

## Shared Resources (read as needed, NEVER load into agent prompts)

| Directory | Purpose |
|-----------|---------|
| `~/.claude/skills/webgal-game/shared/constraints/` | Hard rules (human reference) |
| `~/.claude/skills/webgal-game/shared/contracts/` | I/O contracts (human reference) |
| `~/.claude/skills/webgal-game/shared/schemas/` | JSON schemas (human reference) |
| `~/.claude/skills/webgal-game/shared/state/` | Runtime pipeline artifacts |
| `~/.claude/skills/webgal-game/pipeline/prompt_templates/` | Runtime prompt templates |
| `~/.claude/skills/webgal-game/pipeline/runtime/` | Runtime working directory |

Key constraint files (for orchestrator's own reference when assembling prompts):
- [syntax.md](~/.claude/skills/webgal-game/shared/constraints/syntax.md)
- [limits.md](~/.claude/skills/webgal-game/shared/constraints/limits.md)
- [naming.md](~/.claude/skills/webgal-game/shared/constraints/naming.md)

---

## Pipeline Overview

**Before starting any phase, compute these paths:**

- `PROJECT_DIR` = current working directory (absolute path, no trailing slash)
- `STATE_DIR` = `~/.claude/skills/webgal-game/shared/state` (resolve `~` to absolute path)
- `TEMPLATE_DIR` = `~/.claude/skills/webgal-game/pipeline/prompt_templates`

All `{{PLACEHOLDERS}}` must be filled with resolved absolute paths before Agent dispatch.

```
Phase 1: Narrative Designer  → 1 Agent  (global architecture)
Phase 2: Asset Planner       → 1 Agent  (manifest) + Python scripts
Phase 3: Scene Writer        → N Agents (one per scene, minimal context)
Phase 4: Validator           → 1 Agent  (full validation)
Phase 5: Repair (if errors)  → 1 Agent  per cycle (max 3)
```

Gate conditions between phases enforced strictly. No phase starts before previous phase outputs verified.

---

## Phase 1: Narrative Designer

**Template:** `{{TEMPLATE_DIR}}/phase1_narrative.md`

### Context Assembly
1. Read `{{TEMPLATE_DIR}}/phase1_narrative.md`
2. Fill placeholders:
   - `{{STATE_DIR}}` → computed STATE_DIR
   - `{{SOURCE_MATERIAL}}` → user's source material from conversation
   - `{{MIN_CHARS}}` → 3, `{{MAX_CHARS}}` → 7
   - `{{MIN_SCENES}}` → 5, `{{MAX_SCENES}}` → 15
   - `{{MIN_BRANCHES}}` → 5, `{{MAX_DEPTH}}` → 2
   - `{{ENDING_COUNT}}` → 5

### Dispatch
Agent(subagent_type="general-purpose", description="Phase 1: Narrative Designer", prompt=<filled template>)

### Gate Check
```bash
test -f {{STATE_DIR}}/characters.json && test -f {{STATE_DIR}}/variables.json && test -f {{STATE_DIR}}/scene_graph.json && test -f {{STATE_DIR}}/branch_map.json && test -f {{STATE_DIR}}/ending_matrix.json
```

If any file missing → report failure, stop pipeline.

Collect agent's summary. Proceed to Phase 2.

---

## Phase 2: Asset Planner

**Template:** `{{TEMPLATE_DIR}}/phase2_assets.md`

### Context Assembly
1. Read the prompt template
2. Fill:
   - `{{STATE_DIR}}` → computed STATE_DIR
   - `{{PROJECT_DIR}}` → computed PROJECT_DIR

### Dispatch
Agent(subagent_type="general-purpose", description="Phase 2: Asset Planner", prompt=<filled template>)

Agent writes `assets_manifest.json` to project root.

### Gate Check (Agent returns, then orchestrator runs scripts)
```bash
python ~/.claude/skills/webgal-game/script/generate_assets.py assets_manifest.json
python ~/.claude/skills/webgal-game/script/remove_bg.py public/game/figure/figure_*.webp
python ~/.claude/skills/webgal-game/script/make_avatar.py public/game/figure/figure_*.webp
```

Then verify:
```bash
# Check all manifest entries exist on disk
ls public/game/background/bg_*.webp && ls public/game/figure/figure_*.webp && ls public/game/figure/miniavatar_*.webp
```

If any script fails or assets missing → report failure, stop pipeline.

### Post-Gate: Update config.txt

After assets are verified, the orchestrator updates `public/game/config.txt`:

1. Read existing `public/game/config.txt` to preserve fields not being changed
2. Set `Game_name` to the game title (derived from user's source material)
3. Set `Title_img` to the title background if a `title_*.webp` or `bg_title*.webp` exists in `public/game/background/`; otherwise use the first background from the manifest
4. Set `Game_Logo` to the protagonist's mini avatar (look up protagonist from characters.json, then find `miniavatar_<protagonist_id>.webp` in `public/game/figure/`)
5. Keep `Game_key`, `Title_bgm`, `Enable_Appreciation` as-is

Collect agent's summary. Proceed to Phase 3.

---

## Phase 3: Scene Writer (Per-Scene Loop)

**Template:** `{{TEMPLATE_DIR}}/phase3_scene.md`

This is the critical compiler pass — each scene gets its own isolated Agent with minimal context.

### Context Assembly (per scene)

1. Read `{{STATE_DIR}}/scene_graph.json` → get ordered list of scenes (topological sort from start)
2. Read `{{STATE_DIR}}/characters.json` → character profiles
3. Read `{{STATE_DIR}}/variables.json` → variable definitions
4. Read `{{STATE_DIR}}/branch_map.json` → choice points and variable effects
5. Read `{{PROJECT_DIR}}/assets_manifest.json` → extract all filenames, grouped by subdir
6. Build `{{AVAILABLE_ASSETS}}` once (same for all scenes):
   ```
   Backgrounds: bg_office, bg_journey, bg_home, bg_garden, ...
   Figures: figure_tao_yuanming, figure_wife, ...
   Mini avatars: miniavatar_tao_yuanming, miniavatar_wife, ...
   ```
7. Read `{{TEMPLATE_DIR}}/phase3_scene.md` once

**For each scene** in topological order:

a. **Extract characters subset:**
   - Find characters whose ID appears in `scene.character_ids`
   - For each: include only `name` and `speech_style` (1 line each)
   - Fill `{{CHARACTERS_SECTION}}` as:
     ```
     - 陶渊明: 悠然淡泊，言语简练有诗意
     - 妻子: 温柔体贴，略带忧虑
     ```

b. **Extract variables subset:**
   - Find branches where `branch.scene == scene.id`
   - For each option: extract `sets` keys as variables to set
   - Also check `-when=` conditions on transitions
   - Fill `{{VARIABLES_SECTION}}` as:
     ```
     - setVar:visit_garden=1 (choice option "去花园")
     - setVar:trust=+10 (choice option "坦诚相告")
     ```

c. **Build upstream summary:**
   - Find connections where `to == scene.id`
   - If upstream scene already processed: use the 1-2 sentence summary collected from that agent's result
   - If no upstream (start scene): use "This is the opening scene. Establish the world and protagonist."
   - Fill `{{UPSTREAM_SUMMARY}}`

d. **Build transitions:**
   - Find connections where `from == scene.id`
   - Also check branch_map for choice-based transitions
   - Fill `{{TRANSITIONS_SECTION}}` as:
     ```
     - (default) → callScene:act2_journey.txt
     - If visit_garden==1 → callScene:act4_garden.txt
     ```

e. **Fill remaining placeholders:**
   - `{{PROJECT_DIR}}` → computed PROJECT_DIR
   - `{{SCENE_ID}}` → scene.id
   - `{{FILE_PATH}}` → `{{PROJECT_DIR}}/public/game/scene/` + scene.file

### Dispatch (one Agent per scene)
```
Agent(subagent_type="general-purpose", description="Phase 3: Write scene <scene_id>", prompt=<filled template>)
```

After each agent returns:
- Verify the .txt file exists: `Bash: test -f public/game/scene/<scene.file>`
- Check scene has ≥30 lines: `Bash: wc -l public/game/scene/<scene.file>`
- Collect a 1-2 sentence summary from agent's result (for downstream scenes)

### Gate Check (after all scenes)
```bash
# Verify all scenes from scene_graph exist
for f in $(grep -o '"file":"[^"]*"' {{STATE_DIR}}/scene_graph.json | cut -d'"' -f4); do test -f "{{PROJECT_DIR}}/public/game/scene/$f" || echo "MISSING: $f"; done
```

Also verify `config.txt` has Game_name, Title_img, Game_Logo set and they resolve.

---

## Phase 4: Validator

**Template:** `{{TEMPLATE_DIR}}/phase4_validate.md`

### Context Assembly
1. Read the prompt template
2. Fill:
   - `{{STATE_DIR}}` → computed STATE_DIR
   - `{{PROJECT_DIR}}` → computed PROJECT_DIR
   - `{{ENDING_COUNT}}` → 5, `{{MIN_CHARS}}` → 3, `{{MAX_CHARS}}` → 7, `{{MAX_DEPTH}}` → 2

### Dispatch
Agent(subagent_type="general-purpose", description="Phase 4: Validator", prompt=<filled template>)

### Gate Check
```bash
test -f {{STATE_DIR}}/validation_report.json
```

Check `summary.errors` and `summary.warnings`:
- `errors == 0` → Game complete. Report success with warning count.
- `errors > 0` → Proceed to Phase 5 (Repair).

---

## Phase 5: Repair (Conditional)

**Template:** `{{TEMPLATE_DIR}}/phase5_repair.md`

Only triggered if Phase 4 `errors > 0`. Max 3 repair cycles.

### Context Assembly
1. Read `{{STATE_DIR}}/validation_report.json` → extract errors list
2. Read the prompt template
3. Fill:
   - `{{STATE_DIR}}` → computed STATE_DIR
   - `{{PROJECT_DIR}}` → computed PROJECT_DIR
   - `{{ERRORS_LIST}}` → formatted error entries from report
   - `{{CYCLE_NUM}}` → current cycle (1, 2, or 3)

### Dispatch
Agent(subagent_type="general-purpose", description="Phase 5: Repair cycle <N>", prompt=<filled template>)

### After Each Cycle
Re-run Phase 4 (Validator). If errors == 0, done. If errors remain and cycle < 3, increment cycle and re-run Phase 5.

If cycle > 3 and errors remain → stop, report unfixable errors to user, ask for guidance.

---

## Gate Conditions Summary

| Transition | Check |
|------------|-------|
| 1 → 2 | All 5 JSONs in `{{STATE_DIR}}/` |
| 2 → 3 | All assets from manifest exist on disk (in `{{PROJECT_DIR}}/public/game/`) |
| 3 → 4 | All .txt files from scene_graph exist, config.txt configured |
| 4 → 5 | `{{STATE_DIR}}/validation_report.json` errors > 0 |
| After 5 | Re-run Phase 4. Max 3 repair cycles. |

---

## Failure Handling

| Failure | Action |
|---------|--------|
| Any Agent fails or returns error | Log, stop pipeline, report to user |
| Validator finds errors | Invoke Repair Agent, re-validate |
| Repair cycle exceeds 3 | Stop and ask user for guidance |
| Asset generation script fails | Report which asset/script failed |
| Scene agent produces <30 lines | Flag as failure, ask agent to expand |

---

## House Rules

- **Templates only.** Never load SKILL.md or contract files into agent prompts. Use only `pipeline/prompt_templates/`.
- **Minimal context.** Each agent receives only the data relevant to its task. Per-scene agents get character/variable subsets, not full lists.
- **Isolated passes.** Each Agent call is an independent compiler pass with clean context. No agent remembers previous agents' work.
- **Artifact-driven.** All state lives in files (`shared/state/`, `public/game/`). Agents communicate only through disk artifacts.
- **Gate strictly.** No phase begins before previous phase outputs are verified on disk.
- **Orchestrator stays lean.** The orchestrator reads templates, fills placeholders, spawns agents, checks gates. It does not generate content.

---

## WebGAL Syntax Quick Reference

(For orchestrator use when assembling context — do NOT send this full reference to agents.)

```
; comment
setVar:name=value;                          ; variable assignment
setVar:name=value -when=condition;          ; conditional variable
choose:选项A:labelA|选项B:labelB;           ; choice branch
label:labelName;                            ; jump target
jumpLabel:labelName;                        ; intra-file jump
jumpLabel:labelName -when=condition;        ; conditional jump
callScene:filename.txt;                     ; call sub-scene (RETURNS after)
;                                           ; guard against fall-through
intro:Title|Subtitle;                       ; opening animation
end;                                        ; end game (main scene only)
Speaker:对话内容;                           ; character dialogue
:旁白内容;                                  ; narration
changeBg:image.webp -next;                  ; switch background
changeFigure:image.webp -left -next;        ; switch sprite
miniAvatar:avatar.webp;                     ; dialogue avatar
bgm:music.mp3 -volume=80;                   ; background music
```

Full reference (for human use): [syntax.md](~/.claude/skills/webgal-game/shared/constraints/syntax.md)
