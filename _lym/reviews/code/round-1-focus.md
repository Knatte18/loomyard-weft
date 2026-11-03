---
round: 1
focus:
  - "internal/fabricengine/remoteteardown.go and remove.go: does the detached child run the weft-tip push, then the archive tag (with the missing-objects retry), then the leased weft delete, in that order, and hold the teardown lock exclusive until the last step ends?"
  - "The lock handoff from the parent to the detached child (teardownlock.go, remoteteardown.go): can the lock be released early, leaked on a failed spawn, or left unheld between the parent's return and the child's start?"
  - "Async failure reporting: is each failure written to the fabric log and sent as one creel notice line naming slug, operation and way forward, and is a failure in an early step still reported when later steps are skipped?"
  - "taskbranchdiscard.go and cleanup.go: do DiscardTaskBranch and fabric cleanup's per-orphan archive and delete use the same detached child and order, and do fabric remove --remote's origin task-branch delete follow it?"
  - "Sync boundary against the Added-after-Discussion entry: fabric add's ls-remote probes run in parallel and stay sync, and no read-before-continue call (reconcile, fetches, landing pushes, ReadParentTips, GitHub PR calls, sync fabric push, clone, ResetPairCode, cleanup --remote-warp enumeration, add leftover handling, Bolt push) was made async."
  - "Local-only mode removal: is any reference left to --local, the local-only branch mark, its push refusal, skipped-local reporting, the local modes of remove and cleanup, or the local-mark check in the spawn status push, in code, docs (doc.go, SANDBOX-FABRIC-SUITE.md) and wiring?"
  - "remove_localmode_integration_test.go: its purpose after the local mode is gone, and whether it duplicates coverage in the other remove tests."
  - "Test economy across the new remoteteardown_test.go and the reworked integration tests: is each new behavior covered once, and is the detached-child order and lock hold covered at all?"
  - "Commit messages of the review range: any Pattern-Departure trailer, and compliance with PATTERN-slug-remote-order."
---

Seed focus from the decision record and the diff stat.
The record's acceptance criteria are empty, so the amended board entry is the measuring stick.
