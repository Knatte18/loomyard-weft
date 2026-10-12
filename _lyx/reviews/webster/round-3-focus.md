---
round: 3
focus:
  - "Does every package that holds `_test.go` files, including `internal/fabricvocab` which arrived through merge-in commit 35f6a6880, carry a `TestMain` calling `os.Exit(tmuxkit.Main(m))` so that `TestTmuxIsolation_EveryTestPackageRunsThroughTmuxkitMain` in `cmd/lyx` passes? Settle it against the branch's committed diff."
---

The Publish failure was the missing `TestMain` in `internal/fabricvocab`; this round checks the fix and that the merge-in brought in no other test package without one.
