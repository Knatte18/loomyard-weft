---
round: 2
ledger:
  - key: card7-shell-refusal-breaks-revival-trigger-test
    rounds: [1]
    status: resolved
  - key: probe-server-bound-false-premise
    rounds: [1]
    status: resolved
  - key: chaingate-shell-row-duplicates-existing-path
    rounds: [1]
    status: resolved
  - key: tmuxkit-configtext-own-card
    rounds: [1]
    status: resolved
  - key: card5-tickerperiodfor-delete-split
    rounds: [1]
    status: resolved
  - key: card5-unchanged-tick-test-reason-wrong
    rounds: [1]
    status: resolved
  - key: card2-file-comment-stderr-claim
    rounds: [1]
    status: resolved
  - key: card5-poll-ceiling-unbounded-by-dormancy-test
    rounds: [2]
    status: open
    class: design
    severity: LOW
  - key: card5-pollmodebydefault-cannot-observe-backoff
    rounds: [2]
    status: open
    class: design
    severity: LOW
---

Round 1 raised seven findings, led by a BLOCKING scope finding on the shell refusal and a MEDIUM design finding on the probe server bound.
Round 2 did not raise any of them again, and its focus verdicts confirm the shell refusal and probe-server cases are handled, so all seven are resolved.
Round 2 raised two new LOW design findings on card 5: the `PollCeiling` bound misses the dormancy test's 20 ms stability windows, and the `PollModeByDefault` assertion cannot tell a backed-off loop from a fixed cadence.
Both are open and non-gating at LOW.
