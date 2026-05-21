---
name: webgal-asset-planner
description: |
  Translate narrative planning artifacts into a concrete asset manifest,
  then generate all visual assets via the Python toolchain.

  Writes assets_manifest.json and runs:
  generate_assets.py → remove_bg.py → make_avatar.py

  Does NOT write scene files or modify planning artifacts.
allowed-tools:
  - Bash
  - Read
  - Write
  - Glob
  - Grep
---

# WebGAL VN Asset Planner

Translates narrative architecture into visual assets. Reads character/scene
definitions, writes the asset manifest, and drives the Python generation pipeline.

---

## Contract & Constraints

Read before starting:
- [Contract](~/.claude/skills/webgal-game/shared/contracts/asset-planner.md) — full I/O specification
- [Limits](~/.claude/skills/webgal-game/shared/constraints/limits.md) — asset dimensions and formats
- [Naming](~/.claude/skills/webgal-game/shared/constraints/naming.md) — file naming conventions

---

## Workflow

See [contract](~/.claude/skills/webgal-game/shared/contracts/asset-planner.md) for input requirements and output specification.

### Step 1: Write the Asset Manifest

Create `assets_manifest.json` at the project root.

Derive every entry from the planning JSONs:
- Each character → one `figure` entry + implied mini avatar
- Each scene → one `background` entry
- Special event scenes → one `cg` entry (optional)

See [contract](~/.claude/skills/webgal-game/shared/contracts/asset-planner.md) for the exact manifest JSON schema.

### Prompt Writing Guidelines

Every prompt MUST include:
- Art style keyword (e.g. "Chinese ink wash painting", "anime style", "soft watercolor")
- "No text, no watermark"
- Lighting and mood descriptors

Background prompts: describe time of day, setting, key scenery elements, atmosphere.

Character sprite prompts: describe face, age, clothing, expression, pose.
MUST include "clean plain white background" (required for background removal).
MUST include "full body visible".

CG prompts: describe the scene composition, character positions, key action,
emotional tone, lighting.

### Step 2: Generate Images

```bash
# Set API key (if not already set)
export ARK_API_KEY="ark-3e941fe9-5b17-4260-ad1f-84037db96e88-997a4"

# Generate all images
python ~/.claude/skills/webgal-game/script/generate_assets.py assets_manifest.json
```

Uses 火山引擎 ARK (豆包 Seedream) via OpenAI-compatible API. Downloads each
image to `public/game/background/` and `public/game/figure/`.

If an image fails, re-run the same command — it skips already-downloaded files.

### Step 3: Remove Backgrounds from Sprites

```bash
python ~/.claude/skills/webgal-game/script/remove_bg.py public/game/figure/figure_*.webp
```

Uses `rembg` with `u2netp` model. Overwrites originals with lossless WebP
(alpha channel preserved). Requires `pip install rembg pillow`.

### Step 4: Generate Mini Avatars

```bash
python ~/.claude/skills/webgal-game/script/make_avatar.py public/game/figure/figure_*.webp
```

Produces `miniavatar_*.webp` in `public/game/figure/`. Naming:
`figure_tao_yuanming.webp` → `miniavatar_tao_yuanming.webp`.

---

## Optional: Asset Repo Integration

If `C:\SelfCreated\MyProject\assets repo\` exists, use it to skip redundant
API calls:

```bash
python ~/.claude/skills/webgal-game/script/generate_assets.py assets_manifest.json \
    --repo "C:\SelfCreated\MyProject\assets repo"
```

The script auto-searches the repo by prompt text before calling the API.
New images are auto-saved to the repo for future reuse.

---

## Boundaries

- Do NOT write scene files
- Do NOT modify narrative planning artifacts
- Do NOT skip the Python scripts — every asset must be generated, not manually placed
- If any script fails, stop and report the error. Do NOT attempt manual workarounds.
- Reference ONLY assets that were successfully generated — never invent filenames
