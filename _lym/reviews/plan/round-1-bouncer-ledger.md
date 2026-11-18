---
round: 1
ledger:
  - key: card3-misses-battencli-primeshuttlerunner
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
  - key: load-turn-usage-limit-bypasses-switch
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: card2-breaks-configreg-machine-key-test
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
  - key: card16-refused-resume-site-unreachable
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: card5-adopt-test-builder-nil-pickaccount
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: docs-listed-under-extends-but-rewritten
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: card14-deferred-prompt-rule-departs-from-d7
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: card11-hold-has-no-stated-bound
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: card4-resolve-account-test-repeats-card6
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: shuttle-state-omits-relaunching
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: card12-held-strand-removable-sites-unnamed
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card5-reentered-worktree-create-never-picks
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: card7-enter-session-resolves-wrong-location
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 opened thirteen findings, none carried from an earlier round.
The gating ones are the missed launch site in card 3, the skill-load-turn usage limit that never reaches the switch, the config registry test card 2 breaks, the unreachable start-path refused resume in card 16, and the docs rewritten under Extends.
Card 14's deferred-prompt rule departs from D7 and needs a parent decision.
The rest are local corrections.
