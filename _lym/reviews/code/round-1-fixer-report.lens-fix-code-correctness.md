```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fix-code-correctness lens on the F1–F3 code and test changes (`loadturn.go`, `abandonLoad`, `fakes_test.go`, `startup_test.go`).

**Findings**

1. **LOW**, `TestStartup_LoadTurnAPIError`, the new offset assertions (`run.offset`, `persisted.PromptOffset`).
   - Summary: the assertions cannot fail on a hold-specific defect.
   - Evidence: `loadSkillsThenPrompt` sets `run.offset` and `PromptOffset` to `eventsSize` right before the prompt, whatever `readLoadTurn` did to the offset during the hold. Dropping `run.offset = newOffset` in `readLoadTurn`, or re-reading the re-typed turn, would still pass.
   - The assertions only restate what `TestStartup_SkillLoadTurns` already pins, so F2 point (3) is covered nominally rather than for the hold.

**Traced and correct, no finding**

- **`heldDeadline`:** it is only reached when `watch.open` is true. `began` is always at or after `since` there, because `turnStartRead` and the retype both reset `began`. A hung re-typed turn gets a deadline of `run.deadline + (began - since)`, which is fixed, so it expires. Using `now - since` would never have expired.
- **Open-turn row:**
  - A hung turn with `OpenTurns` reads START, so `open` is true and the loop runs until the one-hour deadline.
  - `loadDeadlineError` then goes through `settleLoadTurn` (the `err != nil` path) to `abandonLoad`, which gives `OutcomeTimeout` and an error naming `outcome "timeout"`.
  - With the old 15-minute skip the prompt would have been typed, so `wantTyped: ""` fails on the old code.
- **`abandonLoad`:** `APIError` is only set for a rejected model. The existing model-rejected test still holds.
- **Callers:** `loadSkillTurn` is called only from `settleLoadTurn`, and the unchanged signature keeps it working.
- **Carries row:**
  - Startup consumes one `Status` call, so START lands in iteration 1's `Status` call and is read at iteration 2, before the re-type.
  - If `turnStartRead` stopped clearing `pending`, the re-type would put `LOAD:a` into the typed texts and fail the row.
  - If START landed during startup instead, the same read would still clear `pending`.
- **Busy row:** the failed send is unrecorded, since `LoadSendErrs` is checked before `fakeReed.SendText`. `retypeAt` moves by the doubled 60 s, so elapsed is 90 s or more. A non-doubling `retypeAt` would give about 60 s and fail `wantHeld: 3*apiErrorRetryBackoff`.
- **Ceiling row:** the waits sum to 17.5 minutes, or 31.5 minutes if the doubling were uncapped. The 25-minute limit separates the two.
- **Minor:** the ceiling row hard-codes `30*time.Second` where `apiErrorRetryBackoff` would track the constant.

Tool calls made: none; my context already held everything this lens needed.
