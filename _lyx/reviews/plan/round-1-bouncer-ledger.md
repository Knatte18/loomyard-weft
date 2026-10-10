---
round: 1
ledger:
  - key: card1-misses-run-resume-test-startedfresh
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
  - key: card1-tick-table-on-old-rule
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: card1-hold-test-duplicates-existing-coverage
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: card4-newverifygate-error-path-undecided
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
  - key: cards10-12-guard-bound-unstated
    rounds: [1]
    status: open
    class: scope
    severity: LOW
---

Round 1, first ledger.
Card 1 carries three findings: a test file that stops compiling once `Run.startedFresh` is dropped (F1), a tick-level shell table still asserting the old rule (F2), and a reworked test that repeats existing coverage (F3).
Card 4 leaves the error path of `NewVerifyGate` undecided (F4).
Cards 10 to 12 leave the bound on the weakened guard unstated (F5).
