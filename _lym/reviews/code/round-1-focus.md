---
round: 1
focus:
  - "contracts/recipes/sampler-recipe.yaml: do the added Drift rows match the rows tapestry and darn carry (stages, order, gates, ids), and do they run only when drift.scope is full, with scope off leaving the sampler's run unchanged?"
  - "internal/driftengine/verify.go and internal/driftcli/{cli,wiring}.go: what behavior change allows drift verbs in a sampler worktree, and does a run with drift rows off, or a tapestry or darn run, still behave as before?"
  - "internal/loomcli/wiring.go and internal/driftengine/template.yaml: is the sampler's default drift.scope off, using the existing key and no new run_settings key?"
  - "The record says tapestry and darn stage gates stay as they are: does any hunk in loomrecipe, loomcli or driftengine change those gates?"
  - "Tests (drift_test.go, samplerrecipe_test.go, wiring_darn_test.go, verify_test.go): does each new behavior have coverage, and does any test duplicate an existing one?"
  - "docs/overview.md and the doc.go files: do the doc changes match the code, and does the summary claim anything the diff does not bear out?"
---

Seed focus from the board entry's Goal alone; the record's other sections are empty.
