---
round: 1
ledger:
  - key: step-fold-test-no-verdict-write-after-fold
    rounds: [1]
    status: resolved
  - key: awaiting-interleave-case-shape
    rounds: [1]
    status: resolved
  - key: card2-impactsummary-history-length-readers
    rounds: [1]
    status: resolved
  - key: battencli-maxstatushistoryentries-stale-comment
    rounds: [1]
    status: resolved
  - key: card4-d9-equal-length-unstated
    rounds: [1]
    status: resolved
---

Round 1 raised five findings against cards 2 and 4, and the fixer resolved all five before this judgment.
F1 added a test in which a no-verdict pause/resume write lands right after a fold, which exercises the carry-forward side of `persistTransient`'s new discriminator and D4's pause-spanning fold.
F2 reshaped the Awaiting-interleave step test to fold before the Awaiting, and stated that the step after the halt is a plain `Step`.
F3 rewrote card 2's ImpactSummary to name the history-length readers (batten's `history_length`, battencli's window) and to state that loomengine is unaffected.
F4 added battencli's `maxStatusHistoryEntries` doc comment as an `Edit:` target, as a doc-only correction.
F5 made card 4's D9 case state the equal-length precondition it depends on.
The judge re-checked the discriminator and the Awaiting resume path against `internal/shedengine/run.go` and found both sound.
Nothing is open.
