```yaml
lens: code-tests
outcome: reported
```

**Per-behaviour list (test that pins it, or none)**

1. **API-error turn end → load re-typed after the 30s back-off, prompt after it:** pinned by `TestStartup_LoadTurnAPIError` row 1 (`startup_test.go:1715`). The expected typed text `LOAD:a|do the task` and the `wantHeld` check both fail if the retype or the ordering breaks.
2. **A turn past its timeout that then ends on an API error is not skipped (the incident path):** pinned by row 2 (50ms timeout, `LateEnds` 8, `OpenTurns`). Without the open-turn extension the skill would be skipped, the prompt typed at once and the typed text would differ.
3. **Transcript-marker turn end counts as an API error (`TurnEndActivity` path in `readLoadTurn`):** pinned by row 3 (`apiErrorLine`).
4. **Doubling back-off and one parent notice at the second error:** pinned by row 4 (`wantHeld` 7×, `wantNotices` 1). The 5-minute ceiling (`loadHoldBackoffCeiling`) is **not pinned**, because the row stops at 4× the base.
5. **A retry (typed) turn that the API ends spends no retry:** pinned by row 5.
6. **A pane that dies during the hold is a died startup:** pinned by row 6.
7. **Open turn that never ends is skipped at `loadTurnOpenBound`:** only weakly pinned by row 7. That row sets `wantFailed`, so the test returns before any elapsed-time check, and nothing asserts the 15-minute bound or that the prompt goes out after the skip. The existing hang row (`startup_test.go:1594`) already covers the no-turn-start timeout skip.
8. **Model rejected ends the start:** pinned by `TestRun_LoadTurnModelRejectedEndsTheStart` for the outcome `OutcomeAPIError`, `Result.APIError`, `NotStarted`, nothing typed and one strand removal.
9. **Run deadline moved by the hold (`endLoadHold`):** pinned on the success path only (`run.deadline.Sub(run.createdAt)` in the success branch). The check is a loose `>=`.
10. **Notifier line:** pinned for the count only (rows 1–5). Nothing covers the interactive-run skip, a nil notifier or a notifier error being only logged.

**Findings**

- **MEDIUM, design, `loadturn.go:67-73` and `loadturn.go:112-130`:** no test covers a turn start read during the back-off (an operator or revive carried the session on). `turnStartRead` resetting `pending` and `began` is what stops a duplicate load line from being typed; no row would fail if it broke. Evidence: no row in the table writes a turn start during a back-off, since `OpenTurns` only writes one for hanging or late turns.
- **MEDIUM, design, `loadturn.go:120-123`:** no test covers the busy-session retype, where `ErrSessionBusy` moves `retypeAt` to the next back-off and keeps holding. Existing `ErrSessionBusy` rows (`startup_test.go:1981-1985`) cover only the first typed load and prompt-delivery sends, not the retype.
- **MEDIUM, design, `loadturn.go:98-131` with `run.go:831-843`:** no test asserts the run offset or the persisted `PromptOffset` after a hold, which is what keeps `Wait` from reading the re-typed load turn's start and end as the prompt's. The rows assert only the typed texts, so an offset regression passes.
- **LOW, design, `loadturn.go:124-125`:** a retype send error other than busy (returns an error, so a died start) has no row, and nor does a pause covering the retype send.
- **LOW, design, `loadturn.go:217-228`:** the `abandonLoad` branch that returns the run when every output file already exists (`OutcomeDone` from `classifyDeadlineExpiry`) has no test. The rejected-model test covers only the api-error branch.
- **LOW, design, `loadturn.go:204-212`:** `endLoadHold` is not asserted on the died, error or model-rejected return paths. The deadline-shift test runs only on a successful start.
- **LOW, consistency, `startup_test.go:1771-1778`:** row 7 ("open turn never ends") sets `wantFailed: true` for a skip path and returns before the elapsed check. It pins neither the open bound nor the skipped skill's continuation, and it overlaps the existing hang row at `startup_test.go:1594` for the skip itself (test-economy overlap).
- **LOW, design, `loadturn.go:20`:** the `loadHoldBackoffCeiling` cap is untested (item 4).
- **NIT:** the `cmd/lym/daemonwakeups_test.go` list entry needs no test of its own, because the scan test is the guard.

**Tool calls (1):** a grep of `startup_test.go` for hang, timeout, skip and `ErrSessionBusy` rows, because the manifest held only the diff hunks of that file and I needed to judge overlap with the existing skill-load rows.
