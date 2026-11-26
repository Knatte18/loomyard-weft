---
round: 1
ledger:
  - key: config-keys-named-bare-or-without-module
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: reconcile-help-and-docs-describe-old-behaviour
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: moved-keys-mail-after-failed-walk-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: reconcile-before-load-sites-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: movekeys-order-breaks-yaml-alias
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: move-tables-checked-only-against-themselves
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: set-area-named-old-key-branch-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: movesettings-drop-outcomes-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: move-error-file-attributions-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: testlookup-needs-keep-mark
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: testmodules-locallayerrefusals-needs-keep-mark
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: testkeyname-redundant
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
  - key: dotted-profile-name-block-silently-not-moved
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
---

Round 1 is the first round.
The review reads the move mechanism as correct in the paths it traced, by reading only.
The blocking item is the incomplete sweep for full config key names.
The rest are stale help and docs, missing test coverage, test-economy marks and two edge cases in moving keys.
