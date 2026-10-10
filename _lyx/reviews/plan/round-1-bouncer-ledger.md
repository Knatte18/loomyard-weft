---
round: 1
ledger:
  - key: card7-shell-refusal-breaks-revival-trigger-test
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
  - key: probe-server-bound-false-premise
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: chaingate-shell-row-duplicates-existing-path
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: tmuxkit-configtext-own-card
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: card5-tickerperiodfor-delete-split
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: card5-unchanged-tick-test-reason-wrong
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: card2-file-comment-stderr-claim
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised seven findings, all still open.
F1 (BLOCKING): card 7's POSIX shell refusal fails every row of `TestEnsureServerAndSession_RevivalTrigger` in `revive_test.go`, outside the card's targets.
F2 (design, MEDIUM): the `probe-server-bound` Shared Decision assumes the probe's transient server reads no operator config, but it does.
F3 to F7 are LOW or NIT: a duplicated ChainGate row, `ConfigText` granularity, `tickerPeriodFor` deletion granularity, a wrong reason in card 5, and a wrong file-comment claim in card 2.
