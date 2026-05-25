---
name: webgal-orchestrator
description: |
  Orchestrates the multi-phase WebGAL game generation pipeline,coordinating narrative planning, asset preparation, scene compilation,validation, and repair
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

## Shared Resources (load before starting)

| Directory | Purpose |
|-----------|---------|
| [constraints/](~/.claude/skills/webgal-game/shared/constraints/) | Hard rules |
| [contracts/](~/.claude/skills/webgal-game/shared/contracts/) | I/O contracts |

### Constraint Files

- [syntax.md](~/.claude/skills/webgal-game/shared/constraints/syntax.md)
- [naming.md](~/.claude/skills/webgal-game/shared/constraints/naming.md)
- [limits.md](~/.claude/skills/webgal-game/shared/constraints/limits.md)
- [config.md](~/.claude/skills/webgal-game/shared/constraints/config.md)

---

### Phase Invocation

| Phase | Skill |
|-------|-------|
| 1.Narrative Designer | `webgal-narrative-designer` |
| 2.Asset Planner | `webgal-asset-planner` |
| 3.Scene Writer | `webgal-scene-writer` |
| 3.5.BGM Injection | Run `python inject_bgm.py <project_root>` |
| 4.Validator | `webgal-validator` |
| 5.Repair (conditional) | `webgal-repair-agent` |

Invoke each phase's skill via the `Skill` tool using the exact skill name above.
Each skill reads its contract from `~/.claude/skills/webgal-game/shared/contracts/<skill-name>.md` before executing.

---

## House Rules

- Pipeline execution is strictly sequential.
- Phase transition requires explicit gate validation success.
- Any deviation from contract = phase failure.
- All validation logic is derived from contracts and gate conditions only.

## Execution

Delegate immediately to:

```
Skill: webgal-narrative-designer
```

