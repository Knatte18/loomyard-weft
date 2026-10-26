```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, I name the test that pins it, then apply "Test economy". I made no tool calls beyond the one test diff I read.

**Per behaviour: test that pins it**
- **EAGAIN retry in `run` and `output`:** pinned by `TestTmuxExecRetriesEAGAIN` (`overlay_test.go`, new). It covers success after retries, every attempt failing, a nonzero exit run once, and the doubling pauses, for both the capture and non-capture paths. The pause is injected.
- **`runStdin` replay of non-nil stdin on a retry:** none. `run` goes through `runStdin` with nil stdin, and `output` passes nil stdin to the hook. No row sends bytes and checks the retry gets them again.
- **`start_retries` config:** pinned by three rows in `TestLoadConfig_SubmitSettleKeys` (default, 0, negative refused). The template declaration and `ConfigSpec` classification are not asserted by a new row.
- **Died start, strand removed, launched again:** pinned by `TestStartup_DiedStartLaunchesAgain`. Its three rows are retry on, `start_retries: 0`, and a failed removal. It asserts the AddStrand count, one run directory per launch, and one died outcome.
- **Last attempt's error returned:** none. The test asserts only that the error wraps `ErrNotStarted`.
- **`timeout` start never retried:** none.
- **Output files existing never retried:** none.
- **5 s `startRetryDelay` on the run clock:** none. The fake clock absorbs the sleep and nothing asserts the delay.
- **Strand removal retry (three attempts):** pinned by two new rows in `TestStartup_PaneNotLiveMidStartup`. One row has every attempt failing and `wantRemoveCalls` is `startupRemoveAttempts`, with the by-hand remedy still in the error. The other has one failure then success and expects "the strand was removed".
- **Doubling delays of 1 s then 2 s in the removal retry:** none. No test reads the clock's sleeps.
- **Head and tail of the not-ready capture:** pinned by `TestStartupCaptureExcerpt`. Its rows are short, exactly one tail, a little past one tail (no repeated lines), and a long capture.
- **Excerpt wired into the `abandonStartup` error:** only the last-lines half is asserted. The `wantTail` check was already there, and nothing checks the "first lines" block in the error text.
- **Held notice withheld, ungated shell expiry:** pinned by "a shell outstanding past the bound on an engine that wakes on background work holds without a notice".
- **Held notice withheld, gated plain-turn-end on un-awaited shells:** pinned by "a gated turn end on an unawaited shell on an engine that wakes…".
- **Notice still sent on an engine without background wake:** pinned by the gated "does not wake" row.
- **Plain held turn end still notified on a background-wake engine:** pinned by "a plain stop on an engine that wakes on background work is notified".
- **Ungated expiry on an engine without background wake:** no new row. An existing ungated-expiry notice row, which I did not open, presumably covers it.

**Findings**

1. **LOW, consistency, `internal/shuttleengine/startup_test.go` near the `keyTimesReed` type.** The new `TestStartup_DiedStartLaunchesAgain` and `TestStartupCaptureExcerpt` were inserted between the `// keyTimesReed records …` comment and `type keyTimesReed struct`. The type lost its doc comment, and that comment now opens the doc of `TestStartup_DiedStartLaunchesAgain`. Evidence: the diff hunks at the "keyTimesReed records the clock time" context line and the trailing `type keyTimesReed struct`.

2. **MEDIUM, design, `internal/reedengine/overlay_test.go` (`TestTmuxExecRetriesEAGAIN`).** The focus asks whether stdin is replayed correctly on a retry, and nothing pins it. No row sends non-nil stdin through `runStdin` and checks the retry's hook call carries the same bytes. This is an added behaviour with no coverage, which Test economy lists as a finding. A fix is one row in the same table with a `runStdin` variant.

3. **MEDIUM, design, `internal/shuttleengine/startup_test.go` (`TestStartup_DiedStartLaunchesAgain`).** The decision record's never-retry cases for a `timeout` start and for a run whose output files exist have no row. The table covers only the "removal failed" and "retry off" non-retry cases. If `startRetryable` dropped its `OutcomeDied` check, no test would fail. The same goes for an output-files guard (the record asks for one, and `startRetryable` in `run.go` has none). Uncovered behaviour.

4. **LOW, design, same test.** It does not assert that the returned error is the last attempt's, which the acceptance criteria require and the record states. It also does not assert the retry's clock wait (`startRetryDelay`). The strand-removal retry test likewise does not assert the 1 s then 2 s doubling on the run clock. Each of these is a stated bound with no assertion.

**Test economy**
- No new test duplicates an existing test's coverage.
- The `start_retries` rows extend an existing table.
- The held-notice rows extend `TestWait_HeldTurnEndNotice`.
- The removal-retry rows extend `TestStartup_PaneNotLiveMidStartup`.
- No stencil-prose assertions were added.
- The "a capture of one tail shows once" row overlaps the "short capture" row only partially (it checks the boundary), so I raise nothing.

**Tool calls**
None. The test diff was already in context from the shared diff read, and nothing I needed lay outside it.
