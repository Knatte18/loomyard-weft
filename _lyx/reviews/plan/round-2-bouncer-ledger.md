---
round: 2
ledger:
  - key: card3-no-tail-rows-unreachable-conditions
    rounds: [1]
    status: resolved
  - key: card4-gate-dismissal-gap-duplicate-test
    rounds: [1]
    status: resolved
  - key: card1-lone-eol-mark-first-send-only
    rounds: [1]
    status: resolved
  - key: card4-stale-testtiming-keep-reason
    rounds: [1]
    status: resolved
  - key: card1-fake-reed-middle-capture-unspecified
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised four findings, one gating (card 3's no-tail rows).
Round 2 did not re-raise any of them, and its focus verdicts show card 3 and card 4 sound, so all four are resolved.
Round 2 raised one new LOW consistency finding on card 1: the fake reed's mid-send capture is unspecified and circular.
