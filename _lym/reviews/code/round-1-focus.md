---
round: 1
focus:
  - "internal/adoptcli/caller.go, worker.go, deps.go: does the pause exemption cover every network call of the adopt and its worker, and only those, so no other command gains the exemption?"
  - "internal/fabricengine/originreconcile.go, originreconcilemoves.go: does PushLockWait wait only on a held push lock of a records branch, stay bounded, and fail the same busy stop when the lock outlasts it?"
  - "internal/fabricengine/destroy.go: does the new ownedUntrackedTrackedAt gate delete only untracked files origin's tip tracks and lym owns, after the archive tag exists, and never a file the operator authored or one origin does not track?"
  - "internal/adoptcli/caller.go: when a record is stopped at done and re-run, are steps cleared and the reconcile restarted correctly, and does a stop that still holds still reach the operator with the `lym adopt --discard` way forward?"
  - "Tests in caller_test.go, worker_test.go, droplocal_integration_test.go, originreconcilemoves_integration_test.go: does each of the three faults have coverage of its failure path, and does any new test duplicate another?"
---

Seeded from the decision record's three notes: pause exemption with push-lock wait, drop-local clearing untracked weft files, and re-judging a held-back stop.
