# Phase 1: Narrative Designer — Runtime Prompt Template

Orchestrator fills `{{PLACEHOLDERS}}` then sends as Agent prompt.

---

TASK: Design complete narrative architecture for a WebGAL visual novel.

SOURCE MATERIAL:
{{SOURCE_MATERIAL}}

STATE DIRECTORY: {{STATE_DIR}}

---

## Output Files (write ALL 5 to {{STATE_DIR}}/)

### 1. characters.json
```json
{"characters": [{"id":"snake_case","name":"显示名","role":"protagonist|supporting|authority|rival|family","personality":"1句","motivation":"1句","speech_style":"1句","emotional_arc":"1句"}]}
```
{{MIN_CHARS}}-{{MAX_CHARS}} characters. Distinct roles.

### 2. variables.json
```json
{"attitude_variables":[{"id":"snake_case","range":[0,100],"description":"1句"}],"event_flags":[{"id":"snake_case","values":[0,1],"description":"1句"}]}
```
Max 12 total. Booleans use 0/1, never true/false.

### 3. scene_graph.json
```json
{"scenes":[{"id":"snake_case","file":"xx.txt","description":"1句","character_ids":["char1"]}],"connections":[{"from":"s1","to":"s2","condition":null}]}
```
{{MIN_SCENES}}-{{MAX_SCENES}} scenes. Connected from start to all endings.

### 4. branch_map.json
```json
{"branches":[{"id":"choice_xxx","scene":"parent_scene_id","options":[{"label":"选项文本","sets":{"flag":1},"next_scene":"target_id"}]}]}
```
Min {{MIN_BRANCHES}} choice points. Max depth {{MAX_DEPTH}}. 3-4 options per choice.

### 5. ending_matrix.json
```json
{"endings":[{"id":"ending_xxx","category":"canon|bittersweet|recluse|home|between","emotional_tone":"2字描述","trigger":{"type":"variable|fallback","condition":"flag=1"},"narrative_meaning":"1句"}]}
```
Exactly {{ENDING_COUNT}} endings. Every ending must be reachable via at least one valid path from start.

---

## Hard Limits
- Characters: {{MIN_CHARS}}-{{MAX_CHARS}}
- Scenes: {{MIN_SCENES}}-{{MAX_SCENES}}, each 30-300 lines (when written in Phase 3)
- Variables: max 12 global
- Choices: min {{MIN_BRANCHES}}, max depth {{MAX_DEPTH}}, 3-4 options each
- Endings: exactly {{ENDING_COUNT}}

## Boundaries
- Do NOT write .txt scene files
- Do NOT generate image prompts or asset descriptions
- Do NOT produce WebGAL syntax (choose/jumpLabel/callScene/etc.)
- This is narrative architecture ONLY

## Output
Write all 5 JSON files. Then report: character count, scene count, branch count, and ending IDs.
