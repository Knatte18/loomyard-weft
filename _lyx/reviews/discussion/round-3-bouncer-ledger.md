---
round: 3
ledger:
  - key: d6-dead-fork-recover-batch-refuses-at-cap
    rounds: [1, 2, 3]
    status: open
    class: design
    severity: MEDIUM
  - key: d2-glob-bound-assumes-bash-parens-unchecked
    rounds: [1, 2, 3]
    status: open
    class: design
    severity: BLOCKING
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
    status: resolved
  - key: d6-acceptance-rows-skip-fork-at-cap-with-amendment
    rounds: [2]
    status: resolved
  - key: d6-exhausted-remedy-uncovered-by-merriam-rung
    rounds: [3]
    status: open
    class: decision
    severity: MEDIUM
---

Round 1 raised nine findings, round 2 four, round 3 three.
Seven round-1 keys and the two round-2 LOW keys are resolved; the round-2 LOW row key is subsumed by round 3's F2, which finds the state it pins unreachable.
Two design keys have now been open three rounds.
D2's glob enumeration has gained a hole each round (parentheses, then zsh `EXTENDED_GLOB`, now `#`), so the open question is the method of bounding globs, not the next character.
D6's dead-fork case moved from "recover-batch refuses at the cap" to "no code path produces a dead fork record".
A new decision-class key concerns begin-batch's exhausted text at the cap and Merriam's missing rung for it.
