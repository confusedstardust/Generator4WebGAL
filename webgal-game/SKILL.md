---
name: webgal-game
version: 2.0.0
description: |
  Design and build multi-branch, multi-character, multi-ending immersive
  visual novel games for the WebGAL engine. Takes any source material
  (poem, story, novel, concept, historical event) and transforms it into
  a complete WebGAL game with branching narrative, variable tracking,
  and multiple endings.
triggers:
  - make a WebGAL game
  - create a visual novel
  - build a branching story game
  - turn this story into a game
  - make an interactive narrative
  - WebGAL game from
allowed-tools:
  - Bash
  - Read
  - Skill
---

# WebGAL Game Builder

Build complete multi-branch, multi-character, multi-ending WebGAL visual novels.

## Quick Start

```bash
npm run setup    # Install dependencies (first time only)
npm run dev      # Start dev server → http://localhost:3000
```

## Project Structure

```
.
├── src/                          # WebGAL engine source (rarely modified)
├── public/
│   └── game/
│       ├── config.txt            # Game config (title, cover, etc.)
│       ├── scene/                # Script files (your core workspace)
│       │   └── start.txt         # Entry point
│       ├── background/           # Backgrounds (2560×1440 WebP)
│       ├── figure/               # Character sprites (1440×2560 WebP, transparent)
│       ├── bgm/                  # Background music (mp3)
│       ├── template/             # Engine UI templates
│       └── tex/                  # Effect textures
├── package.json
└── vite.config.ts
```

## Instructions

When this skill is triggered, immediately delegate to the orchestrator:

```
Skill: webgal-orchestrator
```

The orchestrator runs a 5-phase pipeline:

```
Phase 1: Narrative Designer   → story planning artifacts
Phase 2: Asset Planner        → image generation + processing
Phase 3: Scene Writer         → .txt script files
Phase 4: Validator            → validation report
Phase 5: Repair (conditional) → fix errors, re-validate
```

Full pipeline documentation: [webgal-orchestrator/SKILL.md](webgal-orchestrator/SKILL.md)
Shared rules and contracts: [shared/](shared/)
