---
round: 2
ledger:
  - key: guard-never-lifted-when-turn-start-alone-in-tick
    rounds: [1]
    status: resolved
  - key: same-batch-position-test-not-conditioned-on-armed-guard
    rounds: [1]
    status: resolved
  - key: held-wait-not-ended-by-same-batch-turn-start
    rounds: [1]
    status: resolved
  - key: card4-test-names-nonexistent-step-hook
    rounds: [1]
    status: resolved
  - key: observeclocks-first-observation-stamp-unrecorded
    rounds: [1]
    status: resolved
  - key: card2-grep-misreads-routing-bounces-comment
    rounds: [1]
    status: resolved
  - key: turnstartread-reference-offset-ambiguous-when-hold-and-new-turn-end
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised six findings, two BLOCKING, in cards 2 to 6.
Round 2's review settled each of them (turn-start guard lifting and arming, same-batch hold ending, card 4's hook, card 3's stamp, card 2's grep paragraph), so all six are resolved.
Round 2 raised one new LOW consistency finding: card 6's `turnStartRead` names two reference offsets without saying which governs when a held wait is on show and the tick returns a new held turn end.
