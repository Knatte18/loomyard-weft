---
round: 1
ledger:
  - key: resume-failed-status-replace-silent
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: resume-status-restore-seam-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: sharedbootstrap-docs-say-start-only
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: summary-claims-resume-never-switches-window
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: resume-short-says-starts-nothing
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 opens every key.
The gating pair concerns the status-pane restore on resume:
a failed replace is swallowed, so the envelope reports no `status_pane`, and no test exercises the real restore.
The rest are text that no longer matches the behavior: the `sharedbootstrap.go` comments and warnings,
the change description's window claim and the `resume` verb's `Short`.
