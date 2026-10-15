---
round: 1
ledger:
  - key: external-sender-name-formed-outside-agentname
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
---

Round 1 raised one finding: `ExternalSender` in `internal/creelengine/seam.go` built the `<shortname>:EXTERNAL` name by string concatenation instead of `agentname.FormatProgram`.
The review's fix text fixes the result (use `FormatProgram`, return `(Sender, error)`, propagate, add one table row), so the fix is mechanical.
