---
round: 1
ledger:
  - key: shortcut-false-valued-help-word-undocumented
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised one finding (F1): `routeWords` drops a false-valued help or json word, `doc.go` says a json word prints help, and no `TestShortcut_Run` row pins the case.
It is LOW and non-gating.
