```yaml
lens: code-correctness
outcome: reported
```

Task: run the code-correctness lens over the diff, listing each changed function or branch, the case I ran, and whether it was correct.

**List**

- `SessionFold.settle`, unproven liveness plus a turn start (`sessionstate.go:283`):
  - Case: `LivenessUnproven` with a single `turnStart`.
  - Walk-through: the session-end, dead and unreadable cases all miss. `hasNewest` and `hasDerived` are true, so it reaches `default` and settles `derived`, which is busy/turn.
  - Result: correct.
- `settle`, unproven liveness plus a process-ending session end:
  - Case: the newest signal is a session end with `EndsProcess`.
  - Walk-through: the first case settles `derived` (dead, with the end's reason), as before.
  - Result: correct.
- `settle`, dead process:
  - Case: `LivenessDead`.
  - Walk-through: it settles dead/process-gone, which ranks above the derived state.
  - Result: correct.
- `settle`, unreadable events file:
  - Case: `EventsUnreadable` together with unproven liveness.
  - Walk-through: it settles unknown/events-unreadable. The history gets busy first, then unknown, which the changed test row expects.
  - Result: correct.
- `settle`, empty signal history:
  - Case: no signals, unproven liveness.
  - Walk-through: it settles unknown/no-signal. A fold whose signals produced no derived state settles unknown/no-state.
  - Result: correct.
- `settle`, Codex interrupt:
  - Case: session start, turn start, then `Interrupted` with `InterruptAt`.
  - Walk-through: `applyTranscriptMarkers` sets derived to idle-stalled/interrupt before `settle`, and `settle` takes the derived state. The history is idle-fresh, busy, idle-stalled, matching the new test row.
  - Result: correct.
- `settle`, engine with no `SessionProber`:
  - Case: the zero-value `Liveness` is unproven, so these engines now read the derived state where they used to read unknown/liveness-unproven.
  - Result: correct and intended. `readSessionFacts` still sets liveness only when a prober exists and a session id is known.
- `settle`, Claude with a transiently unproven liveness:
  - Case: `processLiveness` returns unproven for a reused pid, an unreadable registry, or an entry without `procStart`.
  - Walk-through: the state now follows the hook signals. A Claude process that died mid-turn behind a live pane, with a reused pid, reads busy/turn. `judgeReadiness` then holds sends with "a turn is running". Before, the unknown reading made readiness fall back to the pane.
  - Result: not wrong against the decision record, which says unproven only stops the state from claiming the process is alive. It is a narrow regression risk: `sessionGone` still needs a proven dead reading or a process-ending session end, so nothing else catches it.
- `Spec.validate`, hidden anchor (`spec.go:220`):
  - Case: `Anchor == AnchorHidden`.
  - Walk-through: the earlier checks pass, the empty anchor is not defaulted, and the hidden check returns an error naming why and own-window. It runs inside `Run`/`Start` at `run.go:559`, the only caller.
  - Result: correct.
- `Spec.validate`, empty anchor:
  - Case: `Anchor == ""`.
  - Walk-through: it defaults to own-window and passes the hidden check.
  - Result: correct.
- Attach path (`attach.go:415`):
  - Case: a spec with the hidden anchor.
  - Walk-through: Attach does not call `validate`, so a hidden spec is not refused there. Attach never relaunches or moves a strand, so no hidden strand starts.
  - Result: correct, and matches the summary's note that older records may carry a hidden anchor.
- `checkLivenessTick` (`wait.go:1405`) and Attach's candidate verdict (`attach.go:559`) hidden branches:
  - Case: an old run record with a hidden anchor.
  - Walk-through: they classify it as before.
  - Result: correct, but unreachable for new runs.
- `lym shuttle run` anchor check (`run.go:122`):
  - Case: `--anchor hidden`.
  - Walk-through: it is refused before `c.runner` is touched and before the prompt file is read.
  - Result: correct.
  - Case: `--anchor ""`.
  - Walk-through: refused, as before.
  - Result: correct.
  - Case: `--anchor sideways` or `overview`.
  - Walk-through: refused, but with the hidden-specific text "reed starts no command for a hidden strand".
  - Result: finding F-A.
- `tuiReed.frame` and `OperatorActive` (test fakes):
  - Case: the `working` flag, set once in the gate test's `onSleep`.
  - Walk-through: the pane shows "working" from then on and `OperatorActive` is bounded by `activeUntil`. The one-way flag matches the single writer-turn start that row models.
  - Result: correct.

**Findings**

- F-A: LOW, consistency. `internal/shuttlecli/run.go:123`. The refusal for any non-own-window value, including `sideways` and `overview`, says "reed starts no command for a hidden strand". For a value that is not hidden, that cause is wrong. The text should name the hidden cause only when `anchor == AnchorHidden`. The `AnchorUnknown` test row (`cli_test.go:124`) only asserts "own-window", so it does not catch this.
- F-B: LOW, design. `internal/shuttleengine/sessionstate.go:283`. An unproven Claude liveness (pid reuse, unreadable registry) behind a dead process now keeps a stale busy/turn state. `readiness.go:99` then holds sends as "a turn is running" where it used to fall back to the pane. The record accepts this ("unproven only stops the state from claiming alive"), so no fix is demanded. I raise it only as a residual risk.

Outside my lens, for the other lenses: `internal/battencli/wire_seams_test.go:415` still sets `wantState, wantCause = "unknown", "liveness-unproven"` as the fallback when `proc.StartTime` is not ok. That cause no longer exists, so the fallback branch is wrong on a platform without a process start time. The live-registry setup is also no longer needed.

Tool calls: none. My inherited context already held everything I needed.
