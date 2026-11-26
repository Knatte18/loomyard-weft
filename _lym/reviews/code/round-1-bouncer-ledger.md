---
round: 1
ledger:
    - key: cancel-rerun-refuses-with-only-origin-records-branch
      rounds:
        - 1
      status: resolved
      class: design
      severity: BLOCKING
      fix: design-wrong
    - key: fabric-docgo-describes-batten-cancel
      rounds:
        - 1
      status: resolved
      class: consistency
      severity: BLOCKING
      fix: mechanical
    - key: records-archive-origin-condition-dropped
      rounds:
        - 1
      status: open
      class: design
      severity: MEDIUM
      fix: design-wrong
    - key: pair-gone-naming-narrowed-unrecorded
      rounds:
        - 1
      status: open
      class: decision
      severity: MEDIUM
      fix: design-wrong
    - key: pending-record-never-cleared-when-pair-gone
      rounds:
        - 1
      status: open
      class: design
      severity: MEDIUM
      fix: design-wrong
    - key: pair-remote-background-four-argument-remove
      rounds:
        - 1
      status: open
      class: consistency
      severity: MEDIUM
      fix: mechanical
    - key: remove-held-lock-refusal-rerun-untested
      rounds:
        - 1
      status: open
      class: design
      severity: MEDIUM
      fix: mechanical
    - key: pairteardown-rerun-forwarding-untested
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: mechanical
    - key: summary-blast-radius-and-finish-claim
      rounds:
        - 1
      status: open
      class: consistency
      severity: LOW
      fix: mechanical
---
First round, so every finding is new and open.

The core problem is that a removal record now names the batten cancel as its re-run in states where the cancel does not behave as the replaced `lym fabric remove` did.
It refuses when only the origin records branch is left, the origin condition was dropped from the records-archive way forward, and nothing retires the record once the pair is gone.
`doc.go` and the change description repeat the claim that the cancel always finishes.

The rest is a stale pattern background, two missing tests for the forwarding of the caller's re-run, and one narrowing of the board entry that is the parent's to record or refuse.
