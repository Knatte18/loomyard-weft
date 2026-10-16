---
round: 1
ledger:
  - key: verify-omits-tools-sandbox
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: watchdog-nil-branch-ctx-guard-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: down-hubpath-guard-unreachable
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: truncation-test-should-be-a-row
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: looping-rows-vs-empty-logs-assertion
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: stop-affirmative-row-passes-on-wrong-exit
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised six findings, none BLOCKING.
F1 (`verify-omits-tools-sandbox`) is the one gating finding: card 6 edits embedded sandbox suite files, but `## verify:` never runs `./tools/sandbox`.
F2 is a MEDIUM `scope` finding about an untested guard on the watchdog's nil-return branch.
F3 to F6 are LOW: an unreachable hub-mode guard, a per-helper test that belongs as a table row, an empty-logs assertion that the new looping rows contradict, and a row that can pass on a wrong exit.
