---
round: 3
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
    status: resolved
  - key: relaunch-finish-has-no-start-path-return-contract
    rounds: [2]
    status: resolved
  - key: cards14-16-shuttleengine-doc-listed-under-extends
    rounds: [2]
    status: resolved
  - key: stop-ended-start-path-relaunch-outcome-not-a-result
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
---

Round 1 opened thirteen findings, all resolved by round 2.
Round 2 opened three BLOCKING findings, and the round 3 review resolves all three.
Round 3 opened one BLOCKING design finding, F17, in the text round 2's F15 fix added to card 14.
Its stop-ended start-path relaunch names an outcome that is never a `Result.Outcome`, and no row drives a stop through `Start`.
Card 16's refused-resume path returns the same way, so F17's fix covers both.
Findings per round fell from thirteen to three to one, and every key but the latest is resolved.
