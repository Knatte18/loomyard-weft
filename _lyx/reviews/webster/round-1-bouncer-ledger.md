---
round: 1
ledger:
  - key: template-missing-trailing-newline
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: doc-pins-hold-constants
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised two findings.
F1 (`template-missing-trailing-newline`): removing `fix_start` left `internal/loomengine/template.yaml` without a final newline, so the appended-key rows in `TestLoadConfig_Loads` become part of a comment and test nothing.
F2 (`doc-pins-hold-constants`): `internal/shuttleengine/doc.go` repeats the hold deferral constants' values, which `hold.go` declares.
Both stay open until a later round's fresh review confirms the fixes.
