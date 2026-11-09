---
round: 1
focus:
  - "Every changed `_test.go` file in the range: does each rewritten test still prove its original claim (for example 'does not wait on X') now that a controlled release replaces the elapsed-time comparison, or can it pass when the code does wait?"
  - "`internal/loomcli` `TestIntegrationDriverBootstrap_ReturnsWithoutWaitingOnTheDriver`, the named failing case: is it in the diff, and does it block the stub on a release the test controls instead of comparing against the stub's 5s delay?"
  - "Tests that still compare a measured elapsed time to a fixed limit anywhere in `_test.go` files, including the packages the goal names (`shuttleengine`, `creelengine`, `shedverbs`, `verifyrun`, `tenderengine`, `seatengine`, `reedengine`): did the diff miss any that the goal asks to be made load-proof?"
  - "Changes to tests that only set a deadline: does the diff touch tests that never compared a measured time to a limit, beyond the goal's scope?"
  - "Each new release, channel or goroutine in the rewritten tests: can it leak, deadlock or race when the test fails early, and is every blocked path bounded by a deadline branch?"
  - "`pattern/PATTERN-test-speed.md` diff: does the added text match what the diff does and the repository's pattern conventions, and are the other docs a pattern change requires in the same diff?"
  - "`internal/gatecli/test_integration_test.go` and `internal/reedengine/ensuresession_integration_test.go`, where lines were net removed: did the removal drop coverage or an assertion the goal did not ask to drop?"
---

Seed focus from the goal in the decision record, which holds only the board entry under `## Goal`, and from the diff's file list.
