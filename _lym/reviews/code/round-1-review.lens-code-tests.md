```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, I name the test row that pins it, or say none.

**Per behaviour**

1. **A child that ends before its record carries the log's error on one line (`readChildReport` and `startFailure`, no fallback).** Pinned by the `spawn_test.go` row "a child that ends on an error envelope carries its error on one line". The row has a non-envelope line before the envelope, and the error text contains an escaped newline that gets folded.
2. **The last envelope wins.** No row pins it. Every row's log holds one envelope at most, so a version that kept the first envelope would pass every row. Finding 1.
3. **A non-string `error` or `ok:true` envelope is ignored.** No row pins it. This is inside the "non-envelope tail" item of the focus file, but the code's trust-boundary reading of a log makes it a minor gap. Finding 2.
4. **`boundChildError` cuts at `childErrorMaxRunes` and marks the cut with an ellipsis.** Pinned by the row "an overlong error is cut to the bound", which checks the cut text and that the uncut text is absent. The test uses ASCII only, so a byte-based cut would pass. Rune safety is not pinned. Finding 3.
5. **`startFailure`, lasting error with FallbackDetach: no fresh capture offered, message says it cannot change, `lasting` set.** Pinned by the row "a lasting error offers no fresh capture, with fallback". It checks `want` and `wantNot` ("run lym drift capture again"), and the `errors.As` assertion checks `lasting`.
6. **`startFailure`, lasting error without FallbackDetach: `lasting` is carried and the message names the log.** Pinned by the row "a lasting error is marked for the calling row, no fallback".
7. **`startFailure`, non-lasting error with FallbackDetach: the old way-forward text is kept.** Pinned by the existing fallback rows, which now check "run lym drift capture again". The loop's `errors.As` assertion also pins `lasting == false` on every non-lasting row.
8. **The stall path (`did not record its start within`) with a lasting error.** No row. It shares `startFailure` with the exit path, so this is covered through shared code only. Not a finding.
9. **`captureAgain`, lasting `childNoRecordError`: halt drops "lym loom resume to capture again" and names defer plus the fix of the cause.** Pinned by the `baseline_test.go` row "a child that ended on an error a resume cannot change…". Its `wantNoReason` and `wantReason` checks would fail if the lasting case were moved below the non-lasting `errors.As` case.
10. **`captureAgain`, non-lasting `childNoRecordError`: the old halt text is kept.** Pinned by the existing DriftBaseline rows.
11. **`runCaptureChild` marks an `openWorktree` failure with `lasting_fault` in the envelope.** Pinned by the tier-1 integration assertion in `anchor_integration_test.go:77`. It runs `capture-child` from the worktree root of an anchored-subdirectory hub and checks the error text and the `LastingFaultKey` value.
12. **An `openWorktree` failure that is not lasting (for example `ErrNotAGitRepo`) is not marked, or is marked.** No row. The diff marks every `openWorktree` error. That is not tested either way, and it is a behaviour question for the code-correctness lens, not a missing-test finding for me.
13. **The `lym drift capture` (FallbackDetach) message with the lasting form, end to end through the CLI.** Only the engine-level row 5 pins it. The CLI integration test never runs `capture` against a lasting child, because the main capture there succeeds. The wiring is a single `FallbackDetach: true` field, so this is not a finding.

**Findings**

1. LOW, `design`, `internal/driftengine/spawn_test.go` (the `failures` table, about lines 110–150). Nothing pins that `readChildReport` takes the last error envelope. The entry says "last error line". Add a row whose log holds two envelopes, with the earlier one lasting and the later one not (or the reverse), and check the later one's text and `lasting`. Evidence: the `readChildReport` loop in `spawn.go` overwrites `report` on each match, and no row has two envelopes.
2. NIT, `design`, `internal/driftengine/spawn_test.go`. No row has a non-string `error` or an `ok:true` envelope, and the tail of the log is never a non-JSON line after the envelope. The filtering branches (`envelope["ok"] != false`, the `.(string)` assertion) are unpinned. Fold these into a single extra row, not separate tests.
3. NIT, `design`, `internal/driftengine/spawn_test.go`, row "an overlong error is cut to the bound". It uses ASCII `x` only, so a byte-count cut would pass. Using a multibyte rune such as `å` in the repeated string would pin the rune-based cut.

I found no redundant test. The two `spawn_test.go` lasting rows hit different `startFailure` branches, and the `baseline_test.go` row tests the `captureAgain` layer, not `startFailure`.

Tool calls: none; the diff, the existing test names and both source files were already in context.
