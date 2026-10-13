---
round: 1
ledger:
  - key: failed-client-listing-guard-test-unreachable
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: lyx-hidden-doc-claim-goes-stale
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: placed-new-window-double-target-flag
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: overview-anchor-unbounded-grouped-display
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: new-agent-window-selected-pinned-twice
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: hidden-placement-pinned-by-pure-and-engine-row
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: new-window-locked-two-homes
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: replace-strand-never-refused-too-broad
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first round, with no prior ledger.
Eight findings are open, four of them gating design MEDIUMs: an unreachable guard test on card 7, a doubled `-t` flag on placed `new-window` in card 5, an unbounded overview anchor in card 6, and duplicated pure and tmux-tier rows for `newAgentWindowSelected` in card 7.
The rest are a stale `@lyx_hidden` doc claim, two LOW duplicates or disagreements in card 5, and a NIT on card 6's `ReplaceStrand` wording.
