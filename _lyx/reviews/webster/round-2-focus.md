---
round: 2
focus:
  - "Go comments that the branch's diff adds or changes in internal/shuttleengine (run.go, doc.go, claudeengine/skillload.go), internal/orchengine (doc.go, watcher.go, watcher_test.go) and internal/loomcli/integration_driverbootstrap_test.go: does each break at every independent-clause boundary (semicolon or comma before a coordinating conjunction with its own subject and verb) and avoid mid-phrase fixed-column wraps, as PATTERN-comment-line-breaks requires?"
  - "Tests the diff adds or rewrites in internal/orchengine/watcher_test.go that reload skills (the retry-step restart test and the old-style-step test): does each call t.Parallel or carry a comment naming the process-global state it touches, as PATTERN-test-speed requires?"
  - "Watcher.reloadStep in internal/orchengine/watcher.go: a persisted retry step with an empty reload_retry list is read as the pointer step; does a test reach that branch, as PATTERN-test-economy requires for added behavior?"
  - "The orch warn vocabulary for skill loading (a 'skill skipped' warn per skill carrying skill and cause, and one 'skill load unverified' warn listing the skills): does a test in internal/orchengine assert the logged output for each cause, so an implementation that logs nothing fails?"
  - "Assignments to State.ReloadStep in internal/orchengine (state.go ResetForFreshLaunch, watcher.go toIdle and startReload): do they use the named ReloadStep constants consistently rather than a literal 0?"
---

Each entry names a place to look and a question to settle; none states how the answer should come out.
