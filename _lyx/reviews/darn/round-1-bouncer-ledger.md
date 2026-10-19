---
round: 1
ledger:
  - key: help-escape-matches-option-values
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: quoted-option-values-break-subcommand-skip
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: backslash-continuation-splits-git-call
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: attr-source-value-not-skipped
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: valued-option-forms-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised five findings on the mutating-git segment classifier in `internal/burlerengine/cluster.go`.
Three are gating design gaps where a mutating git call passes: the help escape, quoted option values and backslash continuations.
Two are LOW scope findings: the missing `--attr-source` option and missing test rows for valued-option forms.
