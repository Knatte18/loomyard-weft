---
round: 3
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
    rounds: [2, 3]
    status: resolved
---

Round 1's seven reviewer findings were fixed before judgment.
The judge's round-1 finding was that the same watcher never retries a failed injection.
Round 2 closed that finding along with `clear-error-counts-cycle` and `handoff-render-after-persist`, but its fix counted any failing `SessionIdle` probe as proof that an injection landed.
That left `failing-probe-confirms-injection` open: after a `clearing` timeout, a resume `Send` error could be confirmed without ever being sent, stranding the cleared session.

Round 3's reviewer raised the same gap as F1, and the fixer closed it.
Only a turn end past the entry offset now confirms an injection, and a failing probe keeps waiting with the flag unchanged.
A passing probe with no turn end re-sends.
A restarted watcher saves `PhaseInjected` false before its first check.
Card 6's field definition and four new card 8 tests match the new rule.
No entry is open, and the plan is approved at round 3.
