---
round: 2
ledger:
    - key: prreview-mail-test-handwritten-seams
      rounds:
        - 1
        - 2
      status: resolved
      class: design
      severity: BLOCKING
      fix: mechanical
    - key: prreview-mail-test-new-standing-fake-uses
      rounds:
        - 1
        - 2
      status: open
      class: decision
      severity: BLOCKING
      fix: design-wrong
    - key: prreview-mail-rows-lost-no-push-and-record-checks
      rounds:
        - 1
      status: resolved
    - key: envkit-reed-strand-answers-still-handwritten
      rounds:
        - 1
      status: resolved
    - key: tmuxrec-paste-text-placeholds-every-binding
      rounds:
        - 1
      status: resolved
    - key: tmuxrec-stdin-binding-untested
      rounds:
        - 1
      status: resolved
    - key: tmuxrec-bare-shell-fallback-untested
      rounds:
        - 1
      status: resolved
    - key: stand-ins-inventory-lists-removed-fakes
      rounds:
        - 1
      status: resolved
    - key: merge-danger-blast-radius-too-narrow
      rounds:
        - 1
      status: resolved
---
Both open keys concern stand-ins in `internal/loomcli/prreview_mail_integration_test.go`.

The hand-written seams were fixed in part after round 1:
the seams round 1 named now come from the wiring over the real pair,
while the dirty-path, missing-commit, task-commit, pause and record-watch seams are still canned `envkit` closures.

The new uses of standing fakes, with the hand-written change description, have been open since round 1.
No added entry in the decision record settles them, so they wait on the run's parent.

The remaining round-1 keys, on lost assertions, `envkit`, `tmuxrec` and the two documents, are resolved.
