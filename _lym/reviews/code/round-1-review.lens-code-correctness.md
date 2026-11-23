```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens over the diff (`internal/loomcli/gotohook_integration_test.go`, commit `350370384`).

**Branches traced, with the case run through each and its result**

1. **Row `AnotherRowKeepsTheRecord`, record `held`, target Finalize:** correct.
   - `prReviewEntryReset` returns nil for any target that is not PR-Review (`prreviewgoto.go:19-23`), so the record is never touched.
   - The record read back is `held`, and `reflect.DeepEqual` follows the `Autonomy` pointer to true on both sides, so it holds.
   - The `wantErr == ""` branch sees a nil error.
2. **Row `PRReviewClearsTheAutonomyAndTheAskMark`, record `held` with no strand, target PR-Review:** correct.
   - `clearPRReviewOnGoto` reads the record and finds `StrandGUID == ""`, so it skips the stop.
   - `ClearReviewSession` (`record.go:97-108`) sets `Autonomy` to nil and `ParentReviewAsked` to false.
   - The commit is skipped through the test's environment settings.
   - The record read back is `{Number:7}`. Every cleared or unset field is omitted from the JSON, so it reads back as its zero value, and DeepEqual against `Record{Number: 7}` holds.
   - Switching from the old three-field check to DeepEqual makes this row stricter. It now also asserts that the strand, session and engine fields are empty, which the old check did not.
3. **Row `PRReviewWithARecordedStrandStopsItThroughARunnerOfItsOwn`, record `withStrand`, target PR-Review:** correct.
   - `StrandGUID` is "guid-1", so the hook's stop closure runs.
   - The test builds the receiver with no runner (`c := &loomCLI{location: pair, runID: ...}`), so `c.runner == nil` and `stepStopRunner` builds a runner (`looparm.go:175-198`).
   - `Runner.StopStrand`'s `FindRun` finds no run, so it calls `removeStrandIfPresent` (`stop.go:150-157`).
   - `reed.Status()` then fails in `requireSessionLocked` (`lifecycle.go:1300`). No reed session exists on tmuxkit's private socket (`testmain_test.go:22`), and the hub path is unique per `CopyHub`.
   - That error is wrapped as "loom: the goto onto PR-Review is written, but it could not stop the recorded PR-Review session: …" (`prreviewgoto.go:55-57`), and `ClearReviewSession` never runs.
   - The record read back is still `withStrand`, so DeepEqual holds, and the substring check passes.
   - Before the fix that built a runner, this path would have panicked on a nil `*Runner` (`StopStrand` reads `r.toldErr`, `stop.go:151`), so the row would have caught the original defect.
4. **Error branches of the loop:** correct.
   - A row expecting no error that gets one reports with `Errorf` and still checks the record.
   - A row expecting an error that gets nil is caught by the `err == nil ||` check, so a nil error cannot slip through.
   - Each row writes its own starting record first, so a failure in one row does not carry state into the next.
5. **Whether the stop's failure is deterministic:** correct on Linux.
   - With no tmux session, the probe fails whether or not tmux is installed.
   - Nothing depends on the clock or the machine's own tmux, since tmuxkit swaps in its own socket directory.

**Findings**

- **LOW, design, `internal/loomcli/gotohook_integration_test.go:54`:** the row's error substring cannot tell its two failure causes apart.
  - The substring "could not stop the recorded PR-Review session: " matches a failure in `stepStopRunner`'s config or geometry load as well as a failed `StopStrand` after the runner is built.
  - Both errors carry the same `prReviewClearFailure` wrapping (`prreviewgoto.go:74-78`).
  - So the row still passes if the hook never builds a runner but fails loading config, even though its name and doc comment (lines 20-22) say a runner of its own is built and the stop fails.
  - Evidence: `prreviewgoto.go:72-78` and `stop.go:226-230`.
  - Fix: assert a substring only the post-build path produces, such as "probe reed before removing it".

Tool calls: none; the inherited context held every definition this lens needed.
