# Round 3 facts

| Round | Findings | BLOCKING | MEDIUM | LOW | NIT | Highest | design | scope | decision | consistency | Gating |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | 5 | 1 | 2 | 2 | 0 | BLOCKING | 3 | 1 | 0 | 1 | 3 |
| 2 | 3 | 2 | 0 | 1 | 0 | BLOCKING | 2 | 0 | 0 | 1 | 1 |
| 3 | 0 | 0 | 0 | 0 | 0 | - | 0 | 0 | 0 | 0 | 0 |

## Recurring keys (ledger rounds each key was open in)

- `faulted-fetch-refspec-read-as-two-repositories`: rounds 1, 2
- `seed-board-push-lands-after-rollback`: rounds 1, 2

## Keys open in an earlier round (ledger rounds each key was open in)

- `faulted-fetch-refspec-read-as-two-repositories`: rounds 1, 2
- `prime-key-names-missing-branch`: rounds 1
- `prime-key-refusal-spells-suffix-literal`: rounds 2
- `seed-board-push-lands-after-rollback`: rounds 1, 2
- `seed-way-forward-misnames-hook-decline`: rounds 1
- `seeded-records-branch-has-no-upstream`: rounds 1

## Campaign facts

- Charter: `/home/knatte/Code/loomyard-LYMHUB/harden-fabricengine/_lym/reviews/harden/round-3-focus.md`, the focus file this round ran under; the next focus file restates its standing sections.
- Operator's ask: `/home/knatte/Code/loomyard-LYMHUB/harden-fabricengine/_lym/discussion/decision-record.md`; the focus its `## Goal` states, as its `Added after Discussion` entries amend it, binds the next focus file.
- Models: reviewer `sonnet`, fixer `sonnet`.
- Live report: green. The `harden-live` run passed and started 2 test binaries.
- Copies run: refused: gate test: --copies runs one tier below tier3, and --gate "harden-live" names tiers [0 1]; way forward: set the key to one tier below tier3 with lym config gate --set from the prime; a session that is not the prime takes none of these steps; it reports status: FAILED and the orch runs them
- Strays: none.
- Fix commits: none.
- Deferred by the fixer: none.
- Deferred by the red live report: none.
