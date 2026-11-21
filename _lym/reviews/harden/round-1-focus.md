---
round: 1
focus:
  - "One-repository hub layout (57015a52): drive clone, seed push and one-URL join against a sandbox hub made by `lym harden sandbox new`. Does each leave records and board under origin's `refs/lym/` with the expected refspecs, and does a second clone see the same state?"
  - "Refspecs and archive tags under `refs/lym/`: after add, bolt, checkout, cleanup and archive, do fetch and push refspecs stay correct, and do archive tags land on the right refs in the one-repository layout?"
  - "Parity with the two-repository layout: run the same drive on both layouts. Where do they diverge in `clone.go`, `add.go`, `bolt.go`, `cleanup.go` or `archive.go` behaviour that is not intended?"
  - "Failure and rollback paths: interrupt or refuse a seed push, a join, an add and a bolt on a live hub. Does the hub end with no half-written refs, leftover remotes or task paths?"
  - "Concurrency: two hubs running the same add or bolt against one origin. Prove any claim with a sequential control and a second hub beside the first."
  - "Hardener recipe itself, as the first real run: note any defect in `lym harden sandbox new`, the facts file, the live gate or the stray reaping. Report each in the review file as a recipe finding."
  - "Live gate coverage: `harden-live` tiers 0 and 1, `harden-live-copies` tier 1 and `harden-burler-fix` tiers 0 and 1. Which fabricengine behaviours do these tests leave undriven, and does driving them by hand show a defect?"
---

Round 1 has no prior findings, so the focus follows the decision record's goal and scope.
Fixes stay inside `internal/fabricengine` and its tests; a defect elsewhere is deferred as a board note.
