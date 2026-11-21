---
round: 2
ledger:
  - key: vstest-result-words-reach-every-args-tier
    rounds: [1]
    status: resolved
  - key: measurecall-tier-fallback-conflicts-with-pattern
    rounds: [1]
    status: resolved
  - key: shell-spawning-tests-in-tier0
    rounds: [1]
    status: resolved
  - key: measurecall-renders-refused-call
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: env-line-role-clause-failed-instruction
    rounds: [1]
    status: resolved
  - key: env-line-prints-session-value
    rounds: [1]
    status: resolved
  - key: trx-test-leaves-baseline-and-fallbacks-unchecked
    rounds: [1, 2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: session-env-capture-unpinned
    rounds: [1]
    status: resolved
  - key: darn-wiring-gate-config-unpinned
    rounds: [1]
    status: resolved
  - key: caller-result-options-take-trx-away
    rounds: [1]
    status: resolved
  - key: warning-run-messages-become-causes
    rounds: [1]
    status: resolved
  - key: pending-console-line-read-after-replacement
    rounds: [1]
    status: resolved
  - key: no-args-tier-gets-no-crash-cause-undocumented
    rounds: [1]
    status: resolved
  - key: env-line-repeats-under-copies
    rounds: [1]
    status: resolved
  - key: darn-measure-call-halts-ungated-runs
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: duplicate-tier0-trx-tests
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
---

Round 1 raised F1 to F14; round 2 resolved all but two of them.
F4 (`measurecall-renders-refused-call`) remains as F17: a covered tier still adds a target word the gate refuses.
F7 (`trx-test-leaves-baseline-and-fallbacks-unchecked`) remains as F18: the test's expected result words call the production helper.

Round 2 raised two BLOCKING findings.
F15 (`darn-measure-call-halts-ungated-runs`): `wireDarn` and `DarnTable` require the measuring call for every darn writer, so a hub whose darn-write tiers hold no `{args}` halts ungated runs at Preflight.
F16 (`duplicate-tier0-trx-tests`): two new tier0 tests duplicate `TestRunTierReadsTheVSTestResultsFile`.
F15 changes a contract between wiring and engine, so it is `design-wrong`; F16 is a deletion the finding text fixes.
