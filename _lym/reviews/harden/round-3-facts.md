# Round 3 facts

| Round | Findings | BLOCKING | MEDIUM | LOW | NIT | Highest | design | scope | decision | consistency | Gating |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | 7 | 1 | 5 | 1 | 0 | BLOCKING | 5 | 0 | 0 | 2 | 5 |
| 2 | 4 | 3 | 1 | 0 | 0 | BLOCKING | 4 | 0 | 0 | 0 | 4 |
| 3 | 0 | 0 | 0 | 0 | 0 | - | 0 | 0 | 0 | 0 | 0 |

## Recurring keys (ledger rounds each key was open in)

- `archive-tag-plain-refs-tags-one-repo`: rounds 1, 2
- `push-missing-fetch-refspec-no-way-forward`: rounds 1, 2

## Keys open in an earlier round (ledger rounds each key was open in)

- `archive-tag-plain-refs-tags-one-repo`: rounds 1, 2
- `archive-tags-share-one-origin-prefix`: rounds 2
- `guard-push-bypass-any-paths`: rounds 1
- `guard-refusal-omits-sandbox-way-forward`: rounds 1
- `killed-teardown-child-orphans-unnamed`: rounds 1
- `pending-teardown-rerun-skips-owed-archive`: rounds 2
- `pull-missing-fetch-refspec-reports-ok`: rounds 1
- `push-missing-fetch-refspec-no-way-forward`: rounds 1, 2
- `unpushed-commits-no-upstream-says-none`: rounds 1

## Campaign facts

- Charter: `/home/knatte/Code/loomyard-LYMHUB/harden-fabric-single-repo-verbs/_lym/reviews/harden/round-3-focus.md`, the focus file this round ran under; the next focus file restates its standing sections.
- Operator's ask: `/home/knatte/Code/loomyard-LYMHUB/harden-fabric-single-repo-verbs/_lym/discussion/decision-record.md`; the focus its `## Goal` states, as its `Added after Discussion` entries amend it, binds the next focus file.
- Models: reviewer `sonnet`, fixer `sonnet`.
- Live report: red. The `harden-live` run passed and started 0 test binaries.
- Copies run: passed and started 9 test binaries.
- Strays: none.
- Fix commits: 7e498df gatecli: fix gate findings — split the two --no-carry comment lines that reached 120 characters, 592c88c gatecli: fix GATE-LIVE — add --no-carry so the hardener's live call runs its tests over an unchanged tree.
- Deferred by the fixer: none.
- Deferred by the red live report: the `harden-live` run failed or started no test binary and its output names no failing test; read /home/knatte/Code/loomyard-LYMHUB/harden-fabric-single-repo-verbs/_lym/reviews/harden/harden-live-3.json
