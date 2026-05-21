# Pipeline State Directory

Runtime artifacts created by the WebGAL game building pipeline.

## Contents (all generated, not committed)

| File | Created By | Phase |
|------|-----------|-------|
| `characters.json` | Narrative Designer | 1 |
| `variables.json` | Narrative Designer | 1 |
| `scene_graph.json` | Narrative Designer | 1 |
| `branch_map.json` | Narrative Designer | 1 |
| `ending_matrix.json` | Narrative Designer | 1 |
| `validation_report.json` | Validator | 4 |
| `repair_log.json` | Repair Agent | 5 |

## Notes

- This directory is for **runtime artifacts only** — do not commit contents to git
- Downstream agents read their inputs from here
- See `shared/schemas/` for the JSON Schema of each artifact
- See `shared/examples/` for completed examples from a successful build
