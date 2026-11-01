---
round: 2
focus:
  - "internal/driftengine/config.go: drift's ConfigSpec declares capture and compare as open maps decoding into stepFile. Settle whether `--set capture.env.X=1` and `--set capture.timeout_min=30` now land inside the step entry with the decoded type, and whether a misspelled key under capture is refused naming it."
  - "internal/yamlengine/set.go setShapedValue: settle whether a one-segment key under a shaped open map set to a flow mapping (the documented `--set tiers.tierN={...}` form) lands whole, and whether a bracketed value on a single-value list field lands as a flow list, each with a test row in TestSetValues and TestSetSparse."
  - "internal/yamlengine/set.go ShapeOf: settle whether its documented rules (an untagged exported field keyed by its lower-cased name, a tag option stripped from the key, an unexported field left out) are each pinned by a test."
  - "internal/yamlengine/set.go SetValues: settle whether a call with a valid shaped pair followed by a misspelled shaped pair returns the error and no merged bytes, and whether a test pins that all-or-nothing behavior."
---

Round 2 re-checks the four round 1 findings and any new text the fixes add.
