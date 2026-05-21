# Phase 3: Scene Writer — Per-Scene Runtime Prompt Template

Orchestrator fills `{{PLACEHOLDERS}}` per scene, sends each as a separate Agent call.

---

TASK: Write scene `{{SCENE_ID}}` → `{{FILE_PATH}}`

PROJECT DIR: {{PROJECT_DIR}}

## Available Assets (use exact filenames)
{{AVAILABLE_ASSETS}}

## Characters Present
{{CHARACTERS_SECTION}}

## Variables in This Scene
{{VARIABLES_SECTION}}

## Upstream Context
{{UPSTREAM_SUMMARY}}

## Transitions
{{TRANSITIONS_SECTION}}

## WebGAL Syntax Rules (minimal)
1. Open scene: `changeBg:xxx.webp -next;` then `changeFigure:none -left -next;` and `changeFigure:none -right -next;`
2. Before any speaker's first line: `changeFigure:xxx.webp -left -next;` (or -right) then `miniAvatar:xxx.webp;`
3. Dialogue: `SpeakerName:对话内容;` — name must match characters exactly
4. Choices: `choose:选项A:labelA|选项B:labelB;` then labels below, converge with `jumpLabel:merge;`
5. Scene transition: `callScene:next.txt;` followed by `;` on its own line to prevent fall-through
6. Close scene before transition: `changeFigure:none -left -next;` and `changeFigure:none -right -next;`
7. Boolean values: 0/1 (never true/false). `setVar:flag=1;` / `jumpLabel:target -when=flag==1;`
8. Scene length: 30-300 lines

## Hard Rules
- `end;` NEVER in this file (only in start.txt and ending files)
- Every `callScene` MUST be followed by `;` on next line
- `-when=` supports single condition only
- `jumpLabel` is file-local only

## Output
1. Write the scene content to `{{FILE_PATH}}`
2. After writing, respond with exactly this line (nothing else before it):
   `SUMMARY: <1-2 sentence summary of what happens in this scene>`
