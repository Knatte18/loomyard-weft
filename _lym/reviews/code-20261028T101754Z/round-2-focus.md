---
round: 2
focus:
  - "internal/fabriccli/hubpull_integration_test.go: does a test take the prime's parent lock as another owner, run `pull --hub` against it, and show the pull waiting, then finishing once the lock is released, and the Ctrl-C end of the wait?"
  - "internal/fabricengine/hubpull_integration_test.go, the stopped-side test: is a side that really pulls and moves asserted after a stopped side, so a loop that ended at the first stop would fail?"
  - "internal/fabricengine/hubpull.go: with SkipGit set, does every pair side report up_to_date, and does one unreadable beam record stop only that beam?"
---
