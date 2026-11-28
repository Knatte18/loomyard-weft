---
round: 1
ledger:
  - key: prreview-mail-test-handwritten-seams
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: prreview-mail-test-new-standing-fake-uses
    rounds: [1]
    status: open
    class: decision
    severity: BLOCKING
    fix: design-wrong
  - key: prreview-mail-rows-lost-no-push-and-record-checks
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: envkit-reed-strand-answers-still-handwritten
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: tmuxrec-paste-text-placeholds-every-binding
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: tmuxrec-stdin-binding-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: tmuxrec-bare-shell-fallback-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: stand-ins-inventory-lists-removed-fakes
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: merge-danger-blast-radius-too-narrow
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 is the first round, so every key is new and open.

The gating weight sits on test stand-ins in `internal/loomcli/prreview_mail_integration_test.go`:
hand-written seams that the test's real pair could supply, and new uses of two standing fakes.
The second of these waits on a ruling by the run's parent.

The rest are lost assertions in the moved mail rows, narrow or untested behaviour in `envkit` and `tmuxrec`,
and two documents that no longer match the diff.
