## Goal

testmap records each test's coverage across every in-module package its binary links

Takes: testmap and the redundancy report as they stand once testmap-upkeep's design has landed
Delivers: maps whose blocks span every linked in-module package, with per-linked-package file hashes (go:embed targets in subdirectories included), each test's spawn-scan verdict and a format marker holding Commit and BuiltAt; dirty checks over every linked package's files, read fresh before the build and after the last test; the redundancy report and mapDrift unchanged in verdict
Exports: testcover.Map's linked-package field, spawn verdict per test and format marker; BlockKey carrying an import path for another package's block

## Scope

The part changes these packages:

- internal/testcover (about 300 changed lines)
- internal/gatecli (about 500 changed lines)

## Decisions

### D2 — Maps record cross-package coverage

`lym gate testmap` builds each package's coverage binary with coverpkg naming every in-module package the test binary links,
the package itself included, and records each test's covered blocks across all of them.

- A block key from another package carries that package's import path; own-package keys keep their present shape.
- Beside the own-package `Files`, the map records each linked package other than its own in a field of its own.
  The linked packages are every package of that coverpkg list, whether or not a test covered a block of one.
  Per linked package it records the content hash of every tracked file impactset's path-to-package rule maps to it:
  its own Go files, and every other file under its directory outside a nested package directory, `go:embed` targets included.
- Each test records whether `testcover.ScanSpawns` finds it out-of-process or unresolved.
- The map carries a format marker for this shape, holding the `Commit` and `BuiltAt` of the build that wrote it;
  a failed-build map carries it too.
- Tests that cover no block of their own package but some block elsewhere now enter the map.

testmap skips a package, as it does today for a dirty file of its own directory,
when a modified or untracked file belongs to any linked package by the rule above,
and its check for files changed while the tests ran covers every linked package's files.
Both dirty checks of a package are fresh reads of the worktree status, each over every linked package's files:
the first taken immediately before the package's build, the second after its last test;
neither reuses the list testmap read before its walk.
When the second finds a dirty file, testmap writes no map for that package, as it does when files changed at HEAD.
A map's recorded hashes are the content at its `Commit`; the Bound below names the one window in which the measured blocks can differ.
The redundancy report keeps its fresh builds on an own-package coverpkg and judges against kept tests' own-package blocks only, so its verdicts do not change;
`mapDrift` keeps comparing own-package files only.

Rationale: a map that holds only its own package's blocks cannot tell which test of an importing package reaches a changed package.
Bound: an edit to a linked file made after the first dirty read and reverted before the second goes undetected;
the map then holds blocks measured on the edited code under the committed hashes,
and only the landing gate catches a test that this wrongly leaves out.

### Added after Discussion (parent, 2026-11-11): D2 leaves testmap's staleness rule as it is

Decision: F4/Q10: no. The writer removes D2's staleness rule and states in D5 that a dependency-drift miss lasts until testmap rebuilds the map by its present rules (own files or the age limit). The dependency-drift rule D2 proposed goes into note testmap-upkeep as an input, not into this record.

Rationale: Note testmap-upkeep owns the whole staleness and refresh policy, dependency drift included, and has a research step of its own still to run; this record decides how the gate scopes by a map, not when a map goes stale.

### Added after Discussion (parent, 2026-11-11): This motif lands now; the build parts wait on the upkeep design

Decision: Q11: the motif lands its record now without depending on testmap-upkeep. The record states, under Open risks, that coverage narrows only while a map is fresh, that the fresh-map share is set by testmap-upkeep's cadence, and that the tasks raddle files from this record must be ordered after the task that builds testmap-upkeep's design.

Rationale: A motif lands no code, so landing the design costs nothing that the upkeep cadence could make wrong; the ordering belongs to the build tasks, which raddle files later.

### Added after Discussion (parent, 2026-11-11): The verify gate's tier stop and budget from 1b6e3fca stand

Decision: The record builds on the verify gate as commit 1b6e3fca left it: the stop at the first failed tier (--stop-on-fail), the rerun of only the failed tests, and the verify_budget_min time budget. Coverage scoping composes with them and replaces none of them.

Rationale: 1b6e3fca landed after this run's Discussion started; the board tidy flagged that the record may predate it.

### Added after Discussion (arbiter, 2026-11-11): Round-3 hard-cap findings are answered by the fixes in place

Decision: The Discussion review ends at round 3 with no further edit. The round-3 fixes in place answer each open finding: F1 by the parent's two Added after Discussion entries now applied to D2, D5's Bound, Auto-mode assumptions, Open risks and the support log; F2 by D3's host-admission rule for the whole union (holdsTierTest, go/build Context.MatchFile with the tier's tag), D6's -run in the run's own args, and the other-GOOS D3 criterion; F3 by D2's two fresh dirty reads, its reverted-edit Bound and its criterion; F4 by D5's go:embed carve-out from the inert_paths exemption and its criterion; F5 by D5's predeclared-name miss, the cut dot-import Bound clause and the max criterion.

Rationale: Hard-cap rule: accept when the fix in place already answers each open design-wrong finding. The round-3 fixer report and the current decision record show every proposed fix applied; internal/gatecli/tierrun.go confirms holdsTierTest and emptyTierFailure behave as the F2 fix relies on.

## Constraints

- Go 1.26 (`go.mod`, toolchain go1.26.8): `go test -c -cover -coverpkg=<list>` instruments every listed package, and the set-mode profile lists blocks of each by file.
- PATTERN-config-layers: `coverage` is a new value, not a changed meaning of `diff`; `gateslot` stays a strict loader, so a machine on an older binary refuses a hub file holding `coverage` until it updates.
- PATTERN-tolerant-records: a map read never refuses an unknown key; D2's fields and marker are added fields.
  `testcover.WriteMap` keeps undeclared keys on a rewrite, so the marker's own `Commit` and `BuiltAt` tell apart a map an older binary rewrote.
- PATTERN-fabric-boundary: file content at a map's `Commit` for D4 is read through `fabricengine.Fabric.CodeFileAt`,
  the reader testmap's file hashes and the redundancy report's map drift already use, never a new git call.
- PATTERN-gate-slot: map builds stay in `lym gate testmap`'s held slots; the scope reads maps before any tier takes its slots, and starts no build.
- PATTERN-test-isolation: the fixture module D2's criteria build on uses hermetic git.
- Measured cost split: the `darn-write` tier run records under the hub's gate runs directory from 2026-11-04 on, as of 2026-11-11,
  build, guard and carried records left out, sum to tier0 191 min, tier1 516 min and tier2 268 min over about 245 runs each.
  Tiers 0 and 1, which this task narrows, carry about 73 percent; tier 2 keeps the import scope.
- Measured non-body change rate: main's first-parent history from 2026-11-04 to 2026-11-11 holds 118 commits
  that touch non-test Go under `internal/` or `cmd/`.
  At least 95 of them hold a D5 non-body miss; before D5 exempted added consts, vars and types, the same count was 103.
  The count compares each changed file's top-level declarations and `//go:` lines with the first parent's through `go/parser`.
  It takes every added in-module import as no miss, so it is a lower bound.
  Commits touching a widely linked package, and those of them that still miss:
  `internal/fabricengine` 15 of 26, `internal/configreg` 3 of 12, `internal/configengine` 1 of 6, `internal/lymdirs` 0 of 9.

## Auto-mode assumptions

- A larger coverage binary per package is an acceptable testmap cost.

## Open risks

- Every existing map lacks D2's marker, so every package needs one rebuild before any selection.
  testmap makes it by its present rules, which leave a map without the marker fresh,
  so at the latest at the age limit, or at once through a testmap call naming the package;
  that also delays the full map set `redundancy-sweep-existing` waits on, at the pace note `testmap-backlog-drain` governs.
- Coverage narrows only while a map is fresh.
  By the measured rate under Constraints, landings to `internal/fabricengine` still miss about twice a day,
  so the share of fresh maps is set by the cadence note `testmap-upkeep` governs.
  The tasks raddle files from this record must be ordered after the task that builds `testmap-upkeep`'s design.

## Acceptance criteria

- D2: testmap on a fixture module records, for a test of package B that calls into package A, A's blocks keyed with A's import path,
  A's files and hashes in their own field, an embedded file in a subdirectory of A included,
  and the format marker holding the map's `Commit` and `BuiltAt`.
- D2: testmap records a test the spawn scan marks out-of-process as such.
- D2: testmap leaves a map as fresh when only a linked package's file changed at HEAD and the map's own files and age are unchanged.
- D2: testmap leaves package B's map as it was while linked package A holds a modified or untracked file.
- D2: a linked package's file dirtied after the walk started and still dirty after a package's tests
  makes testmap write no new map for that package and leave any earlier one as it was.
- D2: the redundancy report's verdict for a fixture test is the same whether the map holds cross-package blocks or not, and `mapDrift` reports no other package's file.
