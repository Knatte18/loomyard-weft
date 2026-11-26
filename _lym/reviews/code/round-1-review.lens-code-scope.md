```yaml
lens: code-scope
outcome: reported
```

Task: the code-scope lens, which traces each change in the diff to the decision record's goal points and added entry.

**Per change**

- **C1** — the `running *sync.WaitGroup` field on `watchedSession` and its doc lines, plus `enterSession` starting the OOM watch through `running.Go` (`internal/reedcli/watchdog.go:200-207`, `:354-357`, `:364`).
  - Traces to G2, "make the goroutine's exit awaitable", and to A1.
- **C2** — the name-repair goroutine also started through `running.Go`, and the final return carrying `running` (`watchdog.go:367-372`).
  - Traces to the Goal's idea paragraph: "make the test wait for whatever it started to stop".
  - The issue section names only `watchPaneOOMKills`, but the goal's own wording covers every goroutine the session starts.
  - `TestEnterSession_DrivesTheToldTmux` uses `watchdog: off`, so this branch never runs there. It runs for no test the diff touches.
  - Even so, the goal's "whatever it started" licenses it, so it is not unrequested scope.
- **C3** — the `stopWatchedSession` helper, and `TestEnterSession_DrivesTheToldTmux` cleaning up through it (`internal/reedcli/entersession_integration_test.go:49`, `:58-74`).
  - Traces to G3, "have the cleanup wait for it before the temp directory is removed", and to A1.
- **C4** — the `ReEntryReReadsFlippedConfig` subtest of `TestWatchdogDaemon` cleaning up through `stopWatchedSession` (`internal/reedcli/watchdog_integration_test.go:515`).
  - Traces to G3: it is the only other test that calls `enterSession` and has the same cancel-only cleanup.
  - The entry names only `TestEnterSession_DrivesTheToldTmux`, but the issue's "What to do" asks to "make the goroutine's exit awaitable and have the cleanup wait for it". That applies to each test cleanup that cancels the goroutine.
  - It is a small, same-defect extension, not new behaviour, so it is not unrequested scope.
- **C5** — the `time` import added to `entersession_integration_test.go`.
  - Forced by C3, since the helper's 30s bound uses `time.After`.

**Per decision**

- **G1**, confirm the likely cause: built by the commit message's cause paragraph and its `Root-Cause:` trailer, and by the summary's `## Cause`.
  - These carry the confirmation by reading the code. They are not a diff change, which is correct for this point.
- **G2**, make the goroutine's exit awaitable: built by C1, and by C2 for the other goroutine.
- **G3**, the cleanup waits before the temp directory is removed: built by C3, and by C4.
- **A1** (Added after Discussion, parent, 2026-11-26), no reproduction run, confirm by reading, make awaitable, cleanup waits:
  - Built by C1 and C3.
  - The range adds and changes no test-run artifact, and the commit cites code reading only, so there was no reproduction run.

**Findings:** none. Every change traces to a goal point or the added entry, or is forced by one. No decision is left unbuilt.

**Tool calls:** none. The inherited context held the diff, the decision record, the commit log and the summary.
