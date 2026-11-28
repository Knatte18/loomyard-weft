---
round: 1
ledger:
    - key: reset-counts-lym-own-ignored-files-as-work
      rounds:
        - 1
      status: resolved
      class: design
      severity: BLOCKING
      fix: design-wrong
    - key: reset-misses-bolt-checkout-head-only-commits
      rounds:
        - 1
      status: open
      class: design
      severity: MEDIUM
      fix: mechanical
    - key: reset-unreadable-store-refusal-has-no-way-forward
      rounds:
        - 1
      status: open
      class: design
      severity: MEDIUM
      fix: mechanical
    - key: bug-fix-lacks-root-cause-trailer
      rounds:
        - 1
      status: open
      class: design
      severity: MEDIUM
      fix: mechanical
    - key: sandbox-core-suite-claims-idempotent-reset
      rounds:
        - 1
      status: open
      class: consistency
      severity: MEDIUM
      fix: mechanical
    - key: discard-work-cli-pass-through-untested
      rounds:
        - 1
      status: open
      class: design
      severity: MEDIUM
      fix: mechanical
    - key: ignored-path-cap-and-count-untested
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: mechanical
    - key: hub-without-bolt-refusal-names-refused-reset
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: mechanical
    - key: store-lstat-error-reads-as-no-repository
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: mechanical
---
Round 1 opened every entry above.
The central defect is the rule for which ignored files count as work: it includes files lym itself writes,
so a plain `--reset` of a used hub is refused and `--discard-work` becomes the routine path.
The other entries are gaps in what the reset check reads or how it fails, missing tests, a missing commit trailer
and a stale sandbox suite step.
