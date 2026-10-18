---
round: 1
focus:
  - "internal/loomshed/webster.go and loompreflight.go: does Webster resolve `batcher.Active` and partition the plan once per run exactly as the deleted `Batchifier` row did, and does the fail-fast on an unresolvable `batcher.yaml` still happen before any LLM row?"
  - "internal/shedrecipe/entries_implementer.go and registry.go: does the `Implementer` engine refuse a missing key, unknown, empty, null and non-string `loom.implementer` values at recipe build with a way forward, and does every other row Config key reach the constructor unchanged?"
  - "Removal of `Batchifier` across loomrecipe, loomshed, shedbuild and loomengine: is any reference left behind (interrupt policy, gate lists, resume or status-pane tables, docs, loom-recipe.yaml), and does `Plan-Bouncer` `on_done` name `Webster`?"
  - "internal/reportengine/timingspans.go: does a past run's `Batchifier` history row fold into the webster phase without double-counting, and is the retired row name confined to the phase mapping?"
  - "loom.yaml and loomengine config and template: is `implementer: webster` registered with category none, so local layer and `run_settings` can set it, matching the board entry?"
  - "contracts/specs/implementer-spec.md and the pointers in webster-spec.md, docs/overview.md: does the contract state in, out and outcomes as the board entry asks, and do docs and PATTERN land in the same diff as the repository's CLAUDE.md requires?"
  - "creelengine/deliver_test.go: the change is outside the board entry's scope; check whether the summary's claim about the flaky test matches the diff, and whether it is licensed."
  - "Test economy: do the new tests in entries_implementer_test.go and webster_test.go duplicate existing coverage, and is any behavior the diff adds left uncovered?"
---

Seed focus from the stat and summary only; the diff was not read line by line.
