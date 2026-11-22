---
round: 1
ledger:
  - key: d2-pause-does-not-hold-inner-run
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: cancel-archive-tags-not-on-origin
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: no-check-new-parent-takes-member
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: stale-ref-workaround-outside-procedure
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: partial-finish-vs-refusal-undefined
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: h2-move-note-in-stored-record
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: glossary-move-note-names-writer
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised seven findings.
The two BLOCKING ones concern the D2/D4/D7 hold-and-cancel procedure: the pause used does not stop the inner run, and cancel's archive tags are not yet on origin when it returns.
The MEDIUM design findings add a pre-cancel parent check, move the stale-ref prune into D4, and fix the H2 carrier in a stored record.
The remaining two are consistency fixes: defining partial finish versus refusal, and the glossary wording.
