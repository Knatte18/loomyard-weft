---
round: 1
focus:
  - "internal/driftengine/baseline.go and baseline_test.go: does Drift-Baseline still return Done without waiting on a held capture lock, return once the detached child's record exists, and keep the Windows breakaway fallback through RunCapture; is any code only WaitCapture reached left behind, or any code still needed removed?"
  - "internal/driftengine/records.go, verify.go, verbs.go and their tests: does CountedCapture count a passed record only with awaited_at set, and do waive and Drift-Verify judge by awaited_at on every recipe; is any AwaitRow reference left in Deps, VerbStatus or the wiring?"
  - "internal/shedrecipe/entries_drift.go: does the DriftBaseline entry refuse a capture key with a message that names the way forward, and does its doc drop the key?"
  - "contracts/recipes/darn-recipe.yaml and the internal/loomrecipe tests: does the row order read Preflight, Drift-Baseline, Drift-Await, Darn-Write with entry_gate drift.enabled on Drift-Await, are the edges and row indexes correct, and does the header comment say a parked run is settled by hand?"
  - "internal/loomcli and internal/driftcli wiring, plus reportengine timing changes: is the await-row flag no longer computed or passed, and is the reportengine change in scope for the answer key?"
  - "Docs in the same diff: driftengine, shedrecipe, driftcli and loomrecipe doc.go and the darn paragraph of docs/overview.md; do they match the code, with no remaining mention of the wait capture mode?"
  - "Test economy: do the changed tests drop the removed modes' rows rather than keep dead coverage, and does any new test duplicate an existing one?"
---

Seed focus drawn from the board entry's Change bullets and the decision record's scope and acceptance criteria.
