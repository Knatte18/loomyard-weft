---
round: 1
ledger:
  - key: d4-way-forward-per-site
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d6-profiles-collision-drops-operator-edit
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d7-retry-only-next-build
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: run-settings-set-one-token-name
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: d5-prime-seed-not-moved
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: mixed-build-risk-overstates-refusal
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d6-criterion-byte-equality
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised seven findings, none carried from an earlier round.
The gating ones are D4's single way forward (it reaches no fix at most refusal sites), D6's profiles.yaml collision that drops an operator's edit, and D7's retry that waits for the next build.
The `decision` finding on the one-token dotted name for `run_settings` and `--set` waits on the parent.
The remaining three are a prime-seed gap in D5 and two wording corrections.
