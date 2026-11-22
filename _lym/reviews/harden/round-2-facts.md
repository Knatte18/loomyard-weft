# Round 2 facts

| Round | Findings | BLOCKING | MEDIUM | LOW | NIT | Highest | design | scope | decision | consistency | Gating |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | 5 | 1 | 2 | 2 | 0 | BLOCKING | 3 | 1 | 0 | 1 | 3 |
| 2 | 3 | 2 | 0 | 1 | 0 | BLOCKING | 2 | 0 | 0 | 1 | 1 |

## Recurring keys (ledger rounds each key was open in)

No ledger key recurs across rounds.

## Keys open in an earlier round (ledger rounds each key was open in)

- `faulted-fetch-refspec-read-as-two-repositories`: rounds 1
- `prime-key-names-missing-branch`: rounds 1
- `seed-board-push-lands-after-rollback`: rounds 1
- `seed-way-forward-misnames-hook-decline`: rounds 1
- `seeded-records-branch-has-no-upstream`: rounds 1

## Campaign facts

- Charter: `/home/knatte/Code/loomyard-LYMHUB/harden-fabricengine/_lym/reviews/harden/round-2-focus.md`, the focus file this round ran under; the next focus file restates its standing sections.
- Operator's ask: `/home/knatte/Code/loomyard-LYMHUB/harden-fabricengine/_lym/discussion/decision-record.md`; the focus its `## Goal` states, as its `Added after Discussion` entries amend it, binds the next focus file.
- Models: reviewer `fable`, fixer `fable`.
- Live report: green. The `harden-live` run passed and started 3 test binaries.
- Copies run: passed and started 9 test binaries.
- Strays: none.
- Fix commits: 8a8ff40 fabricengine: fix gate finding — open the unanswered-push rule's second line with a clause the comment lint reads, e9d677c fabric sandbox suite: cover a killed records push, the kept hub and two more records-store faults, ba00dc2 fabricengine: fix F7 — the prime-key refusals name the clone-time records branch through RecordsBranchName and refuse a key that is no records branch, 704c7ae fabricengine: fix F6, F8 — read origin before rolling back an unanswered records push, and read a branch's merge setting as a records-store mark.
- Deferred by the fixer: none.
- Deferred by the red live report: none.
