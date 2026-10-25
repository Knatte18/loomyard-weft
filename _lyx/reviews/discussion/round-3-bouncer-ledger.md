---
round: 3
ledger:
  - key: scope-full-accept-resume-loop
    rounds: [1, 2]
    status: resolved
  - key: d4-runproc-false-premise
    rounds: [1, 2]
    status: resolved
  - key: scope-partial-accept-unbounded
    rounds: [1, 2]
    status: resolved
  - key: goto-drift-baseline-replans-loom
    rounds: [1]
    status: resolved
  - key: adopt-loses-baseline
    rounds: [1]
    status: resolved
  - key: drift-await-clock-mismatch
    rounds: [1]
    status: resolved
  - key: compare-side-verbs-no-row-guard
    rounds: [1]
    status: resolved
  - key: goto-webster-regression-second-halt
    rounds: [1]
    status: resolved
  - key: drift-verify-placement-parent-align
    rounds: [1, 2]
    status: resolved
  - key: drift-await-done-vs-skipped
    rounds: [1, 2]
    status: resolved
  - key: capture-holds-back-plan-gate
    rounds: [1]
    status: resolved
  - key: no-code-change-uncompared-too-broad
    rounds: [1]
    status: resolved
  - key: e2e-proof-location-sandbox-hub
    rounds: [1]
    status: resolved
  - key: ac1-darn-drift-await
    rounds: [1]
    status: resolved
  - key: lyx-exclude-existing-behaviour
    rounds: [1]
    status: resolved
  - key: forward-goto-refused-no-waive
    rounds: [2]
    status: resolved
  - key: step-idle-watchdog-kills-drift-wait
    rounds: [2]
    status: resolved
  - key: flag-off-mid-capture-unbounded
    rounds: [2]
    status: resolved
  - key: verify-budget-bound-false-under-arbiter
    rounds: [2]
    status: resolved
  - key: darn-failed-capture-no-way-forward
    rounds: [2]
    status: resolved
  - key: baseline-reentry-unstated-cases
    rounds: [2]
    status: resolved
  - key: plan-reviewers-build-beside-capture
    rounds: [2]
    status: resolved
  - key: finalize-reroute-misstated
    rounds: [2]
    status: resolved
  - key: capture-log-commit-rule-gaps
    rounds: [2]
    status: resolved
  - key: darn-env-settingknown-false-note
    rounds: [2]
    status: resolved
  - key: support-log-stale-against-round-1
    rounds: [2]
    status: resolved
  - key: scope-full-unruled-acceptance-survives-interrupt
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: flag-off-gate-only-at-row-entry
    rounds: [3]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: waive-precondition-contradicts-halt
    rounds: [3]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: compare-marker-liveness-and-slot-wait
    rounds: [3]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: adopt-cannot-end-old-machine-capture
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: scope-in-omits-pairteardown-runproc
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: spawnloop-breakaway-left-to-plan
    rounds: [3]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
---

Round 1 raised fifteen findings and round 2 sixteen, five of them BLOCKING.
Round 3 raised seven: one BLOCKING, two MEDIUM and four LOW.
Every earlier key lapses, since round 3 does not raise any of them again; all are resolved.

The remaining BLOCKING finding is a new scope-full hole: an acceptance an agent session makes before an error, transient or interrupted stop survives into the resumed row, which compares first and returns Done without the parent ruling.
It is a different mechanism from the arbiter accept of the earlier scope-full key, so it opens a new key.
The two MEDIUM findings are record contradictions: the flag-off claim holds only for a row not yet entered, and the waive precondition contradicts the halt that names waive.
The LOW findings are the compare marker's liveness, the adopt claim about another machine, the Scope In omission and the unsettled `spawnLoop` breakaway question.
