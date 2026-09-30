---
round: 2
ledger:
  - key: return-to-idle-retires-cycle-events
    rounds: [1]
    status: resolved
  - key: smoke-low-threshold-cannot-settle
    rounds: [1]
    status: resolved
  - key: clearing-restart-tests-contradict-rule
    rounds: [1]
    status: resolved
  - key: watcher-live-probe-starves-spawn
    rounds: [1]
    status: resolved
  - key: agent-tool-override-untested
    rounds: [1]
    status: resolved
  - key: card4-impactsummary-misses-clone
    rounds: [1]
    status: resolved
  - key: card12-impactsummary-contradicts-edits
    rounds: [1]
    status: resolved
  - key: failed-resume-injection-never-retried
    rounds: [1, 2]
    status: resolved
  - key: clear-error-counts-cycle
    rounds: [2]
    status: resolved
  - key: handoff-render-after-persist
    rounds: [2]
    status: resolved
  - key: failing-probe-confirms-injection
    rounds: [2]
    status: open
---

Round 1's seven reviewer findings were all fixed before judgment.
The judge's own round-1 finding was that a failed injection is never retried by the same watcher.

Round 2's reviewer raised that finding again as F1, along with F2 (a `ClearSession` error still counted as a cycle) and F3 (the handoff rendered after the phase was persisted).
The fixer closed all three:

- card 8 renders both stencils before their phases are persisted;
- card 6 gains `PhaseInjected` and `PendingResume`;
- every tick in an unconfirmed phase runs D10's landed check;
- `CycleCount` moves only on leaving `clearing` with a confirmed clear.

One gap remains, recorded as `failing-probe-confirms-injection`.
The landed check counts any failing `SessionIdle` probe as proof that the injection landed, while D10 counts only a turn in progress.
After a `clearing` timeout the probe was already failing before the resume `Send` errored, so the prompt is confirmed without ever being sent.
The cleared session is stranded, which is the harm the retry was added to prevent.
The fix: only a turn end confirms an injection, and a failing probe defers the check to a later tick.
