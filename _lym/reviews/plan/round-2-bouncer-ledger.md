---
round: 2
ledger:
  - key: card3-misses-battencli-primeshuttlerunner
    rounds: [1]
    status: resolved
  - key: load-turn-usage-limit-bypasses-switch
    rounds: [1]
    status: resolved
  - key: card2-breaks-configreg-machine-key-test
    rounds: [1]
    status: resolved
  - key: card16-refused-resume-site-unreachable
    rounds: [1]
    status: resolved
  - key: card5-adopt-test-builder-nil-pickaccount
    rounds: [1]
    status: resolved
  - key: docs-listed-under-extends-but-rewritten
    rounds: [1]
    status: resolved
  - key: card14-deferred-prompt-rule-departs-from-d7
    rounds: [1]
    status: resolved
  - key: card11-hold-has-no-stated-bound
    rounds: [1]
    status: resolved
  - key: card4-resolve-account-test-repeats-card6
    rounds: [1]
    status: resolved
  - key: shuttle-state-omits-relaunching
    rounds: [1]
    status: resolved
  - key: card12-held-strand-removable-sites-unnamed
    rounds: [1]
    status: resolved
  - key: card5-reentered-worktree-create-never-picks
    rounds: [1]
    status: resolved
  - key: card7-enter-session-resolves-wrong-location
    rounds: [1]
    status: resolved
  - key: card5-battencli-pickaccount-nil-in-create-tests
    rounds: [2]
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
  - key: relaunch-finish-has-no-start-path-return-contract
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: cards14-16-shuttleengine-doc-listed-under-extends
    rounds: [2]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
---

Round 1 opened thirteen findings, and the round 2 review marks every one resolved.
Round 2 opened three new BLOCKING findings, each on a new key.
Two are fix-regressions of the same class as round-1 keys: the nil seam in battencli's create tests repeats the adopt-builder defect, and the doc listed under Extends repeats the earlier doc defect for cards 14 and 16.
The third is the missing return contract when a relaunch finishes the run on the start path, whose fix states new behaviour.
