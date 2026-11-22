# Round 2 facts

| Round | Findings | BLOCKING | MEDIUM | LOW | NIT | Highest | design | scope | decision | consistency | Gating |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | 7 | 1 | 5 | 1 | 0 | BLOCKING | 5 | 0 | 0 | 2 | 5 |
| 2 | 4 | 3 | 1 | 0 | 0 | BLOCKING | 4 | 0 | 0 | 0 | 4 |

## Recurring keys (ledger rounds each key was open in)

No ledger key recurs across rounds.

## Keys open in an earlier round (ledger rounds each key was open in)

- `archive-tag-plain-refs-tags-one-repo`: rounds 1
- `guard-push-bypass-any-paths`: rounds 1
- `guard-refusal-omits-sandbox-way-forward`: rounds 1
- `killed-teardown-child-orphans-unnamed`: rounds 1
- `pull-missing-fetch-refspec-reports-ok`: rounds 1
- `push-missing-fetch-refspec-no-way-forward`: rounds 1
- `unpushed-commits-no-upstream-says-none`: rounds 1

## Campaign facts

- Charter: `/home/knatte/Code/loomyard-LYMHUB/harden-fabric-single-repo-verbs/_lym/reviews/harden/round-2-focus.md`, the focus file this round ran under; the next focus file restates its standing sections.
- Operator's ask: `/home/knatte/Code/loomyard-LYMHUB/harden-fabric-single-repo-verbs/_lym/discussion/decision-record.md`; the focus its `## Goal` states, as its `Added after Discussion` entries amend it, binds the next focus file.
- Models: reviewer `fable`, fixer `fable`.
- Live report: green. The `harden-live` run passed and started 2 test binaries.
- Copies run: passed and started 9 test binaries.
- Strays: none.
- Fix commits: 3366b8b fabricengine: fix gate findings — end this round's two wrapped comment lines at sentence boundaries, 1207f0b tools/sandbox: fix G1, G2, G3, G4 — extend the fabric suite's one-repository scenario with this round's behaviours, bacdd32 fabricengine: fix O1, O2 — name the seed for an origin a stopped seed left, and name a clone directory that is not there, 3a5e28e fabricengine: fix G2 — read the records layout before the board's pull, so reconcile names the refspecs to restore, a1d8b87 fabricengine: fix G3, O3 — push the archive tags a dead teardown left locally before any origin delete, and keep the pending record of a failed step, bb29278 fabricengine: fix G1, G4 — give a one-repository hub's code store its own archive-tag prefix, whatever URL it spells origin by.
- Deferred by the fixer: none.
- Deferred by the red live report: none.
