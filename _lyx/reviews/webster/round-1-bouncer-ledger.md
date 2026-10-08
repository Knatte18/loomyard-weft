---
round: 1
ledger:
  - key: serial-tests-no-named-reason
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
  - key: unsent-reprompt-cap-not-in-plan
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: send-go-header-narrower-than-file
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised three findings.
The BLOCKING one is two shuttleengine tables (`TestStartup_SendFailuresTearDown`, `TestRun_Send_ConfirmsSubmission`) that run serially with no `t.Parallel` and no comment naming global state.
The two LOW ones are an unsent re-prompt count cap in `wait.go` that the plan does not carry, and a `send.go` header narrower than the file's contents.
