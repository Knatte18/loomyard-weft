---
round: 1
ledger:
  - key: short-envelope-conditional-keys-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: steprecord-test-not-gofmt
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: parkmarker-comment-scratch-dir-source
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised three findings, all open.
The short step envelopes' conditional `friction` and `parent_notice` keys lack a test (F1, gating).
`steprecord_test.go` is not gofmt-formatted (F2).
The `ParkMarker` comment names the printed step envelope as `scratch_dir`'s source, which only the recorded full envelope now carries (F3).
