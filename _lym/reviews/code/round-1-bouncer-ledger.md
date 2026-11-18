---
round: 1
ledger:
  - key: contained-test-missing-keep-marker
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: fragment-comments-claim-to-open-every-steer
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised two findings, both consistency-class.
F1 (BLOCKING) maps to `contained-test-missing-keep-marker`: `TestRows_SteersOpenWithWholeCommandRefusal` needs a `//testtiming:keep` marker.
F2 (LOW) maps to `fragment-comments-claim-to-open-every-steer`: three fragment comments in `rows.go` still say they open every steer.
Both fixes are local and fully specified by the finding text.
