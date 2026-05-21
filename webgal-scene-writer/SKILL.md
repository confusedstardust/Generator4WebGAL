---
name: webgal-scene-writer
description: |
  Write all WebGAL .txt scene files based on narrative planning artifacts
  and generated assets. Translates scene graph + branch map + ending matrix
  into complete, playable scripts with dialogue, choices, and transitions.

  This skill writes WebGAL syntax. It does NOT design narrative structure
  or generate assets.
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
---

# WebGAL VN Scene Writer

Converts narrative architecture into playable WebGAL scripts. Reads planning
artifacts, writes `.txt` files, and verifies all asset references resolve.

---

## Contract & Constraints

Read before starting:
- [Contract](~/.claude/skills/webgal-game/shared/contracts/scene-writer.md) — full I/O specification
- [Syntax](~/.claude/skills/webgal-game/shared/constraints/syntax.md) — complete syntax reference and critical rules
- [Limits](~/.claude/skills/webgal-game/shared/constraints/limits.md) — file size limits (30-300 lines per scene)
- [Naming](~/.claude/skills/webgal-game/shared/constraints/naming.md) — file naming conventions

---

## Writing Workflow

See [contract](~/.claude/skills/webgal-game/shared/contracts/scene-writer.md) for input requirements and output specification.

### 1. Load All Planning Artifacts

Read every planning JSON. Cross-reference to build a complete mental model:
- `characters.json` → who speaks, speech style, emotional arc
- `scene_graph.json` → which files to create, flow between them
- `branch_map.json` → where choices go, what each option sets
- `variables.json` → which variables to set/check
- `ending_matrix.json` → ending triggers and priority order

### 2. Write start.txt First

Entry point. Initialize all variables to 0, set intro, then callScene to first act.

```
; <Game Title>
; <Brief description>

setVar:respect=0;
setVar:empathy=0;
setVar:openness=0;
setVar:flag_a=0;
setVar:flag_b=0;

intro:Title|Subtitle;

label:game_start;
:narration opening...
callScene:act1_scene.txt;

label:game_end;
:Game Over text
end;
```

### 3. Write Scene Files in Scene Graph Order

For each scene in `scene_graph.json`, create the corresponding `.txt` file.

Standard scene structure:
```
; <Scene Name>
; Characters present: <list>

changeBg:bg_<scene>.webp -next;

; --- Opening ---
:旁白内容;

; --- Dialogue ---
miniAvatar:miniavatar_<name>.webp;
Speaker:dialogue text;

; --- Choice Point (if applicable) ---
choose:选项A:labelA|选项B:labelB|选项C:labelC;

label:labelA;
setVar:var_name=var_name+15;
:unique content for path A...
jumpLabel:converge_point;

label:labelB;
setVar:other_var=1;
:unique content for path B...
jumpLabel:converge_point;

label:labelC;
:unique content for path C...
jumpLabel:converge_point;

label:converge_point;
:common dialogue continues...
callScene:next_scene.txt;
;
```

### 4. Write Ending Files

One file per ending. Place ending branch logic in the climactic scene,
ordered by priority (highest first):

```
; Check most demanding conditions first
setVar:bestCheck=0;
setVar:bestCheck=bestCheck+1 -when=respect>=50;
setVar:bestCheck=bestCheck+1 -when=empathy>=50;
setVar:bestCheck=bestCheck+1 -when=openness>=50;
jumpLabel:ending_best_path -when=bestCheck>=3;

; Then specific action flags
jumpLabel:ending_redemption_path -when=practical_help==1;

; Then failure states
jumpLabel:ending_lonely_path -when=empathy<30;

; Default fallback
jumpLabel:ending_canon_path;
```

Each ending file must be ≥30 lines. Use `end;` in ending files (they are
terminal — no return expected).

---

## Writing Rules

### Dialogue Density

Each choice point should be preceded by 5-15 lines of dialogue/narration.
Don't rush from choice to choice.

### Inner Voice

Use parenthesized inner monologue to guide the player:
```
CharacterName:（inner thought）spoken dialogue.
```

### Cultural Authenticity

When adapting classical material:
- Quote the original text directly where appropriate
- Use the original as narration between dialogue exchanges
- Let characters react to and interpret the source text

### Emotional Beats

After every major revelation, include a quiet moment:
```
:没有人说话。只有冷月静静地映在江中。
```

### Avatar Rule (MANDATORY)

Every named character with a sprite MUST display their avatar:
- Before their first line in each scene
- After every speaker change
- After `changeFigure` calls

```
miniAvatar:miniavatar_wife.webp;
妻子:渊明……？真的是你！

miniAvatar:miniavatar_tao_yuanming.webp;
陶渊明:我……我辞官了。
```

Narration (`:text`) and unnamed speakers skip the avatar.

### Branching Pattern

Every `choose` → unique content per option → converge:
- 3-4 options per choice (never binary)
- Each option sets different variables
- Each option has unique dialogue before `jumpLabel:converge_point`
- After `callScene`, always place `;` on next line

---

## Boundaries

- Do NOT modify planning artifacts in `shared/state/`
- Do NOT generate or modify assets
- Reference ONLY assets that exist on disk — verify with `ls`/`Glob` first
- Do NOT validate — that's the Validator's job
- `end;` only in `start.txt` and ending files
