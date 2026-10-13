---
round: 2
ledger:
  - key: failed-client-listing-guard-test-unreachable
    rounds: [1]
    status: resolved
  - key: lyx-hidden-doc-claim-goes-stale
    rounds: [1]
    status: resolved
  - key: placed-new-window-double-target-flag
    rounds: [1]
    status: resolved
  - key: overview-anchor-unbounded-grouped-display
    rounds: [1]
    status: resolved
  - key: new-agent-window-selected-pinned-twice
    rounds: [1]
    status: resolved
  - key: hidden-placement-pinned-by-pure-and-engine-row
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: new-window-locked-two-homes
    rounds: [1]
    status: resolved
  - key: replace-strand-never-refused-too-broad
    rounds: [1]
    status: resolved
  - key: open-window-moves-literal-stale-allowlist
    rounds: [2]
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
  - key: doc-go-sentences-contradict-new-code
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: open-window-nil-state-and-argv-unstated
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: overview-window-size-only-refusal-untested
    rounds: [2]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised eight findings, four of them gating design MEDIUMs; round 2 no longer raises seven of those keys.
The eighth, duplicated pure and tmux-tier placement rows, is back as F3 in a wider form covering cards 5 and 6.

Round 2 adds four findings.
F1 is BLOCKING: card 5 moves `OpenWindow`'s `new-window` literal into `spawn.go`, so the `window.go` entry in `windowCreationAllowlist` goes stale and no card targets the allowlist.
F2 is a MEDIUM listing four `doc.go` sentences that cards 5, 7 and 8 falsify without naming them.
F4 and F5 are LOW: the unstated nil-state and exact `wantArgv` change in card 5, and the untested `WindowSize`-only overview refusal in card 6.
