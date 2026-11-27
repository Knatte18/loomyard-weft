---
round: 1
ledger:
  - key: heredoc-body-bare-word-line-ends-body
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: shell-c-script-after-heredoc-unread
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: heredoc-delimiter-and-two-operator-gaps-unnamed
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: empty-heredoc-body-row-missing
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: package-doc-reach-statement-contradiction
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: summary-understates-blast-radius-and-false-denial
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 opened every key above; there is no earlier ledger.

The two gating findings are about the reach of the after-heredoc match in `internal/shellguard/rows.go`.
It takes any bare-word line as a heredoc's closing line, which denies commands that edit nothing,
and it does not read a shell -c script that follows a closed heredoc.
The first asks for a different way of finding the closing line,
with a deferral to the run's parent if the fixer judges that unworkable.

The rest are unrecognised delimiter shapes with no comment naming them, a missing empty-body test row,
a package-doc sentence the new reach contradicts, and a change description that understates what the change can break.
