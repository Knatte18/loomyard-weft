---
round: 1
ledger:
  - key: shared-build-env-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: gate-hub-config-dirs-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: drift-wiring-sites-unpinned
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: reed-launch-export-order-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: gate-broken-env-refusal-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: hubgeom-paneenv-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: path-variable-replaces-prelude
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: lym-prefix-case-sensitive
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: env-entries-follow-build-server-switches
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 is the first review.
Four gating test gaps (F1 to F4) cover the shared build, the hub branch of the gate's config dirs, the Drift wiring sites and the reed launch ordering.
F5 and F6 are further test gaps.
F7 and F8 are validation holes in the variable-name check, and F9 is the order of the module's entries against the build-server switches.
