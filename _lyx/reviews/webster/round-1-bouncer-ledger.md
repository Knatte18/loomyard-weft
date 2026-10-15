---
round: 1
ledger:
  - key: map-literal-blank-line-split
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised one finding (F1): a stray blank line splits the `kickstart_pack_inline` entry from its sibling map entries in `render.go` and `template_test.go`.
It is a local formatting correction, so it is mechanical.
