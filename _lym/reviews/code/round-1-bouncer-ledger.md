---
round: 1
ledger:
  - key: body-flag-names-raw-literals
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: padded-path-trim-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: directory-and-empty-body-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: refusal-path-unquoted
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
---

First round, no prior ledger.
F1 asks for named constants for the `body` and `body-file` flag names in `internal/selfreportcli/cli.go`.
F2 asks for a padded-path refusal row, F3 for directory and empty-body rows, and F4 for a quoted path in the refusal.
Every fix is local and fully specified by its finding text.
