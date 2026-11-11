## Goal

Select an importing package's tests from its coverage map, or name the miss that runs it whole

Takes: testmap-cross-package's map shape, the working tree and fabricengine.Fabric.CodeFileAt for file content at the map's Commit
Delivers: for one importing package's map: the tests selected by changed functions of linked packages per D4, or a D5 miss with its reason (map unreadable or unmarked, Commit unreadable, own files changed, non-body change, //go: line, method set, added init, predeclared name, parse failure, non-Go file change with the go:embed carve-out from inert_paths)
Exports: a selector in internal/testcover returning the selected test names or a miss reason, which coverage-gate-scope calls per importing package

## Scope

The part changes these packages:

- internal/testcover (about 900 changed lines)

## Decisions

### D4 — What a test is selected by

For each importing package, the scope compares every linked package its map records with the working tree.
A linked package's non-test Go file is changed when its working-tree hash differs from the map's recorded hash,
or when it exists on only one side.
The scope reads a changed file's content at the map's `Commit` through `fabricengine.Fabric.CodeFileAt`,
parses it and the working-tree file with `go/parser`,
and finds the changed functions: a function or method present in both whose declaration differs, and one present only at the map's `Commit`.
A declaration compares its receiver, signature and body together, ignoring comments and formatting.
Functions sharing a key name, such as several `init` functions of one file,
count as changed together when any of them differs or their count differs.
A file deleted since the map's `Commit` makes every function it held changed.
A test of the importing package is selected when its map records a covered block of a changed file whose enclosing function is a changed function.
The function names come from the same enclosing-function naming the block keys use.
A change on main since the map was built counts as a change, which can only add tests.

Rationale: block keys carry their enclosing function, so a function-level match needs no block boundaries from a coverage build,
and it selects a superset of the tests a block-level match would.
Comparing from the map's own `Commit` lets a map built before or after the tier's base select on changes inside function declarations;
a change outside them, on either side of the build, is a miss unless D5 exempts it.

### D5 — Map misses widen, never narrow

Each case below runs the importing package whole for the tier, with the reason in the scope report:

- Its map is missing, unreadable, records a build failure, lacks D2's format marker,
  or holds a marker whose `Commit` and `BuiltAt` differ from the map's own.
- Its map's `Commit` is not readable in the worktree's object store.
- Its own Go files, test files included, differ from the map's recorded `Files` in the working tree: a file added, removed or rehashed.
- The tier is one testmap does not map, such as tier 2: every importing package with tests of that tier runs whole, as under `diff`.
- A changed file of a linked package, per D4, has a change outside function declarations other than comments, formatting,
  a reordered or regrouped import block, and an added declaration no unchanged code can reach:
  an added const outside a group that uses `iota` or an implicit value, an added var with no initializer, or an added type with no methods.
  A const group that uses `iota` or an implicit value compares as one declaration, so a spec added to it is a change.
  A changed or removed const, var or type, struct fields included, and an added var with an initializer each count as such a change.
  An added or removed import, named, blank or dot, counts as such a change,
  except an added import of a package the map's linked set holds or of a standard-library package.
- A changed file of a linked package adds, removes or changes a compiler directive line, one opening with `//go:`.
  The comment exception above never covers such a line, wherever it sits in the file.
- A changed file of a linked package changes its method set: it adds a method the map's `Commit` lacks, removes one,
  or changes a method's receiver or signature.
- A changed file of a linked package holds more `init` functions than it did at the map's `Commit`.
- A changed file of a linked package adds a top-level declaration, a function included, whose name is a predeclared identifier
  of the universe scope, such as `max`, `min`, `clear`, `new`, `len`, `error` or `string`.
- A changed file of a linked package does not parse with `go/parser` at the map's `Commit` or in the working tree.
- A file of a linked package's recorded set other than a Go file, other than one `inert_paths` matches,
  differs in the working tree from the map's record, or the working tree holds such a file, not ignored by git, that the record lacks.
  The `inert_paths` exemption never covers a file that a `go:embed` pattern in the linked package's Go files names;
  the scope reads those patterns with `go/parser` from the recorded Go files, and a directory pattern names every file below it.

Rationale: a map tells which tests reached code at the content it hashed; every case above is code or state the map cannot speak for.
A new plain function, a const outside such a group, a var with no initializer or a type with no methods,
under a name that is no predeclared identifier, is reached only through changed code, which selects through D4 or misses here.
Bound: a test that reaches a function of a linked package it did not reach when the map was built, through code unchanged since, is not selected,
nor is a test that relies on the `init` of a standard-library package a changed file newly imports;
only the landing gate catches these.
A dependency-drift miss lasts until testmap rebuilds the map by its present rules:
the package's own files changed at HEAD, the age limit, a failed build at another HEAD, or a testmap call naming the package.
A checkout whose `core.autocrlf` converts line endings rehashes every importing package's own files, so every importing package misses.

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

### Added after Discussion (parent, 2026-11-11): Spawn-marked tests are selected whenever any linked function changed

Decision: The selector selects every test whose spawn verdict is out of process or unresolved whenever any linked function of the package changed, in addition to the tests whose recorded coverage names a changed function. When no linked function changed, it selects none of them on this account. D4's criterion that no test which covered only other functions is selected holds for tests with recorded coverage; a spawn-marked test has no recorded coverage (testmap-upkeep-design D2 and D4), so it cannot be shown unaffected and is selected as the safe widening. Keep the code as it stands.

Rationale: A spawn-marked test runs the code out of process, so a changed linked function can affect it with nothing recorded to prove otherwise. Selecting it widens the run and never narrows it, and it follows D5's rule that misses widen. Refusing it would let a changed function skip a test whose coverage is unknown.

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

## Acceptance criteria

- D4: an edit inside function F of A selects B's tests that covered a block of F and no B test that covered only other functions.
- D4: changing the parameter types of a plain function F of A, its body unchanged, selects B's tests that covered a block of F.
- D4: deleting a file of A selects every B test that covered a block of it.
- D4: a change to function G of A made on main after B's map was built selects B's tests that covered G,
  whether the tier's base predates or postdates that change.
- D4: an edit to one of two `init` functions in a file of A selects every B test that covered a block of either.
- D5: each listed miss case runs the importing package whole and names the reason,
  among them a Go file with an `init` function added to A since the map's `Commit`, an edit to one of B's own test files since then,
  an edit to an embedded file in a subdirectory of A, a changed const in a linked package no B test covered a block of,
  a method of A whose receiver changed from value to pointer with the same body, a changed file of A that does not parse,
  and a marker whose `Commit` and `BuiltAt` differ from the map's.
- D5: an edit to a comment, a reordered import block, an added const in A, or an added import of a standard-library or linked package is no miss;
  a removed import, an added import of any other package, a const added to an `iota` group of A, an added function named `max` in A,
  or an added, removed or changed `//go:` directive line is a miss.
- D5: an edit to an embedded file of A that `inert_paths` matches is a miss.
