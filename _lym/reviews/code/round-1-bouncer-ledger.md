---
round: 1
ledger:
  - key: pair-remote-runmail-bullet-splits-settle-bullet
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: creel-wire-path-tags-and-droprun-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: caller-set-run-tags-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: pushnotifier-tags-not-asserted
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: seam-filled-check-omits-runnotices-post
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: removepair-doc-paragraph-order
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: pairslug-records-case-ignores-prime
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: runmail-dropper-closure-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised eight findings, all open: one BLOCKING doc-structure slip in `PATTERN-pair-remote.md`,
four MEDIUM test-coverage gaps around the new run and topic tags and the seam check,
and three LOW items (doc comment order, `pairSlug` on a prime's records branch, the untested run-mail dropper closure).
All fixes are local corrections the finding text already fixes.
