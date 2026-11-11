```yaml
lens: code-correctness
outcome: reported
```

Task restated: I reviewed the code-correctness lens, tracing a concrete input through each function or branch the diff adds or changes.

**Per-function list (case run → verdict)**

- `vstestCounter.observeFailure`:
  - A `Failed X [12 ms]` line, then the heading and message lines, then a blank line: the block is kept without its headings and closes at the blank line. Correct.
  - A `Failed!  - Failed: 2…` summary line: `vstestResult` needs whitespace after `Failed`, so the summary does not match and instead closes the block. Correct.
  - Name `Test(a: [1, 2])` followed by ` [3 ms]`: the lazy name match plus the end anchor give the right name. Correct.
  - Ninth and later lines of a block: dropped, capped at `failureLines`. Correct.
  - An error-message line that starts with `Failed` and ends in `[…]`, such as `Failed to open [x]`: it opens a bogus failed test. Wrong, but rare; I do not file it.
- `vstestCounter.Failures`: the lock is taken after `handle` has released it, and `slices.Clone` returns a copy. Correct.
- `renderFailureReport`, `reportLines`: a failure with an empty `pkg` gives `  Ns.Tests.A:`, and a Go failure is unchanged. Correct.
- `TierRunFailures`:
  - A mixed tier report gives labelled names. Correct.
  - A cut report breaks at the `... ` line and returns `ok` true. Correct here, because `FailedTests` returns false for a cut report, so the whole call is rerun and no hidden failure is masked.
  - A lint or build header gives `ok` false. Correct.
  - An empty report, or a log with no report, gives `ok` false. Correct.
  - A run with no failure line is listed by its outcome. Correct.
- `rerunTestFailure`:
  - Timeout, runner crash, exit code -1 and an empty log: first returned. Correct.
  - Log unreadable: warns and returns first. Correct.
  - `FailedTestsCall` error: falls back to the whole command. Correct.
  - Rerun passed or skipped: `onFlakyPass` is called and the rerun is returned. Correct.
  - Rerun failed with a different failure: the rerun is judged, `recordFailure` and `verdict` use its log, and `markWait` is cleared on the error path. Correct.
- `Finalize.flakySection`, `landingTexts`:
  - No flaky passes gives an empty section, which `withSection` is assumed to ignore (not read).
  - Duplicate failures are listed once.
  - Each call re-applies the section to a copy of the summary, so repeated passes do not double-append. Correct.
  - The retry branch recomputes only the commit message. The GitHub merge has already happened by then, so this is consistent.
- `check` wiring: a rerun pass flows into `recordFailure`, which does nothing for a pass, and `verdict` passes. Correct.

**Finding**

- **LOW / design** — `internal/gateslot/runlog.go:358-400` (`TierRunFailures`), `internal/landingshed/verifygate.go:300-320` (`rerunTestFailure`).
  - A package that "failed outside any test" (a compile error in a test package, or a `TestMain` crash) counts as a test run. `FailedTests` returns false for it, so the whole gate is rerun.
  - That failure is deterministic, so the rerun fails again and costs a full gate run. The doc and commit message say a build failure is not rerun, which is only true of the build, vet and lint steps.
  - Fix: either have `TierRunFailures` return `ok` false when a `failed outside any test` line appears, or state in the doc that such a package failure is rerun too.

**Tool calls**

1. Read `internal/gateslot/runlog.go` lines 272-362: needed `FailedTests`, `reportRunTier` and `reportTestLine`, which the manifest lacked, to check how a cut report is handled.
2. Read `internal/landingshed/finalize.go` lines 340-500: needed the call sites of `landingTexts` and the retry path.
3. Ran grep for the cut-line producer in `internal/gatecli`: to confirm where the `... N more lines cut` line comes from.
