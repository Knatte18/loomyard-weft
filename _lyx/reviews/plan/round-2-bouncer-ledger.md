---
round: 2
ledger:
  - key: generic-rows-caller-coverage-conflict
    rounds: [1]
    status: resolved
  - key: dispatch-skip-bound-unstated-cards-10-11
    rounds: [1]
    status: resolved
  - key: stale-containment-tier-comments-unowned
    rounds: [1]
    status: resolved
  - key: checkglyphmalformed-comment-unenumerated
    rounds: [2]
    status: open
    class: scope
    severity: LOW
---

Round 1 raised three findings; the round-2 review's focus checks found each one addressed.
The `shapes#Generic` rows are consistent across cards 1, 2, 10 and 11.
Cards 8, 10 and 11 state their dispatch bounds.
The `handle_test.go` comments are assigned to card 6.

Round 2 raised one new finding, F1, mapped to `checkglyphmalformed-comment-unenumerated`.
Card 5 omits the `checkGlyphMalformed` doc comment in `internal/planparser/validate.go` that still names `containment-unit-overlap`.
It is LOW and non-gating.
