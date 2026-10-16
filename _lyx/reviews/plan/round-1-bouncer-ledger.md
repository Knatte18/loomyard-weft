---
round: 1
ledger:
  - key: restored-prompt-clear-undercounts-blank-rows
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: styled-capture-trailing-cells-differ-from-plain
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: daemon-log-open-failure-wording
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: mechanical
  - key: interrupt-restoring-untracked-guid-refusal-unreached
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised four findings, none seen before.
The gating one is the restored-prompt clear in card 4, which counts only interior rows with text and so leaves a prompt with a blank middle row partly uncleared; its fix changes the counting contract in cards 4 and 5.
The other three are local corrections: trimming trailing cells in the escape-stripped styled capture (card 2), naming the log-could-not-be-opened case in the give-up error (card 12), and adding a test row that reaches the untracked-guid refusal (card 5).
