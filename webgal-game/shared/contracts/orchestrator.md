# Orchestrator Contract

Root entry point. Invoked when user triggers `/webgal-game`. Does NOT contain syntax reference or writing guides — those live in `~/.claude/skills/webgal-game/shared/constraints/`.

---

## Reads

| Source | Path | Description |
|--------|------|-------------|
| User input | (conversation) | Source material: poem, story, concept, historical event, etc. |
| Constraints | `~/.claude/skills/webgal-game/shared/constraints/*.md` | Hard limits, syntax rules, naming conventions |
| Contracts | `~/.claude/skills/webgal-game/shared/contracts/*.md` | Interface contracts for each sub-agent |

---

## Pipeline (Strict Order)

```
Phase 1: Narrative Designer   → shared/state/{characters,variables,scene_graph,branch_map,ending_matrix}.json
Phase 2: Asset Planner        → assets_manifest.json + run generate_assets.py + remove_bg.py + make_avatar.py
Phase 3: Scene Writer         → public/game/scene/*.txt
Phase 4: Validator            → shared/state/validation_report.json
Phase 5: Repair (conditional) → fix violations, re-run Validator
```

Each phase MUST complete before the next begins. Before starting Phase N, verify all outputs from Phase N-1 exist on disk.

---

## Gate Conditions

| Transition | Check |
|------------|-------|
| Phase 1 → 2 | All 5 planning JSONs exist in `shared/state/` |
| Phase 2 → 3 | All assets referenced in manifest exist in `public/game/background/` and `public/game/figure/` |
| Phase 3 → 4 | All .txt files from scene_graph exist in `public/game/scene/` |
| Phase 4 → 5 | If `validation_report.json` has `errors > 0`, invoke Repair Agent. If `warnings > 0`, flag to user. If clean, game is complete. |

---

## Produces

Nothing directly. The orchestrator delegates all artifact creation to sub-agents. Its output is a complete, validated WebGAL game ready to run via `npm run dev`.

---

## Success Criteria

- All gate conditions pass
- `npm run dev` starts without errors
- Every ending is reachable via at least one valid choice path
- Every asset referenced in scene files exists on disk

---

## Failure Handling

| Failure | Action |
|---------|--------|
| Any Phase fails | Log error, stop pipeline, report to user |
| Validator finds errors | Invoke Repair Agent, re-validate. Max 3 repair cycles. |
| Repair cycle exceeds 3 | Stop and ask user for guidance |
| Asset generation fails | Stop and report which asset failed, let user fix API key or prompt |
