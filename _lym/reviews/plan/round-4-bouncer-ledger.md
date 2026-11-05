---
round: 4
ledger:
  - key: judge-snapshot-counts-loom-status-commits
    rounds: [1, 2]
    status: resolved
  - key: duplicate-delete-verifyfixture-helpers
    rounds: [1]
    status: resolved
  - key: cards-4-16-26-miss-test-members
    rounds: [1, 2]
    status: resolved
  - key: drift-amend-refusal-no-way-forward
    rounds: [1]
    status: resolved
  - key: check-before-expect-names-defer
    rounds: [1]
    status: resolved
  - key: checkbaseline-unguarded-hides-missing-dir
    rounds: [1]
    status: resolved
  - key: card-18-names-removed-verbs-in-headers
    rounds: [1]
    status: resolved
  - key: capture-lock-wait-no-cadence-no-pattern-listing
    rounds: [1]
    status: resolved
  - key: stale-docs-drift-rows-and-verbs
    rounds: [1]
    status: resolved
  - key: defer-held-lock-wrong-way-forward
    rounds: [1]
    status: resolved
  - key: countedcapture-left-without-caller
    rounds: [1]
    status: resolved
  - key: capture-current-lock-wait-no-injected-sleep
    rounds: [2]
    status: resolved
  - key: drift-expect-row-name-repeated-literal
    rounds: [2]
    status: resolved
  - key: judgement-yaml-never-committed-or-removed
    rounds: [2]
    status: resolved
  - key: overview-darn-sentence-lacks-drift-expect
    rounds: [2]
    status: resolved
  - key: drift-expect-never-counted-stuck-untested
    rounds: [3]
    status: resolved
  - key: drift-verify-interim-command-stuck-untested
    rounds: [3]
    status: resolved
  - key: card-16-needs-row-constants-created-by-card-19
    rounds: [3]
    status: resolved
  - key: judge-snapshot-exempt-set-unbounded
    rounds: [3]
    status: resolved
  - key: parser-refusals-without-test-rows
    rounds: [3]
    status: resolved
  - key: card-22-deletes-live-compare-lock-retry
    rounds: [4]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
---

Rounds 1 to 3 raised sixteen findings across the earlier keys; each was cleared by the next round's review.
Round 4's Rejected section settles the five keys open after round 3: the `Drift-Expect` and `Drift-Verify` Stuck refusals have rows that reach them, card 17 replaced the interim `command` Stuck, `darnWriteRow` and `websterRow` are declared once by card 16, the parser refusals have rows, and card 15's Bound names what bounds the exempt paths.

Round 4 raises one new key, BLOCKING and `design`: card 22 deletes `compareLockRetry`, which the check verb's compare closure in `checkDeps` (`internal/driftcli/wiring.go`) still reads.
The fix drops that `Delete` bullet and nothing else, so it is mechanical.

A baseline finding of the plan gate, card 25 deleting `internal/driftengine#testFiles` while `Check` still references it, is not a finding of any round; it constrains card 25's fix.
