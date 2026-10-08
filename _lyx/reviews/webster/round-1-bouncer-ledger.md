---
round: 1
ledger:
  - key: new-tests-omit-parallel-without-global-state-comment
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
  - key: validatefankeys-reads-burler-yaml-with-no-fan-key
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: transcripttokens-dead-code-and-dependent-doc-comment
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: touched-go-files-not-gofmt-clean
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 is the first round, so there is no prior ledger.
Four findings opened: a BLOCKING `PATTERN-test-speed` violation on new tests, a LOW `design` regression in `validateFanKeys`, a LOW dead-code finding in `claudeengine/usage.go`, and a LOW gofmt finding on three touched files.
