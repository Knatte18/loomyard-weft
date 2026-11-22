# Round 1 facts

| Round | Findings | BLOCKING | MEDIUM | LOW | NIT | Highest | design | scope | decision | consistency | Gating |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | 5 | 1 | 2 | 2 | 0 | BLOCKING | 3 | 1 | 0 | 1 | 3 |

## Recurring keys (ledger rounds each key was open in)

No ledger key recurs across rounds.

## Keys open in an earlier round (ledger rounds each key was open in)

No ledger key was open in an earlier round.

## Campaign facts

- Charter: `/home/knatte/Code/loomyard-LYMHUB/harden-fabricengine/_lym/reviews/harden/round-1-focus.md`, the focus file this round ran under; the next focus file restates its standing sections.
- Operator's ask: `/home/knatte/Code/loomyard-LYMHUB/harden-fabricengine/_lym/discussion/decision-record.md`; the focus its `## Goal` states, as its `Added after Discussion` entries amend it, binds the next focus file.
- Models: reviewer `opus`, fixer `opus`.
- Live report: red. The `harden-live` run failed and started 3 test binaries.
- Copies run: failed and started 9 test binaries.
- Strays: none.
- Fix commits: c25e17a fabricengine: fix gate finding — shorten the killed-push test file's header under 120 characters, f7af2cb fabricengine: fix gate finding — move the tier2 killed-board-push test to a file of its own, 3d24c44 fabricengine: fix gate findings — break the round's new comments at sentences and under 120 characters, 5a1bf5a fabric sandbox suite: cover the one-repository seed and records-store faults, 8bd71e3 fabricengine: fix F3 — refuse the orphan sweep when the prime key names no branch, a628d0f fabricengine: fix F2 — a records store that lost its namespaced fetch refspec is an error, not two repositories, 5a5f952 fabricengine: fix F1, F4, F5 — withdraw a board push that landed unanswered, name a declined board, give the seeded records branch its upstream.
- Deferred by the fixer: none.
- Deferred by the red live report: FAIL	internal/fabriccli [setup failed]
- Deferred by the red live report: FAIL	internal/fabricengine [setup failed]
- Deferred by the red live report: FAIL	internal/gitrepo [setup failed]
