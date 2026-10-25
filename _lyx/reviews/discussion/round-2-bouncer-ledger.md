---
round: 2
ledger:
  - key: scope-full-accept-resume-loop
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d4-runproc-false-premise
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: scope-partial-accept-unbounded
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
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
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: drift-await-done-vs-skipped
    rounds: [1, 2]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
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
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: step-idle-watchdog-kills-drift-wait
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: flag-off-mid-capture-unbounded
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: verify-budget-bound-false-under-arbiter
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: darn-failed-capture-no-way-forward
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: baseline-reentry-unstated-cases
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: plan-reviewers-build-beside-capture
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: finalize-reroute-misstated
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: capture-log-commit-rule-gaps
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: darn-env-settingknown-false-note
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: support-log-stale-against-round-1
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 2 raised sixteen findings, five of them BLOCKING.
Round 1's keys mostly lapse: the review does not raise them again, so they are resolved.
Five round 2 findings reopen round 1 keys.
F2 maps to `d4-runproc-false-premise` (D4's process model, now the Windows lifetime of the capture), F4 to `scope-full-accept-resume-loop` (the arbiter can accept under scope full), F5 to `drift-await-done-vs-skipped` (the parent's gating ruling not carried into the body), F7 to `scope-partial-accept-unbounded`, and F14 to `drift-verify-placement-parent-align` (the parent's ruling already settles the Open risks line).
The other round 2 findings are new keys.
The BLOCKING design set is the refused forward goto, the Windows capture lifetime, the step idle watchdog and the scope-full arbiter accept.
