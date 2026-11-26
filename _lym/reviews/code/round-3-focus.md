---
round: 3
focus:
  - "In internal/driftengine/loosened.go, what is the range key of a stored loosened-check ruling built from, and does it change when a commit of the review range adds, modifies or deletes any file, including a file the range itself creates? Check the wiring in internal/driftengine/verify.go and internal/loomcli/driftwiring.go and the source method in internal/fabricengine, including which range it reads and how it behaves when there is no review base or the read fails."
  - "Does internal/driftengine/verify_test.go fail if the range key ignores files the range adds, both for an uncaused ruling whose dropped checks later land in a new test file and for a carried_by ruling whose carrier is an added file that later changes?"
  - "Do the statements of when a ruling is reused or judged again agree with the code: the rangeRulingKey and listFiles doc comments in internal/driftengine/loosened.go, the RangeKey comment in internal/driftengine/records.go, internal/driftengine/doc.go, docs/overview.md, the halt text and _lym/landing/summary.md?"
---

Each entry covers the range key's coverage of the review range, its test pins and the prose that describes it.
