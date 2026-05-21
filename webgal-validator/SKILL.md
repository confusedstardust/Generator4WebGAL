---
name: webgal-validator
description: |
  Validate the complete WebGAL game against all constraints. Checks syntax,
  reference integrity, variable correctness, ending reachability, hard limits,
  and naming conventions. Produces a structured validation report.

  Reports ONLY — never patches. Fixing is the Repair Agent's job.
allowed-tools:
  - Read
  - Glob
  - Grep
  - Write
---

# WebGAL Validator

Validates the complete game and produces a structured report. Read-only —
never modifies scene files or planning artifacts.

---

## Contract & Constraints

Read before starting:
- [Contract](~/.claude/skills/webgal-game/shared/contracts/validator.md) — full check categories and report schema
- [Syntax](~/.claude/skills/webgal-game/shared/constraints/syntax.md) — critical syntax rules to verify
- [Limits](~/.claude/skills/webgal-game/shared/constraints/limits.md) — hard numerical limits
- [Naming](~/.claude/skills/webgal-game/shared/constraints/naming.md) — naming conventions

See [contract](~/.claude/skills/webgal-game/shared/contracts/validator.md) for prerequisites and report schema.

---

## Validation Checks (6 Categories)

### 1. Syntax

Check every `.txt` file for violations:

| Check | How to Verify |
|-------|---------------|
| `callScene` followed by `;` | After every `callScene:`, next non-empty line must be `;` or a label |
| `end;` only in start.txt + endings | Grep all files for `end;`, cross-check file list |
| `jumpLabel` targets exist | For each `jumpLabel:XXX`, grep same file for `label:XXX` |
| No multi-condition `-when=` | Grep for `-when=` containing AND/OR/&&/`\|\|` |
| Boolean values are 0/1 | Grep for `=true` or `=false` |
| `choose` options pipe-separated | Each `choose:` must have `\|` between options |

### 2. References

Cross-check every reference against what exists on disk:

| Check | How to Verify |
|-------|---------------|
| Background files exist | Grep `changeBg:(.*) ` → check file exists in `public/game/background/` |
| Figure files exist | Grep `changeFigure:(.*) ` → check file exists in `public/game/figure/` |
| Mini avatars exist | Grep `miniAvatar:(.*);` → check file exists in `public/game/figure/` |
| callScene targets exist | Grep `callScene:(.*);` → check file exists in `public/game/scene/` |
| Speaker names match characters | Extract all `^[A-Za-z一-鿿]+:` lines → cross-check with `characters.json` |

### 3. Variables

| Check | How to Verify |
|-------|---------------|
| All setVar names declared | Extract `setVar:(.*)=` → cross-check with `variables.json` |
| All -when= names declared | Extract `-when=(.*)[>=<]` → cross-check with `variables.json` |
| Attitude vars use numeric range | Check no `setVar:empathy=true` style assignments for attitude vars |
| Total variable count ≤ 12 | Count unique variable IDs in `variables.json` |

### 4. Endings

| Check | How to Verify |
|-------|---------------|
| Exactly 5 endings | Count entries in `ending_matrix.json` |
| Every ending reachable | Trace from `start.txt` through callScene/jumpLabel chain to each ending file |
| Every ending ≥ 30 lines | Check line count of each ending file |

### 5. Limits

| Check | How to Verify |
|-------|---------------|
| 3-7 characters | Count entries in `characters.json` |
| Branch depth ≤ 2 | Follow `choose` chains, count max nesting from root |
| Max 5 major branches | Count `choose:` statements across all files |
| 30-300 lines per scene | `wc -l` each `.txt` file |
| Min 5 choice points | Count `choose:` statements |
| 3-4 options per choice | Count `\|` separators in each `choose:`

### 6. Naming

| Check | How to Verify |
|-------|---------------|
| Scene files: `^[a-z][a-z0-9_]+\.txt$` | Check filenames in `public/game/scene/` |
| Background files: `^bg_` | Check filenames in `public/game/background/` |
| Figure files: `^figure_` | Check filenames in `public/game/figure/` |
| Mini avatar files: `^miniavatar_` | Check filenames in `public/game/figure/` |

---

## Boundaries

- Do NOT fix any issues — that's the Repair Agent's job
- Do NOT modify scene files or planning artifacts
- Report ONLY — never patch
