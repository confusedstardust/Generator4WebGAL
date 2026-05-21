---
name: webgal-orchestrator
description: |
  Pipeline orchestrator for webgal-game. Coordinates the 5-phase game
  building pipeline: Narrative Designer → Asset Planner → Scene Writer →
  Validator → Repair Agent. Enforces gate conditions between phases and
  handles failures. Each phase runs sequentially in the same context —
  all script writing, asset generation, and validation happen inline.
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - TodoWrite
  - AskUserQuestion
  - Agent
  - Skill
---

# WebGAL Orchestrator

Pipeline coordinator for building WebGAL visual novels. Drives the 5-phase
pipeline sequentially, enforcing phase ordering, verifying gate conditions,
and handling failures.

---

## Shared Resources (load before starting)

| Directory | Purpose |
|-----------|---------|
| [constraints/](~/.claude/skills/webgal-game/shared/constraints/) | Hard rules enforced by all agents |
| [contracts/](~/.claude/skills/webgal-game/shared/contracts/) | Sub-agent input/output contracts |
| [state/](~/.claude/skills/webgal-game/shared/state/) | Runtime pipeline artifacts |

### Constraint Files

- [syntax.md](~/.claude/skills/webgal-game/shared/constraints/syntax.md) — Complete syntax reference, critical rules, pitfalls table
- [naming.md](~/.claude/skills/webgal-game/shared/constraints/naming.md) — File/variable/asset naming conventions
- [limits.md](~/.claude/skills/webgal-game/shared/constraints/limits.md) — Hard numerical limits
- [config.md](~/.claude/skills/webgal-game/shared/constraints/config.md) — Config.txt mandatory field rules

---

## Pipeline

Five phases, strict order. Each phase MUST complete before the next begins.

```
Phase 1: Narrative Designer   → shared/state/{characters,variables,scene_graph,branch_map,ending_matrix}.json
Phase 2: Asset Planner        → assets_manifest.json + generate_assets.py + remove_bg.py + make_avatar.py
Phase 3: Scene Writer         → public/game/scene/*.txt
Phase 4: Validator            → shared/state/validation_report.json
Phase 5: Repair (conditional) → fix errors, re-run Phase 4 (max 3 cycles)
```

### Phase Invocation

| Phase | Skill | Produces |
|-------|-------|----------|
| 1 | `webgal-narrative-designer` | 5 planning JSONs → `shared/state/` |
| 2 | `webgal-asset-planner` | Manifest + generated images |
| 3 | `webgal-scene-writer` | All `.txt` scene files |
| 4 | `webgal-validator` | `validation_report.json` |
| 5 | `webgal-repair-agent` | Fixed `.txt` + `repair_log.json` |

Invoke each phase's skill via the `Skill` tool using the exact skill name above.
Each skill reads its contract from `~/.claude/skills/webgal-game/shared/contracts/<skill-name>.md` before executing.

### Gate Conditions

| Transition | Check (use Bash: test -f / ls) |
|------------|------|
| 1 → 2 | All 5 JSONs exist in `shared/state/` |
| 2 → 3 | All assets referenced in manifest exist in `public/game/background/` + `public/game/figure/` |
| 3 → 4 | All `.txt` files from `scene_graph.json` exist in `public/game/scene/` **AND** `config.txt` Game_name/Title_img/Game_Logo are set and resolve to existing assets |
| 4 → 5 | `validation_report.json` has `errors > 0`. If `errors == 0`, game is complete. |
| After 5 | Re-run Phase 4. Max 3 repair cycles total. |

### Failure Handling

| Failure | Action |
|---------|--------|
| Any Phase fails | Log error, stop pipeline, report to user |
| Validator finds errors | Invoke Repair Agent, re-validate |
| Repair cycle exceeds 3 | Stop and ask user for guidance |
| Asset generation fails | Report which asset failed; user fixes API key or prompt |

---

## WebGAL Syntax Quick Reference

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
miniAvatar:avatar.webp;                     ; dialogue avatar (before speaker's first line)
bgm:music.mp3 -volume=80;                   ; background music
```

Full rules: [syntax.md](~/.claude/skills/webgal-game/shared/constraints/syntax.md)

---

## House Rules

- **Phase 2 is non-skippable.** Asset files must exist on disk before any `.txt` script is written.
- **Scripts reference only existing assets.** Never invent filenames — verify with `ls`/`Glob` first.
- **All sub-agents read `shared/constraints/` before executing.** No agent re-encodes rules locally.
- **Orchestrator drives the pipeline.** It invokes each phase's skill sequentially, verifying gate conditions between phases.
- **Follow contracts exactly.** Each sub-agent's contract in `shared/contracts/` defines its complete interface.
