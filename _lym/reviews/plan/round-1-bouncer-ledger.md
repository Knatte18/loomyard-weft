---
round: 1
ledger:
  - key: relay-restart-cap-unreachable
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: await-first-read-stale-state
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: relay-verb-location-from-hub-dir
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: relay-fork-bound-unstated
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: outbound-read-timeout-after-handler
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: checkbinary-type-mismatch
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: creel-doc-extends-falsified
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: cards-11-13-stale-docs
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: step6-lock-without-probe-grace
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: per-helper-text-tests
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: readwait-subtest-duplicate
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card8-edits-card7-test-file
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: login-empty-token-no-row
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: slackgeometry-credential-dir-no-row
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: cwd-resolution-bound-list-missing-slackengine
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: setup-missing-scope-names-no-scopes
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first review; the ledger maps F1 to F16 in order onto the keys above.
The gating findings are F1 (BLOCKING), and F2, F3, F4 and F8 (MEDIUM, `design`).
F1, F2, F4 and F5 change the relay's restart accounting, first-read await, hidden-verb bound and read loop, so their fixes are new design text.
F3 changes how the relay verb resolves its location, which is new design text too.
F6 to F16 are local corrections whose result the finding text fixes.
