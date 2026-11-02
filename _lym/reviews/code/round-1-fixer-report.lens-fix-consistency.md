```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1–F5 change set for consistency with its surroundings (lens fix-consistency).

No findings.

- The `export_test.go` header now names the production timer, matching the new `StartDefaultStallTimerForTest`.
- The `defaultStallTimer` initializer depends on `startStallTimer`, so Go's package init order covers it.
- The `startStallTimer` comment in `gitexec.go` is still true: the default ignores `tracePath` and the seam exists for tests.
- The `SetStallCheckForTest` doc still describes `attempt` accurately. The silent row no longer reads it, and the answering row never did.
- The serial-state comment on `TestRun_RemoteBounds` now names the stall check seam. The new production-timer row sets no process-global state, so the comment needs no further change.
- The unarmed row's new comment fits its assertions, which are `Stalled == false`, one run and one kill. The silent row keeps its two-armed-checks assertion, as F2 asked.
- The answering row's `checkRan` assertion matches its comment that the check runs once git has traced the packet.

Calls made: none, because my context already held every changed file and the review.
