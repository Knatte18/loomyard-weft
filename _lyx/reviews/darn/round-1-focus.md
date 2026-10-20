---
round: 1
focus:
  - "internal/testkit/lyxbin/lyxbin.go `Run` and `runUntilDone`: does the timeout error keep the shape callers rely on? The message order changed (timeout text now precedes the args), so check every caller of `Run` and `ErrTimeout` for a match on the old text, and check that `Run`'s doc comment still states its behavior."
  - "`runUntilDone` timeout detection through `context.Cause`: settle whether a command that exits on its own just as the context is canceled, or a context canceled with a cause other than `ErrTimeout`, still takes the right branch and never reports a spurious timeout."
  - "`TestRun_KillsTheGroupOnTimeout` and `cancelOnceFileExists`: settle whether the test still depends on how fast the helper starts (the board entry's expectation), and whether the poller, the `t.Cleanup` cancel and the two-minute cap can leak a goroutine or leave the sleeper group alive on a failure path."
  - "The pid-file rename in `runSleeper`: settle whether the test reads the file only after the rename, and whether the `.tmp` file can be left behind or collide with another reader."
  - "`TestRun_TimesOutNamingTheTimeout`: per the test-economy rule, settle whether it duplicates the existing coverage of `Run`'s timeout path, and whether its 200 ms timeout can leave the sleeper group running after the test."
  - "Scope against the board entry: settle whether every change in the diff serves the entry's ask that the test not depend on helper start speed, and whether the change description's claims match the diff."
---

Seeded from the diff of two files in `internal/testkit/lyxbin`, read against board entry `lyxbin-timeout-test-xdr`.
