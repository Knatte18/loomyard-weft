---
round: 2
focus:
  - "internal/driftengine/compare.go RunCompare and internal/driftengine/spawn.go Active: does a compare started while a reader's probe holds the compare marker still run, with the already-running refusal only when the marker stays held past a bounded retry, as RunCapture retries its take?"
  - "internal/driftcli/cli.go capture child body: is the HEAD stored in the capture record the one read while the capture lock is held, the same HEAD Check verified?"
  - "internal/pairteardown/cancel.go: does the cancel path end the pair's drift capture job the way EndSession does, or is the omission documented?"
  - "internal/driftengine/records.go Ruling const group and Acceptance.Ruling comments: do they state the scope under which Accept sets a ruling and who made the acceptance?"
---

Focus targets the sites round 1 changed, so a reviewer checks the fixes with fresh eyes.
