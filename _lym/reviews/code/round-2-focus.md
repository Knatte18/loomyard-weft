---
round: 2
focus:
  - "internal/websterengine/recordbatch.go, createTargetWayForward and failCardNotDone: when a batch has reached its maximum counted recoveries, does the create-not-done way forward avoid ending in a recover-batch that RecoverSpawnOrAttach refuses with ErrRecoveryExhausted, given Rebaseline records no amendment for a terminal batch (rebaseline.go)?"
  - "internal/websterengine/recordbatch_test.go: does a test reach the signature-mismatch way forward, the cardNotDoneFor signature-mismatch flag, the uncheckable arm of createTargetWayForward, the exhausted-batch case, and the precedence between the delete, glyph and create way forwards?"
  - "internal/planglyph/donecheck.go signatureMismatches and signatureMismatchFinding: are the guards for a target without a parameter list, another owner, another unit and a Go plan pinned by negative rows, and is the Rename new side case pinned and worded correctly in the way forward?"
  - "_lym/landing/summary.md, section on the way forward for a missing Create target: does every claim about amendments and recoveries match what Rebaseline and RecoverSpawnOrAttach do for in-flight and terminal batches?"
---
