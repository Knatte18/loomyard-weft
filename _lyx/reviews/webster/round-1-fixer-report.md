# Webster round 1 — fixer report

All seven findings are fixed, with one commit per fix and nothing pushed.
After the last fix, `go build ./...`, the plan's `go test -tags integration` line and `go vet -tags smoke` all pass.

## Fixed

- **F1 (LOW)**: `a0417f160`, `7760a40c7`.
  `isBoxRule` is back to the strict glyph-only match.
  A new `isTopBoxRule` also accepts the labelled shape, and `IdleSession` uses it only on the upward walk to the top rule.
  So a draft line shaped like a labelled rule below the caret no longer closes the box.
  `session_test.go` gains two near-miss cases: such a draft line is not idle, and a label with no trailing rule glyph is not a top rule.
  The orchengine doc's smoke record now names the labelled top rule instead of `isBoxRule`.
- **F2 (LOW)**: `57372314c`.
  `TestSmokeOrch_SoftCycle` now requires the handoff to name `SOFTBG` or `sleep 900`, and no longer accepts the bare word `background`.
  I re-ran the test live after the change (`go test -tags smoke -run 'TestSmokeOrch_SoftCycle$'`): it passed in 105s.
- **F3 (LOW)**: `0f7913535`.
  The `AllowAgentTool`, `PermissionMode` and `ResumeSessionID` docs and the changed `Interactive` sentences in `spec.go`, plus the `run_resume_test.go` header, now use semantic line breaks.
  The untouched legacy lines of `Interactive` were left as they were.
- **F4 (LOW)**: `bad0d9237`.
  `resume.go` and `resume_test.go` now have file-header comments in the package's style.
- **F5 (NIT)**: `fc55cc763`.
  `checkResumable` keeps the "no transcript" refusal for `os.ErrNotExist` only.
  Any other stat error now refuses as "cannot stat the transcript".
  A new test covers this on non-Windows platforms; on Windows a path through a file reports not-found, so the case cannot be reached there.
- **F6 (NIT)**: `f65319b54`.
  `Prepare`'s doc comment now names all four checks it runs before writing artifacts.
- **F7 (NIT)**: `f5ef95742`.
  The new `TestWatcher_SoftDeferTurnEndAloneNeverRequalifies` shows that with no turn end after a DEFER, the watcher sends nothing even an hour later.
  I checked the test catches the regression: with `toIdle` changed to keep `w.newest`, it fails.

## Deferred with reason

None.
