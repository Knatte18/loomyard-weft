---
round: 2
focus:
  - "internal/fabricengine/destroy.go, function DropLocal and its helpers untrackedOnPaths and ownedUntrackedTrackedAt: does the diff's test set show that an untracked ignored file, a file origin's tip does not track, a file on the code branch, and a path the index tracks each survive or are refused, while a file tracked at origin is removed?"
  - "internal/adoptcli/deps.go, the Reconcile seam built by hubDepsAs and its reconcilePushLockWait constant: does a test fail when the seam stops passing PushLockWait to the reconcile?"
  - "internal/adoptcli/doc.go Step done section and the Resume bullet, and docs/overview.md adopt entry: do they agree with the worker's behaviour that a record stopped at done restarts at reconcile with steps cleared, that an adopt passes every pause, and that its reconcile waits up to the push-lock bound?"
  - "internal/fabricengine/originreconcilemoves_integration_test.go TestReconcileOrigin_Moves and internal/adoptcli/worker_test.go TestRunWorker: is a held write lock under a nonzero PushLockWait reported busy at once, and is a re-judged stop that still holds shown reaching the operator with the kept paused slugs?"
---

Each entry names a site and a question; the changed code is whatever the committed diff over the review range shows.
