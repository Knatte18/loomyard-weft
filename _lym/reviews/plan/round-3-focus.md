---
round: 3
focus:
  - "Cards that drop or rename a field of `driftengine`'s Config, Paths or Deps (cards 24, 25, 26 and any other): grep every `_test.go` file in the tree for each removed name, and settle whether every enclosing test member, fixture and fixture constructor that reads or sets it is listed under that card's Edit."
  - "`TestWire_DarnRecipe` in `internal/loomcli/wiring_darn_test.go` reads `c.env.Drift.BaselineTable(\"\")`: settle whether the card that drops `Deps.BaselineTable` lists the test under Edit, and whether any earlier card lists it only under Uses against the later-target rule."
  - "Card 9's `CaptureCurrent` lock wait: settle whether the interval is injectable as a field or parameter, so `TestCaptureCurrent`'s row for a held lock does not wait out a real interval, as `awaitCaptureEnd` in `wait.go` already takes its sleep."
  - "Cards 8, 10 and 19 name the `Drift-Expect` row in `PastRow` calls and in `Amend` and `Defer` admission: settle whether the row name is one named constant declared once in `driftengine`, beside `verifyRow`, and referenced everywhere."
  - "Card 15's `JudgeSnapshot` and `driftDeps` exempt set: `statusCommitPathspec` in `internal/loomcli/wiring.go` stages the reviews directory and the `_lym/loom` durable directory at every transition. Settle whether a loom commit alone, of content already uncommitted at the snapshot, can make a resumed Drift-Verify row Stuck, and whether `preflight.runStateExemptRels`'s friction archive siblings are exempt."
  - "Card 16's Drift-Verify step after the judge returns: settle what commits or removes `_lym/drift/judgement.yaml` on a Done and on a does-not-match Stuck, and whether `fabricengine.Clean` or the driver's hand-over check then reads the file as dirt."
  - "Card 22 and `docs/overview.md`: settle whether every sentence that states the darn recipe's row order or the drift verbs, not only the drift paragraph, is named for an edit."
---

Focus targets the open keys and the new round 2 findings.
Each entry names where to look and the question to settle, and none caps severity.
