---
round: 2
focus:
  - "internal/weaverengine/run.go runBatch: when a strand stop fails after an attempt, does the recorded stop kind and rung match what the resume path in resume.go would choose for the same outcome, including a needs-parent outcome?"
  - "internal/weaverengine/reset.go Reset: does the strand probe run before the foreign-path check, and are a stopped attempt's uncommitted tracked edits treated as the run's own?"
  - "internal/weaverengine/reset.go Reset: is the run lock held from the probe through the archive, and released on every return path?"
  - "internal/weaverengine/run_test.go: do rows cover a rename that a pending card names, only the true last batch built as last, and a state-save failure after a recorded outcome?"
  - "internal/weaverengine/run.go stopReason and resume.go judgeOpenBatch: does truncation keep a stop error's way forward, and does a fault outcome keep its rung?"
---

The focus lists where the fixes of round 1 land; the reviewer still reads the whole range.
