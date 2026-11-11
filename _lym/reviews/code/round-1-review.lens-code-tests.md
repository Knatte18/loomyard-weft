```yaml
lens: code-tests
outcome: reported
```

**Lens: code-tests.** Per behaviour, the test that pins it, or none.

1. **The eighth API-error turn end ends the start as api-error, with no load typed after it.**
   - Pinned by the new row in `TestStartup_LoadTurnAPIError` (`startup_test.go:1882`).
   - `wantTyped` is `loadHoldMaxAPIErrors-1` loads, so an off-by-one in `>=` fails the row.
   - `wantFailed`, `wantOutcome: "api-error"` and `wantFailedAfter: cappedHold` (the seven back-offs) also fail it, as does a hold that waits past the cap.
2. **The start error names the cap and the count.**
   - Pinned by the `wantErrText` check at `startup_test.go:1958`.
   - The other rows have an empty `wantErrText`, so `strings.Contains` passes trivially for them, which is correct.
3. **The deadline extension (`endLoadHold`) still runs on the capped path.** Pinned by `wantHoldOver: true` on the capped row.
4. **The parent notice goes out once before the cap.** Pinned by `wantNotices: 1`, which asserts the count only.
5. **The parent notice text names the cap (`loadturn.go:272`, "At %d API errors…").**
   - No test asserts the text, so none pins it.
   - The `notices` slice is captured at `startup_test.go:1925-1935` but never compared to any text.
   - This is runtime output, not static prose, so it is not exempt.
6. **`Result.APIError` is set on a capped hold (`abandonLoad`, `loadturn.go:315`), to the newest error text or `apiErrorUnknownText`.**
   - None. `TestStartup_LoadTurnAPIError` asserts the error and the outcome through `Start()`'s error, never `Result.APIError`.
   - The only `result.APIError` assertion is `TestRun_LoadTurnModelRejectedEndsTheStart` (`startup_test.go:2028`), and it covers the model-rejected path.
   - Dropping the `capped` branch at `loadturn.go:315` leaves every test green.
7. **When every output file exists, a capped hold returns the run instead of failing the start (`abandonLoad` → `classifyDeadlineExpiry` → `OutcomeDone`).**
   - None for the capped error. The change here is `errors.As(err, &capped)` joining the `OutcomeAPIError` case.
   - A deadline-expiry test for the model-rejected or timeout cases may cover `classifyDeadlineExpiry` itself, but nothing runs the capped error into it.
8. **A passing API error that clears before the cap resets or preserves the count.** The existing rows (an error followed by a load run through) cover `watch.errors` below the cap. No row crosses a turn start between errors and then reaches the cap.
9. **`StartedAt` is recorded on the done entry.** Pinned by `TestStep_DoneWithNonEmptyOnDone` (`step_test.go:57-59`), which asserts the format and `StartedAt <= At`.
10. **`StartedAt` on the stuck, reroute and reroute-capped entries, and on the fold path (the fold keeps the kept entry's `StartedAt`).**
    - None. `rerouteStuck` takes the new `startedAt` parameter on three entry paths, and nothing asserts it on any of them.
    - The fold-keeps-`StartedAt` behaviour has no row in an `appendOrFold` test either.
    - Without these, the `idleGapsOf` result for a stuck step, which is the case the Goal is about, depends on `StartedAt` values nothing pins.
11. **`idleGapsOf`.**
    - Pinned by `TestIdleGapsOf`: the floor (5m0s reported, 4m59s not), a folded entry ending at `LastAt`, a goto ending at its own `At`, an entry with no `StartedAt`, and an unparseable time.
    - Not covered: a skipped entry (the `OutcomeSkipped` arm of the same condition as goto), an out-of-order or negative gap, and a reroute entry.
    - These are cheap rows in the existing table, so they are not separate findings.
12. **The `idle_gaps` envelope key.** Pinned in the empty case by `TestStatusCmd_FoundEnvelope_GenericCore` and by the key set in `TestStatusCmd_EnvelopeKeySet`. No envelope test pins a non-empty `idle_gaps` or its JSON field names (`after`, `after_outcome`, `before`, `from`, `to`, `idle`), so a rename of a tag passes every test.
13. **The human `idle` line.** Pinned by the new `TestRenderStatusHuman` case, which asserts the format, its position before `next`, and that a short gap prints nothing.

**Findings**

- **F-T1, LOW, design.**
  - Location: `internal/shuttleengine/startup_test.go` (the `TestStartup_LoadTurnAPIError` capped row); evidence: `loadturn.go:315-317` against the only `APIError` assertion at `startup_test.go:2028`.
  - Summary: `Result.APIError` on a capped hold has no test, which is behaviour 6.
  - Fix: assert `Result.APIError` on the capped row, or add a capped counterpart of `TestRun_LoadTurnModelRejectedEndsTheStart`. This also covers behaviour 7 when it uses a done-output variant.
- **F-T2, LOW, design.**
  - Location: `internal/shuttleengine/startup_test.go:1925-1949`; evidence: `loadturn.go:272`.
  - Summary: the notice text that names the cap is never asserted, which is behaviour 5.
  - Fix: add a `wantNoticeContains` field, or compare the captured `notices` entry on the capped row.
- **F-T3, MEDIUM, design.**
  - Location: `internal/shedengine/step_test.go:57` (the only `StartedAt` assertion); evidence: `reroute.go:49,59,70` and `run.go:383`.
  - Summary: `StartedAt` is pinned only on the done path. The stuck, reroute and reroute-capped entries and the fold keeping `StartedAt` (behaviour 10) have no test row, though the Goal's stuck-step idle gap depends on them.
  - Fix: add `StartedAt` assertions to the existing reroute and stuck step tests, and one `appendOrFold` row asserting the kept entry's `StartedAt`.
- **F-T4, LOW, design.**
  - Location: `internal/shedverbs/status_test.go` (`TestStatusCmd_FoundEnvelope_GenericCore`); evidence: `status.go:108-118` JSON tags.
  - Summary: no test checks a non-empty `idle_gaps` in the envelope or its JSON field names (behaviour 12).
  - Fix: seed one over-floor gap in a status-command test and assert `idle_gaps[0]` keys and values.

Test economy: no new case duplicates an existing one. The `EnvelopeKeySet` change adds a required key, which is not duplication.

Tool calls (1):
1. `grep` of `startup_test.go` for `APIError`, `notices` and `wantNotices`, to confirm whether `Result.APIError` and the notice text are asserted; the context showed only the diff, not the rest of the test file.
