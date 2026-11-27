---
round: 1
ledger:
    - key: loadconfig-doc-bare-config-keys
      rounds:
        - 1
      status: resolved
      class: consistency
      severity: BLOCKING
      fix: mechanical
    - key: pr-session-tests-module-stand-ins
      rounds:
        - 1
      status: open
      class: design
      severity: BLOCKING
      fix: design-wrong
    - key: gate-retry-via-pr-continue-ends-review
      rounds:
        - 1
      status: open
      class: design
      severity: MEDIUM
      fix: design-wrong
    - key: session-stopped-while-wait-continues
      rounds:
        - 1
      status: open
      class: design
      severity: MEDIUM
      fix: mechanical
    - key: failed-gate-mail-recorded-delivered
      rounds:
        - 1
      status: open
      class: design
      severity: MEDIUM
      fix: design-wrong
    - key: ended-review-failed-head-ends-done
      rounds:
        - 1
      status: open
      class: design
      severity: MEDIUM
      fix: mechanical
    - key: pr-continue-admits-any-pr-reviewer
      rounds:
        - 1
      status: open
      class: scope
      severity: MEDIUM
      fix: mechanical
    - key: session-stop-and-resume-untested
      rounds:
        - 1
      status: open
      class: design
      severity: MEDIUM
      fix: mechanical
    - key: wirelanding-pr-review-seams-untested
      rounds:
        - 1
      status: open
      class: design
      severity: MEDIUM
      fix: mechanical
    - key: restepped-error-stops-session
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: design-wrong
    - key: closed-pr-starts-session-to-halt
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: mechanical
    - key: session-behaviours-without-test-rows
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: mechanical
    - key: stale-pr-review-mode-comments
      rounds:
        - 1
      status: open
      class: consistency
      severity: LOW
      fix: mechanical
    - key: shellguard-rows-not-gofmt
      rounds:
        - 1
      status: open
      class: consistency
      severity: LOW
      fix: mechanical
    - key: description-edit-unvalidated
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: mechanical
    - key: merge-danger-blast-radius-narrow
      rounds:
        - 1
      status: open
      class: consistency
      severity: LOW
      fix: mechanical
    - key: stencil-header-wrong-fill-function
      rounds:
        - 1
      status: open
      class: consistency
      severity: NIT
      fix: mechanical
    - key: prompt-understates-text-refresh-delay
      rounds:
        - 1
      status: open
      class: consistency
      severity: NIT
      fix: mechanical
---
Round 1 opens the ledger with every finding of the review.
The gating weight sits in the new PR-Review session path: when the wait stops, resumes and ends the session, how a gate failure reaches the agent and is retried, and how those paths are tested.
Two findings are BLOCKING on PATTERN entries: bare config key names in a doc comment, and new stand-ins for repository modules in the session tests and in `envkit`.
