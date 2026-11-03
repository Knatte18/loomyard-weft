```yaml
lens: code-tests
outcome: reported
```

Lens code-tests: for each behaviour the diff adds or changes, the test row that pins it, or none.

**Per behaviour**

| Behaviour | Pinning test |
|---|---|
| A re-send after the back-off lets the agent finish | `TestWait_APIErrorRetry`, row 1, `internal/shuttleengine/apierrorretry_test.go:419` |
| The re-send comes after `apiErrorRetryBackoff`, then double that | Same test, rows 1 and 2 (`wantGaps`, checked at line 487) |
| A turn end past the last re-send ends `OutcomeAPIError` with `Result.APIError` naming the error | Row 2; `wantAPIError` is asserted on every row, so other outcomes are pinned to an empty `APIError` |
| Zero retries re-sends nothing and waits to the deadline | Row 3 |
| Every output file on disk at an API-error turn end ends the run done, with no re-send | Row 4 |
| `OutcomeAPIError` is audited in the completion-signal tripwire | `completionsignal_enforcement_test.go:47,63,160` |
| The arbiter's `concludeWait` stops the strand and hands on with cause `api-error`, error named in reason and question | `TestCall_VerdictOutcomes` new row, `call_test.go:804` |
| Both settler specs set `APIErrorRetries` | `TestSpec` (`shedarbiter/prompt_test.go:160`) and `TestTacklerSpec` (`sheddriver/tacklerprompt_test.go:168`) |
| The tackler's `concludeWait` stops the strand and hands back with cause `api-error` | `TestTacklerCall_SpawnPath` new row, `tackler_test.go:283` |
| A non-API cause's question text is unchanged by `HandBackQuestionCause` | The existing `CauseDied`, `CauseTimeout` and similar rows, which assert cause and reason |
| The tackler's hand-back question names the error | **None** (F1) |
| A pending API error is cleared by a later turn start, turn end or session start | **None** (F2) |
| The retry count resumes from the prompt offset on an attach | **None** (F3) |
| A covered pause blocks the re-send and the end-on-error | **None** (F4) |
| A failed send is logged and retried on a later tick | **None** (F5) |
| `isTerminalOutcome` reads `api-error` as terminal on attach | **None** (F6) |
| `loopAPIError` pairs only with `SessionIdleStalled` in the session-state shadow | **None** (F7) |
| An engine without `SessionSignalParser`, or an unreadable events file, leaves the run unchanged | **None** (F8) |

**Findings**

- **F1, MEDIUM, design.** `internal/sheddriver/tackler.go:440`, evidence `tackler_test.go:283-289`.
  The tackler's `handBack` now builds its question with `HandBackQuestionCause`, so the parent hears the error. This is the Goal's "notice naming the stop and the failed settler".
  The new tackler row asserts only cause, verdict, starts and stops, with no question assertion. Reverting that one line leaves every test green.
  The arbiter's row adds `wantQuestion` and the tackler's has no counterpart.
  The fix is a `wantQuestion` field and assertion on `TestTacklerCall_SpawnPath`, set to `server_error` on the new row.

- **F2, MEDIUM, design.** `internal/shuttleengine/apierrorretry.go:333`, evidence `apierrorretry_test.go:419-437`.
  `pending` is cleared by a turn start, turn end or session start read after the API error. No row reaches that clear.
  After a send, `watch.pending = false` is set directly (line 351), and row 2's reaction is `START` then `APIERR`, where the `APIERR` re-sets `pending` anyway.
  Deleting the `case SessionSignalTurnStart, ...` branch leaves every row passing.
  A missing clear would make Wait re-send into a session that has already recovered on its own.
  The fix is a row where the events file gains `START` and `STOP:` lines on its own after the API error, with the output file still missing, asserting no re-send.

- **F3, MEDIUM, design.** `apierrorretry.go:293,321`, evidence `_lym/landing/summary.md` ("a re-attach resumes the count from the run's own events file").
  The count is seeded from `run.state.PromptOffset` and counts API errors already in the file. No test attaches or reconstructs a run whose events file already holds earlier API errors beyond the one the fixture seeds.
  All rows use `withRunEvents("START\nAPIERR:server_error\n")`, which is one error from offset 0.
  A row with several earlier API errors present at Wait entry, ending `api-error` with no re-send, would pin the claim.

- **F4, LOW, design.** `apierrorretry.go:338`.
  `run.covered` returns early, so there is no re-send and no end while a pause covers the run. No row sets pause coverage.
  The fix is a row, or an assertion beside the existing fixtures, with a covering pause record that re-sends nothing and ends nothing.

- **F5, LOW, design.** `apierrorretry.go:347`.
  A failed `sendWithin` leaves `pending` set and is retried on a later tick, bounded by the run deadline. No row fails the send.
  The fix is a row where the first `SendText` errors and the second succeeds, asserting one delivered re-send.

- **F6, LOW, scope.** `internal/shuttleengine/attach.go:615`.
  `isTerminalOutcome` now returns true for `api-error`, but `attach_test.go` is not in the diff and no row carries a persisted `api-error` record.
  Dropping that case from the switch leaves tests green.
  An attach to such a record would then treat it as non-terminal.
  The fix is a row in the existing table-driven `isTerminalOutcome` or attach-disposition test using `string(OutcomeAPIError)`.

- **F7, LOW, consistency.** `internal/shuttleengine/waitshadow.go:98`.
  `loopAPIError` pairs only with `SessionIdleStalled` in `loopAgreesWithState`. `waitshadow_test.go` is not in the diff and has no `api-error` loop-classification row.
  The shadow is display only, which is why this is LOW.
  The fix is a row in the existing shadow table that ends a run `api-error` and expects no disagreement logged.

- **F8, NIT, design.** `apierrorretry.go:316,323`.
  No row runs `retryAPIError` against an engine without `SessionSignalParser` or an unreadable events file.
  Both are silent no-ops and the first is a documented opt-in capability, so I report it at NIT.
  If it is wanted, it is one row, with `fakeEngine` in place of `sessionFakeEngine`, and `Retries: 2` waiting to the deadline.

**Duplication check:** no new test duplicates an existing one.
The two `TestSpec` and `TestTacklerSpec` assertions and the two `SpawnPath` and `VerdictOutcomes` rows are additions to existing tests, not new functions. `TestWait_APIErrorRetry` is the only new test function and covers a behaviour nothing else covers.

**Tool calls:** none. The manifest already held every file and symbol this lens needed.
