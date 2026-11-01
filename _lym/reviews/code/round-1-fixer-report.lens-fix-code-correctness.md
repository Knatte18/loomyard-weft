Task: check the correctness of the changed code (F1 to F6) through the fix-code-correctness lens.

Findings:

- **LOW**, `unmatchedLaunches` with `countPending` (background.go).
  - **Summary:** a fork `tool_use` that never gets a `tool_result` keeps `OutstandingForks` non-empty permanently, because nothing can ever name it. A crash, or a resume that dropped the result line, would cause this.
  - **Evidence:** the new branch `if countPending && l.resultAt == -1` appends the launch with an empty ID. No notification can match it, since matching needs ids.
  - **Mitigation:** the stencil bound of five consecutive `ready: false` answers (F4) already ends the wait. I propose no change.

Everything else I traced holds:

- **F1 code:** a launch whose result is present but carries no id still skips, so a failed or rejected launch does not hang the wait. Shell launches (Bash, Monitor) are dropped by the fork filter in `unfinishedForks`. `transcriptHasUnmatchedLaunch` keeps `countPending` false, so the Stop-payload fallback and its existing tests are unchanged.
- **F1 test row:** the first line of `forkLaunchTranscript` yields `{fork, ID "", Label "", SignalTranscript}`, which matches `want`. Before the fix the launch was skipped and the result was `nil`, so the row fails without it.
- **F5 test:** with no `background_tasks` key it takes the transcript fallback, and the bare-fork launch with no notification yields `EventWaiting` with agent id `a5e3629456cc7489a`. It fails if `isBackgroundLaunch` stops counting bare forks.
- **F3 test:**
  - The three rows hit the three paths: the capability refusal, the unknown guid (the wrapped message contains the guid), and the reader receiving `session-1` and `paneCwd`.
  - With cap 0, `awaitForks` reads once and returns `(forks, false)`. The earlier failure was only the anchor == worktree validation, which the fixer corrected and the gate run confirmed.
  - Dropping the `SessionID` or `paneCwd` hand-off would fail the assertion.
- **F2:** the `strandResolver` interface keeps the `c.reed = reedEngine` assignment in `PersistentPreRunE` valid, and `reedengine` is still imported for `LoadConfig`.
  - **Test rows:** the unset-env, resolve-error, no-run-record, ready and ready-false rows exercise real branches of the verb.
  - **Interference:** the `t.Setenv` test is non-parallel, so it cannot disturb the parallel `TestAwaitVerb`.
  - **Registration:** the registration subtest fails if `await-forks` is dropped from `cli.go`, which the stray-argument row alone would not catch.
- **F6:** the wrapped text still contains the guid, so the unknown-guid assertion holds.

Tool calls made: none. My context already held every changed file and the gate output.
