---
round: 1
ledger:
  - key: records-push-transient-not-owed
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: records-push-retry-failure-paths-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: syncoptions-backoffsleep-doc-narrow
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 is the first round, so there is no earlier ledger to carry forward.

- `records-push-transient-not-owed` (F1): a records side still unpushed after the retry backoff, and a transport failure of the post-push origin lookup, are not classified as owed pushes.
- `records-push-retry-failure-paths-untested` (F2): the spent backoff and failed lookup have no test, and the new subtest does not count its backoff sleeps.
- `syncoptions-backoffsleep-doc-narrow` (F3): the `SyncOptions.BackoffSleep` doc lists only some of the field's uses.
