---
round: 2
focus:
  - "internal/loomcli/driftwiring.go: do tests in internal/loomcli pin the request fields the Drift Deps pass in CompareAccepted and CaptureAccepted: the compare command's {baseline_dir} rendered as Paths.AcceptedDir, and the skip list holding accepted-skip tests but not accepted tests, against Compare, which lists both? Settle whether swapping the skip list or dropping the profile dir would fail a test."
  - "internal/driftengine/recheck.go lapse: when a recheck finds drift, which acceptances lapse, and does the Drift-Recheck paragraph in internal/driftengine/doc.go state that cost, including the parent rulings under scope full? Settle whether code and doc agree."
  - "internal/driftengine/capture_test.go: does TestRunCapture cover the timeout branch of RunAcceptedCapture as it covers the one of RunCapture?"
  - "Docs and comments naming Drift rows: internal/reportengine/doc.go phase lists against the loomPhases and darnPhases maps in timingspans.go, the enabled comment in internal/driftengine/template.yaml against the rows carrying entry_gate drift.enabled, and the header comment of contracts/recipes/tapestry-recipe.yaml against the perch's on_done. Settle whether each names Drift-Recheck where it applies."
---

Entries name where to look; the reviewer judges the committed diff as usual.
