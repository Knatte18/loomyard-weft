---
round: 1
focus:
  - "internal/gitexec, internal/termwindow, internal/verifyrun, internal/gatecli tests: does any wait end up with a shorter bound than before, rather than an injected clock or an event wait? Check each wait's git history, especially bounds covering a process or binary start, which must keep their length."
  - "Each reworked test (gitexec TestRun_RemoteBounds, gatecli detached run, termwindow poll row, verifyrun expiries): does it still pin what it pinned before, with no assertion dropped or weakened?"
  - "Production seams added for tests (gitexec.go, termwindow.go, gatecli poll-period and stand-in file injection, verifyrun child-signal cancel): do the defaults leave production behavior unchanged, and are failure paths, races and leaked goroutines or files handled?"
  - "internal/impactset watches.go, targets.go, guards.go and internal/gatecli scope.go, tiers.go: is the //lym:watch rule correct, so TestCrossCompileLinux and the dotnet fixture test run in the whole pass but in a card or scoped gate only when a file they guard changes? Look at path matching, _linux.go files, and an empty or malformed watch line."
  - "internal/testkit/lymbin: does TestBuildInto_UnwritableDirReturnsError now fail before building, and does the early check keep the error contract for other callers?"
  - "PATTERN.md and pattern/PATTERN-test-speed.md: do the entries match the behavior the diff lands, and do docs the repository's rules name for this kind of change land in the diff (gatecli doc.go, impactset doc.go)?"
  - "Test economy: new tests in impactset watches_test.go, targets_test.go, tiers_test.go and gatecli tests that duplicate existing coverage, and new behavior left uncovered."
---

Focus is drawn from the board entry in the decision record's Goal, since the plan sections are empty.
