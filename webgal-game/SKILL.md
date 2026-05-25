---
name: webgal-game
version: 2.0.0
description: |
  Generate structured WebGAL game from narrative inputs.
triggers:
  - make a WebGAL game
  - convert story to WebGAL
  - create a branching visual novel
  - generate an interactive fiction game
  - adapt narrative into a game
allowed-tools:
  - Read
  - Skill
---

## Execution

Delegate immediately to:

```
Skill: webgal-orchestrator
```

The orchestrator runs a 5-phase pipeline:

```
Phase 1: Narrative Designer
Phase 2: Asset Planner
Phase 3: Scene Writer
Phase 4: Validator
Phase 5: Repair (conditional)
```
