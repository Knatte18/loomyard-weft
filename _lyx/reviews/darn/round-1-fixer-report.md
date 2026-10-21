# Fixer-report, darn round 1

## Changes

- F1, fixed as proposed (`be549fea`): `tools/deploy/main_test.go` gains `TestLyxMainPackage_MatchesModulePath`, which reads `go.mod`'s module path and asserts `lyxMainPackage` equals it plus `/cmd/lyx`; it spawns nothing.
- F2, fixed as proposed (`cb8bfbc8`): `otherLyxBinaries` in `tools/deploy/main.go` keys `seenFiles` on the `filepath.EvalSymlinks` result at the seed, lookup and insert sites, with no `normDir` folding.
  `normDir` stays for directory keys and its comment says it compares PATH entries.
- Self-check LOW, fixed as proposed: the `main_test.go` file header now lists the `lyxMainPackage` guard.

`lyx gate test ./tools/deploy` is green after F1 and F2.

## Self-check

- fix-consistency, LOW: the test file header omitted the new go.mod guard; fixed as proposed.
- fix-code-correctness: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
