# Round 1 fixer-report

## Changes

- F1 (`cdb3568d`): `linkCallFailures` in `cmd/lym/fslinkonly_test.go` reports a dot import of `os` as a failure; fixture rows for a dot import (1 failure) and a blank import (0).
- F2 (`c1c3970e`): `headlessSpawnFailures` in `cmd/lym/interactivespawn_test.go` consults the allowlist only for a file with findings.
  Fixed differently: the review proposed an optional allowlist field on the fixture loop; every row there is positional, so a separate `TestInteractiveSpawn_Allowlist` holds the admission row and the stale-entry row.
- Self-check follow-up: `pattern/PATTERN-fs-links.md` now says a dot import of `os` is reported at the import line.

## Self-check

- fix-consistency, LOW: stale `PATTERN-fs-links.md` bullet — fixed as proposed.
- fix-consistency, note: separate allowlist test — marked above as fixed differently.
- fix-code-correctness: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
