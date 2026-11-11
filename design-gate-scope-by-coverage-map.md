# A verify gate scopes to nearly the whole suite; scope it by the coverage map

`gate-scope-by-coverage-map` · Task · bug, gate, testkit, darn · running · Worktree-Create

## Seen (orch, 2026-11-09)

The Darn-Write verify gate scopes a run to nearly the whole suite, tier 0 to 2, whenever the diff touches a widely imported package such as `internal/loomengine` or `internal/configreg`.
`testmap-verb` and `arras-recipe-perch` each spent 30 to 40 minutes in it on OSL-1046.
Scope today is by importing packages, so one small change to a leaf of the import graph reruns every package above it.

## Idea

Scope a verify gate by the coverage map project `redundancy-check` builds: run the tests whose recorded blocks cover the changed code, plus the tests of the changed packages, and everything else only in the landing gate.
Read first how the computed scope forms today and what the map holds per test, and decide what a map miss, such as a changed file with no recorded test, does: it must widen the scope, never narrow it.

## Built so far

The map record has landed (`a8d2ae63`): `internal/testcover` writes one `map.json` per package under `<hub>/.lym/testcover/<package>/`, holding the per-file content hashes, the block key table, and per judged test its tier, duration and covered block indexes.
Block keys survive line shifts, and an edit gives a new key no test covers, which reads as a map miss.
This work still waits on the map's verb, `testmap-verb`, which rebuilds stale maps.
