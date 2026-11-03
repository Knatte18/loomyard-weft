```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens, running each changed function and branch on a concrete case.

**List: function or branch, case run, correct or not**

- `retryAPIError`, `APIErrorRetries` 0: it returns at once and changes nothing. Correct.
- `retryAPIError`, engine without `SessionSignalParser`: it returns at once. Correct.
- `retryAPIError`, events file missing or unreadable: it reads as no signals and continues. Correct, since `pollEventsTick` owns the read-failure count.
- `retryAPIError`, retries 3, file holds `START` then an API-error turn end:
  - Error 1 re-sends after 30s, error 2 after 60s, error 3 after 120s.
  - Error 4 ends the run `api-error`, so there are 3 re-sends and then the end. Correct.
  - `errors-1` is never negative, because `pending` is only set together with `errors++`.
- `retryAPIError`, pending error with every output file present: `endOnAPIError(OutcomeDone)` runs `finalize(Done)`, which evaluates the gate. Correct. It does cut off a gated writer's remaining re-prompts, but the final gate evaluation still runs.
- `retryAPIError`, `run.covered` true: it returns before the bound check and the send. After the pause lifts, the old `readAt` makes the re-send due at once. Acceptable.
- `retryAPIError`, `sendWithin` fails:
  - `pending` stays true, so it retries on the next tick, and the run deadline bounds it. Logic correct.
  - It logs Info on every tick; see F2.
- `retryAPIError`, re-sent turn produces no hook signal at all (the send lands but the model never starts a turn): `pending` is false, so no further re-send happens and the run waits out its deadline silently. Edge, LOW. No finding raised, because the verified send makes a turn start likely.
- `retryAPIError`, re-attach after a crash:
  - The cursor restarts at `PromptOffset`, so earlier errors are recounted from the file. Correct.
  - A re-send already made is followed by a turn start, so `pending` is false. Correct.
  - A crash between the error and the send restarts the back-off from the new `readAt`. Acceptable.
- `retryAPIError`, retry budget across a run: the budget counts every API-error turn end since the prompt offset, not consecutive ones. A settler that recovered earlier still burns budget on later transient errors. Documented in the summary as intended, so no finding.
- `endOnAPIError` with `OutcomeAPIError`: `finalize` records the run `api-error` (or `stopped` if the handle was stop-marked). `result.APIError` is set from `watch.text`. Correct.
- `endOnAPIError` with `OutcomeDone`: `APIError` stays empty. Correct.
- `Wait` ordering: `retryAPIError` runs after `pollEventsTick` and before the liveness check and the deadline check. A dead pane with a pending error ends `api-error` rather than `died`, but only after retries are spent, so this is acceptable.
- `isTerminalOutcome`: `api-error` is terminal for attach, the same treatment as `timeout`. Correct.
- `waitshadow.go`:
  - `loopAPIError` pairs with `idle-stalled`. The StopFailure signal derives `idle-stalled` with cause `api-error` (`sessionstate.go:181`), so the pairing holds.
  - The api-error-then-done branch classifies `done` against an `idle-stalled` state and logs one display-only disagreement. No finding.
- `tackler.concludeWait`, `OutcomeAPIError`:
  - It stops the strand and hands back with cause `api-error`, and `waitErr` is checked first.
  - The verdict-on-disk check at the loop top still wins. Correct.
  - The case sits before the generic `!= OutcomeDone` branch, so it is not shadowed.
- `Call.concludeWait` (arbiter), `OutcomeAPIError`: same shape, correct.
- `HandBackQuestionCause("api-error", detail)`:
  - With detail `"…did not clear: server_error"`, the question reads `(api-error: the arbiter's session failed … did not clear: server_error)`. It works, but the error text appears twice, in `Reason` and in `Question`.
  - With empty `result.APIError`, the detail ends in a dangling `": "`. See F1.
- `Spec` constructors: both set `APIErrorRetries`. `AttachIfLive` builds the spec through the same `Spec` and `tacklerSpec`, so a re-attached run keeps the retry. Correct.
- The in-loop timeout re-attach in `tackler.concludeWait` rebuilds the spec through `c.attach`, which uses `tacklerSpec`, so the retry is kept. Correct.

**Findings**

- **F1 — LOW, design.** `internal/shuttleengine/claudeengine/signals.go:225-233`, `internal/shedarbiter/call.go:269`, `internal/sheddriver/tackler.go:301`.
  - `apiErrorText` returns `""` when a StopFailure payload carries none of the three keys.
  - The hand-on and hand-back reason then ends `...did not clear: `, and the question reads `(api-error: ...: )`. The parent is told no error text.
  - Evidence: the detail strings are built from `result.APIError` with no empty fallback.
  - Fix: substitute a fixed text such as `unknown error` when `result.APIError` is empty, at the shuttle result or at the two call sites.
- **F2 — LOW, consistency.** `internal/shuttleengine/apierrorretry.go:347-350`.
  - A failed `sendWithin` logs at Info on every poll tick until the deadline, because `pending` stays true.
  - The same loop's other failing sends use `run.failures.Fail` (once per episode) or Debug while the failure persists (`wait.go:541-546`, `wait.go:578-582`). PATTERN-spawn-observability asks a loop log to follow its Level policy.
  - Fix: log the first failure at Info through `run.failures.Fail`, `Recover` on success, or drop to Debug on repeats, as `handleGatedBoundary` does.
- **F3 — LOW, design.** `internal/shuttleengine/apierrorretry.go:312-353`.
  - A usage-limit StopFailure is treated like a transient server error. It is re-sent three times over about 210s and then handed on as `api-error`.
  - The held-turn path deliberately sends no notice while the account is limited (`wait.go:330-332`), because the parent cannot act until the reset.
  - The settler will therefore end `api-error` and notify the parent of a stop the parent cannot clear.
  - Evidence: `wait.go:330-332`; `usageLimited` is not consulted in `retryAPIError`.
  - Fix: either skip the re-sends and end the run at once with the limit text, or leave this as a documented bound in the `retryAPIError` comment. Coverage of the case belongs to the `code-tests` lens.

No tool calls were made.
