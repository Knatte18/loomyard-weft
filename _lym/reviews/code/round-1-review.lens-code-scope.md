```yaml
lens: code-scope
outcome: reported
```

Lens code-scope: tracing the diff's changes to the board entry's goal points. The record has no `Added after Discussion` entries, so the goal points are the only decisions.

**Per change:**
- **C1, start prompt busy retry.** `sendStartPrompt`, `startPromptBusyRetries`, its call in `loadSkillsThenPrompt` and the doc.go and run.go doc lines. Traces to G3 (shuttle retries a "retry once idle" refusal itself) and extends G2.
- **C2, failure-pane capture.** `keepFailureCapture`, its call before `abandonStartup`, the `lastStartupCapture` doc line and the doc.go line. Traces to G1: it makes `startup-capture.txt` hold the screen the idle reading judged.
- **C3, lost-prompt retype.** `retypeLostPrompt`, the `promptSentAt` and `lostPromptRetyped` fields, the `DeliverPendingPrompt`/`typePendingPrompt` split, the `Wait` tick call and the hold.go and doc.go docs. Traces to G4.
- **C4, `promptSentAt` set after a start's prompt lands** (run.go:877-879). Forced by C3.
- **C5, tests.** The new row and capture assertion in `TestStartup_SendFailuresTearDown`, the `startSendEngine.idleWhen` hook, `TestWait_RetypesAPromptLineThatStartedNoTurn` and `turnStartReed`. Forced by C1, C2 and C3.
- **C6, commit 2054c690.** Rebreaks the comments one sentence per line. Forced by the comment lint on C1–C3.

**Per decision:**
- **G1, check the idle detection against the saved `startup-capture.txt`:** C2 only, partly built. The detection itself is neither examined nor changed.
- **G2, poll for the input box for a bounded time before declaring not idle:** this already existed before the diff. `awaitIdleSession` polls for the whole `send_ready_timeout_s` window (send.go:209-232). C1 adds further windows.
- **G3, shuttle retries a "not idle" refusal instead of killing the strand:** C1.
- **G4, find the cause of a review strand starting without its prompt turn, also after a restart:** C3 and C4. The attach path sets `promptSentAt` through `typePendingPrompt`, so a restart's delivery also gets the retype.
- **Unrequested scope:** none. Every change traces to a goal point or is forced by one.

**Findings:**

1. **MEDIUM, scope.** Location: `internal/shuttleengine/run.go:892-900` (`sendStartPrompt`). G1, the idle-detection mismatch, is not addressed.
   - The entry asks for the "no input box" reading to be checked against the screen it describes: an empty `❯` box and the footer under a `✻ … · done` line. Both reported attempts failed with that reason.
   - The diff neither examines nor changes that classification. It retries the same reading for up to two more `send_ready_timeout_s` windows. If the classifier misreads this layout every time, the start still dies, just later. `summary.md` says so itself: "The exact cause of the missed box in #137 is still unknown."
   - The two named run directories are gone (checked), but the entry's own description of the screen remains as evidence for checking the box reading.
   - Evidence: board body, "Possible causes … the check does not recognize this layout"; `_lym/landing/summary.md`, "The saved capture now shows the failure"; `send.go:228`, the `NoInputBox` hold.

Tool calls:
1. `ls` of the two `.lym/shuttle/&lt;id&gt;` run directories the entry names, to verify the summary's claim that they are gone. They were not in my context.
2. grep for relaunch paths in `shuttleengine` and `reedengine`, to judge whether G4's "also after a restart" covers a reed relaunch as well as an attach. My context held only the attach path.
3. grep for `--resume` and session-id handling in `claudeengine` and reed's `lifecycle.go`, to see whether a relaunched strand keeps its prompt turn. Not settled within budget, so I raise no finding on it.
