```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, I looked for the test row that would fail if that behaviour broke.

**Per behaviour, the test that pins it:**

1. **A start prompt held busy is sent again with a fresh idle wait** (`run.go:892` `sendStartPrompt`). Pinned by the `TestStartup_SendFailuresTearDown` row "a prompt held busy for one send window is sent again" (`startup_test.go:2353`). Without the retry, `Start` would tear down and `wantStarted` would fail.
2. **The retry cap, `startPromptBusyRetries`, and the last busy hold returned as is** (`run.go:896`). Nothing pins it. The row "a prompt delivery held past the send window by an active operator" ends busy on every send, because `fakeReed.ActiveQueue` repeats its last value (`fakes_test.go:223-228`). But it asserts only `wantIs` and the died record, so it passes with 0, 2 or unbounded retries.
3. **The deadline stop of the retry loop** (`run.go:896`, `!run.clock.Now().Before(run.deadline)`). Nothing pins it. No row lets a busy send reach the run deadline (`fastConfig` `RunTimeoutMin: 5`, `fixture_test.go:22`) or asserts how many sends happened.
4. **Only a `*SessionBusyError` is retried; `ErrForeignBoxText` and `ErrSentTextInDraft` end the start.** Nothing pins it. The row "a prompt delivery that finds a foreign box" (`startup_test.go:2350`) checks only that the error is `ErrSessionBusy` and the died record, not how many pastes ran. A retry widened to `errors.Is(err, ErrSessionBusy)` would paste again and still pass.
5. **`keepFailureCapture` saves the pane at the failure on a failed prompt delivery.** Pinned by the row "a prompt delivery that does not land", through `wantCaptureHolds: prompt` (`startup_test.go:2347`, `2395-2403`). The boot capture can't hold the prompt, because `skillReed.CapturePane` echoes `SendTextCalls`, so the assertion tells the two apart.
6. **`keepFailureCapture` takes the styled capture for a `StyledBoxReader` engine** (`run.go:908`). Nothing pins it. `startSendEngine` is not a `StyledBoxReader`.
7. **`keepFailureCapture` keeps the last startup capture when the capture fails** (`run.go:911`). Nothing pins it.
8. **A start sets `promptSentAt`, so `Wait` on the `Start` handle retypes a lost line** (`run.go:877-879`). Nothing pins it. `TestWait_RetypesAPromptLineThatStartedNoTurn` reaches the retype only through `Attach`, and a search of the tests for `promptSentAt`, `typed it again` and `started no turn` finds nothing. This is the main #136 path, a fresh start whose submitted line is dropped, and deleting those lines fails no test.
9. **`retypeLostPrompt` retypes once when no turn start follows within the submit window** (`hold.go:189`). Pinned by the row "a line no turn follows is typed once more" (`attach_test.go:43`).
10. **The retype happens at most once per handle** (`lostPromptRetyped`). Pinned by the same row: the timeout of three submit windows leaves room for a third typing at two windows, and `wantTyped` holds two.
11. **A turn start past the offset ends the watch.** Pinned by the row "a line whose turn starts is typed once" (`attach_test.go:44`).
12. **The retype waits a full submit window after the line went out** (`hold.go:198`, the `&lt; submitConfirmTimeout` term). Nothing pins it. With that term removed, row 1 still gets exactly two typings, only on the first tick that follows the attach typing, and row 2 is unchanged.
13. **No retype while a pause covers the run** (`hold.go:190`). Nothing pins it.
14. **No retype for an engine that does not parse session signals** (`hold.go:193`). Nothing pins it.
15. **`DeliverPendingPrompt` still latches `promptDelivered` after the split into `typePendingPrompt`.** Pinned by `TestAttach_DeliversTheDeferredPrompt` (`attach_test.go:1074-1102`), which expects one typing per row, and by the existing held-turn delivery tests.

**Findings:**

- **MEDIUM, design, `internal/shuttleengine/run.go:877`.** The start-path retype has no test. `promptSentAt` set by `loadSkillsThenPrompt`, and the `Start`→`Wait` retype of a dropped line that it enables, fails no test when removed. Evidence: `run.go:877-879`; `attach_test.go:69` drives only `Attach`; the test search finds no other caller.
- **MEDIUM, design, `internal/shuttleengine/run.go:896`.** The busy-retry cap and the deadline stop have no test. A row whose idle wait always holds asserts no send count, so an unbounded or zero-retry loop passes. Evidence: `startup_test.go:2349` and `fakes_test.go:223-228`, where the active queue repeats its last value.
- **LOW, design, `internal/shuttleengine/run.go:895`.** Nothing pins that only a `*SessionBusyError` is retried. The foreign-box row counts no pastes, so a retry on paste-time `ErrSessionBusy` sentinels, which would type the line twice, passes. Evidence: `startup_test.go:2350`; `send.go:591`.
- **LOW, design, `internal/shuttleengine/hold.go:190`.** The pause guard, the session-signal-parser guard and the submit-window wait of `retypeLostPrompt` have no test. Removing any one of them leaves `TestWait_RetypesAPromptLineThatStartedNoTurn` green. Evidence: `hold.go:190-198`; `attach_test.go:43-44`.
- **LOW, design, `internal/shuttleengine/run.go:906`.** Neither the styled-capture branch nor the capture-failure fallback of `keepFailureCapture` has a test. Evidence: `run.go:908-912`; `startSendEngine` (`startup_test.go:2130-2175`) implements no `StyledBoxReader`.

Tool calls:
1. `send.go:620-660`, the `fakeReed` `ActiveQueue` handling and `fastConfig`: to check which errors are `*SessionBusyError` after a paste, whether the active queue repeats, and the run timeout. The manifest held none of these spans.
2. A search of the tests for `promptSentAt`, the retype log text and pause rows in `TestAttach_DeliversTheDeferredPrompt`: to confirm no existing test covers the start-path retype. The manifest did not cover the other test files.
