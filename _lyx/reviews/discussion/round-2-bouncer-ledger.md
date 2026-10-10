---
round: 2
ledger:
  - key: d6-dead-fork-recover-batch-refuses-at-cap
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
  - key: d2-glob-bound-assumes-bash-parens-unchecked
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
  - key: d2-quoted-glob-vs-fabric-diff-acceptance
    rounds: [1]
    status: resolved
  - key: d3-json-pin-no-acceptance-row-wrong-home
    rounds: [1]
    status: resolved
  - key: d2-after-dashdash-premise-false-for-ls-remote
    rounds: [1]
    status: resolved
  - key: d5-webstercli-missing-from-scope
    rounds: [1]
    status: resolved
  - key: auto-mode-assumptions-restate-decisions
    rounds: [1]
    status: resolved
  - key: open-risks-non-risk-and-routine-note
    rounds: [1]
    status: resolved
  - key: d4-miscounts-rows-already-present
    rounds: [1]
    status: resolved
  - key: d6-default-branch-names-refusing-route
    rounds: [2]
    status: open
    class: scope
    severity: LOW
  - key: d6-acceptance-rows-skip-fork-at-cap-with-amendment
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised nine findings; round 2 raised four.
Round 2 cleared D3, D5, the quoted-glob rule and the `ls-remote` form, and did not re-raise the three non-gating consistency keys.
Two round-1 keys came back with new gaps: D6's recovery-retry rule (now the unrendered amendment below the cap and its disagreement with Merriam's ladder) and D2's shell bound (now zsh `EXTENDED_GLOB`).
Two LOW findings are new: D6's default branch still names recover-batch for a terminal stuck record at the cap, and the D6 acceptance rows omit a dead fork at the cap with an unrendered amendment.
