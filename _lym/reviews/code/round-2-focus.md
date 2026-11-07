---
round: 2
focus:
  - "In `internal/driftengine/check.go`, `unchangeableFindings` builds the Terminal drift-gate failure text: does it name a records fault and a seal fault found beside unchangeable cells, as the `NewDriftGate` doc comment and `doc.go` promise, and does a test cover it?"
  - "In `internal/driftengine/capture.go`, `backUpBaseline` moves the baseline and its `.meta` sibling aside: can any leftover `baseline.N.meta` or stale `baseline.meta` leave the pair half-moved or mismatched?"
---

The focus covers the two design-class sites; the settled remainder needs no re-review.
