---
round: 1
ledger:
  - key: verify-omits-cmd-lyx
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised one finding, F1: the plan's `## verify:` untagged line does not run `./cmd/lyx`, whose tests scan card 3's prose files and check the `Long` text of the commands cards 1 and 2 rewrite.
The fix is a one-line addition to the verify block.
No other finding was raised.
