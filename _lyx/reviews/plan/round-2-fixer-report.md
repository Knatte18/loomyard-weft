# Fixer report — plan review round 2

All three findings are fixed.
`lyx loom validate-plan` reports `ok: true` after the edits, with informational `create-new-unit` findings only.

## Changed

- **F1 (BLOCKING)**, failed injections never retried and the resume prompt rendered after `/clear`:
  - `08-orch-cycle-machine.md`: `handoff-requested` renders `RenderResumePrompt` before persisting `clearing`, and a render error aborts to `idle` with no `/clear`.
    The rendered text is kept in state and `resuming` sends it verbatim.
  - `08-orch-cycle-machine.md` gains a confirmation rule: each non-idle phase is saved with `PhaseInjected` false before its side effect and set true once `Send` or `ClearSession` returns nil.
    An injection error is returned from `Tick`.
    Every tick in an unconfirmed phase runs D10's landed check and re-sends at most once per tick, bounded by the phase timeout and card 9's error cap.
    The restart rules are now the same check, run on a restarted watcher's first tick whatever the flag says.
  - `06-orch-state.md` gains `PhaseInjected` and `PendingResume`, and `ResetForFreshLaunch` clears both.
  - Card 8's test list gains cases for a failing resume render, a `Send` error on the handoff and on the resume prompt followed by an idle pane, and a resume `Send` error whose text did land.
- **F2 (MEDIUM)**, a `ClearSession` error still counting a cycle:
  - `08-orch-cycle-machine.md`: `clearing` re-calls `ClearSession` while the clear is unconfirmed, and moves on only after a confirmed clear and a passing probe, or on the timeout.
    `CycleCount` increments on leaving `clearing` with a confirmed clear, so a timeout exit with no clear does not count and a restart's re-clear does not count twice.
  - `06-orch-state.md` defines `CycleCount` by that rule.
  - Test cases: a `ClearSession` error then success gives `CycleCount` 1, a clear that errors until the timeout leaves it unchanged, and the restart-in-`clearing` case asserts one increment.
- **F3 (LOW)**, the handoff instruction rendered after the phase is persisted:
  - `08-orch-cycle-machine.md`: the `idle` phase renders the handoff instruction before persisting `handoff-requested` or clearing the cycle request.
    A render error returns from `Tick` with state and request untouched.
  - Test case: a failing handoff render causes no phase change and no `Send`, and the request stays.

## Deferred

None.
