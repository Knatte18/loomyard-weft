---
round: 1
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
    rounds: [1]
    status: open
---

Round 1's reviewer raised seven findings, F1 through F7, and the fixer's edits closed all of them before judgment.

- F1: every return to `idle` now retires the turn ends and the context reading the cycle already saw.
- F2: the smoke test's threshold is set out of reach, so the manual `cycle` is its one trigger.
  The departure from the acceptance bullet's "low threshold" is ratified.
- F3: the restart tests are scoped per phase.
- F4: `Run` retries the watch lock.
- F5: the Agent-tool override is a pure, tested function, and D16 licenses it.
- F6 and F7: the two ImpactSummaries are corrected.

The one open item is the judge's own finding against card 8.
The resume prompt is rendered and sent only after `/clear`, and a failed render or `Send` is never retried by the same watcher, because D10's landed check runs only on a restarted watcher's first tick.
A cleared session can therefore sit with no resume prompt until the phase times out.
The fix is to render the resume prompt before clearing, and to rerun the landed check on later ticks whenever a phase's injection errored.
