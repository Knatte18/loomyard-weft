---
round: 1
ledger:
  - key: webster-callsites-edited-three-times
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: profiles-not-declared-open-map
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: tokencount-attribution-ignores-real-transcript-shape
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: card6-repeats-card2-coefficient-error-rows
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: card10-misdescribes-worktree-config-resolution
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 opened five findings, none carried over, since this is the first round.

- `webster-callsites-edited-three-times` (F1): cards 4, 7 and 8 each rewrite the webster verbs' batch-forming line; card 7 should merge into card 8.
- `profiles-not-declared-open-map` (F2): no card marks `profiles` as an open map in `configreg`, so reconcile prunes operator profiles and the load re-injects the template's.
- `tokencount-attribution-ignores-real-transcript-shape` (F3): card 1's pointer rule ignores the `cat -n` prefix and does not separate pointer lines from gate lines.
- `card6-repeats-card2-coefficient-error-rows` (F4): card 6's `TestActive` repeats card 2's coefficient-error rows.
- `card10-misdescribes-worktree-config-resolution` (F5): card 10 gives the wrong reason the worktree `batcher.yaml` resolves.
