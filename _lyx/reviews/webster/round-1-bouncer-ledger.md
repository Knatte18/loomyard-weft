---
round: 1
ledger:
  - key: retry-landed-slash-command-dropped
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: paste-buffer-leaks-on-failed-paste
    rounds: [1]
    status: open
    class: design
    severity: LOW
---

Round 1 raised two LOW design findings.
F1 maps to `retry-landed-slash-command-dropped`: `retryLanded` in `internal/shuttleengine/send.go` reads any later turn start as the send going out.
F2 maps to `paste-buffer-leaks-on-failed-paste`: `pasteSubmitArgv` in `internal/reedengine/io.go` deletes its named buffer only when `paste-buffer -d` succeeds.
Neither is gating, so the loop converged in round 1.
