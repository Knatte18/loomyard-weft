---
round: 1
ledger:
  - key: resume-pushes-ungated-head
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: wake-pushes-before-foreign-head-check
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: board-run-settings-keep-retired-pr-modes
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: retired-pr-reviewer-role-breaks-agents-yaml
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: on-to-review-notice-misstates-meaning
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: foreign-head-order-before-endings-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: merge-danger-blast-radius-understated
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: landing-for-decision-doc-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: retired-key-comment-misstates-machine-layer
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: local-layer-pr-reviewer-load-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: retired-loom-pr-local-and-run-settings-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: pr-continue-strand-env-read-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: in-flight-pr-reviewer-session-never-stopped
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: shellguard-publish-comment-names-removed-session
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: overview-lists-parent-review-ask
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: pattern-landing-summaries-omit-operator-ending
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: duplicate-beam-reinterpretation-test
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: told-worktrees-test-missing-keep-mark
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: template-defaults-test-missing-keep-mark
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: rubric-fixture-names-pr-findings
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first round, so every key is new and open.
The gating work is in three areas:

- PR-Review's push order on resume and on a wake, with its missing ordering tests.
- The upgrade path for the retired values and role: board `run_settings`, hub `agents.yaml` and the rewrite notice.
- The blast radius the change description names.

`on-to-review-notice-misstates-meaning` carries the class and severity set by the relabel in the verdict.
