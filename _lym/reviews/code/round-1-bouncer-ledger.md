---
round: 1
ledger:
  - key: prebuild-check-ignores-whole-command-tier
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: prebuiltlym-field-doc-unconditional
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: prebuiltlymenv-doc-unconditional
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: running-tests-doc-harness-comparison
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: test-speed-doc-build-condition-narrow
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: copies-path-skip-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: directory-target-dot-branches-untested
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
---

Round 1 opened seven findings around the change that makes the `lym` prebuild conditional.

- The one gating item is the build decision for a tier whose command holds no target placeholder.
  Such a command runs whole, but the check reads the explicit targets.
- Three items are prose that still states the old unconditional build or a comparison resting on it, and one is a rule doc stating the new condition too narrowly.
- Two items are test gaps: the skip on the `--copies` path, and the `.` and `..` target branches.
