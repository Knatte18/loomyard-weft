---
round: 3
focus:
  - "internal/loomcli/decision.go `decisionVerb` takes the per-run decision lock and refuses with `decisionWayLock` when it cannot: settle whether a row of the add refusal table in decision_test.go sets the fake's `lockErr` and asserts the refusal's message, its way forward, and that nothing was appended or committed."
  - "internal/loomcli/decision.go `decisionReplaceVerb` refuses when `deps.writeRecord` fails, in two ways: a restore that succeeds, and a restore that fails too with `decisionWayRestoreRecords`: settle whether a test makes the first record write fail and reaches each refusal, asserting message, way forward, and that nothing was committed, stored or posted and the lock was released."
  - "internal/boardengine/runlock.go `ProjectRecordHeld` and internal/boardengine/doc.go: settle whether any doc comment still calls `lym loom decision add` the only way a beam project record changes, now that `lym loom decision replace` changes it too."
---

Round 2 left these three gaps open; each is a local correction.
