# Decision record — gate-scope-by-coverage-map

## Goal

A diff-scoped verify gate runs the changed packages whole and, in every package that only imports a changed one,
only the tests whose recorded coverage reaches the changed code.
Any doubt about what a test reaches widens that package back to today's whole run.
The landing gate keeps running everything.

## Scope

In:

- A new gate scope value, `coverage`, and its use by the diff-scoped Go gate keys.
- Coverage maps that record the blocks a test covers in every in-module package its binary links, not only its own.
- Test selection from those maps, the miss rules that widen it, and a per-package scope report.

Out:

- Partial map updates, idle-time builds, the refresh trigger and sharing maps across machines: note `testmap-upkeep`.
- Draining the map backlog: note `testmap-backlog-drain`.
- Cutting existing redundant tests: note `redundancy-sweep-existing`.
- The whole-tree keys (`webster`, `parent-align`, `finalize`, `adopt`) and the card keys: unchanged.
- The `dotnet` and `none` scopers: `coverage` acts as `diff` under them.
- This hub's own `gate.yaml`:
  the operator switches its `darn-write`, `webster-fix` and `code-burler-fix` keys to `coverage` with `lym config gate` from the prime after landing.

## Decisions

### D1 — A `coverage` gate scope

A gate key may hold `scope: coverage`, beside `card`, `diff` and `all`.
It is the computed `diff` scope narrowed per D3 to D6 when the scoper is `go`, and plain `diff` under any other scoper.
The template's `darn-write`, `webster-fix` and `code-burler-fix` keys default to `coverage`; `pr-review` stays `diff`.
Under `coverage` the scope is first derived exactly as under `diff`: the impactset base per tier, the changed and importing packages,
the whole-tree fallbacks, `//lym:guard` and `//lym:watch` tests and `inert_paths`.
Coverage then only removes tests from packages that are not changed; it never adds a package `diff` would not run.
A call with explicit targets is never narrowed: Darn's `tree_scope` targets, the failed-test rerun and a hand-named target list run as today.

Rationale: a new value keeps `diff`'s meaning, so no reinterpretation is needed (PATTERN-config-layers),
and `pr-review` keeps the wider scope as the last gate before the landing gate.
Bound: the scope can skip tests of importing packages only; the `finalize` landing gate, scope `all`, still runs every test before a landing.

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
After a package's tests end, testmap repeats its dirty check over every linked package's files.
When one is dirty, it writes no map for that package, as it does when files changed at HEAD.
A map's recorded hashes are therefore the content at its `Commit`.
testmap counts a map stale, besides today's rules, when it lacks the marker or when a linked package's recorded file set or hashes differ at HEAD,
whether or not a test covered a block of that package.
The redundancy report keeps its fresh builds on an own-package coverpkg and judges against kept tests' own-package blocks only, so its verdicts do not change;
`mapDrift` keeps comparing own-package files only.

Rationale: a map that holds only its own package's blocks cannot tell which test of an importing package reaches a changed package,
and a map whose dependency drift testmap never repairs would select from an ever older `Commit` until the age limit.

### D3 — Changed packages run whole; importing packages run selected tests

The scope derivation carries which packages the diff touched apart from those it added as importers.
A package the diff touches, test files included, runs every test of the tier, as today.
So does a package added only because its tests use the `lym` binary.
A package in the `diff` set only because it imports a changed package runs, per tier, the union of:

- every test D4 selects from its map;
- every test of the tier the package declares at the working tree that its map does not record;
- every test of the tier its map marks out-of-process or unresolved;
- every `//lym:guard` test of the tier the package declares;
- the whole package on any miss of D5.

The union holds only tests of the tier and drops the package's tests the diff's watch skip names.
A package whose union is then empty, with no miss, counts as `none selected` and runs nothing for that tier;
D6's selected invocation leaves it out.

Bound: a test that runs changed code in a child process the spawn scan does not see is mapped as in-process and is not selected;
only the landing gate catches it.
The scan does not follow a call into another in-module package that is not a testkit,
gives every test in a tier0 file an empty verdict,
and does not see a hook or alias that makes git run module code.

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
- A changed file of a linked package does not parse with `go/parser` at the map's `Commit` or in the working tree.
- A file of a linked package's recorded set other than a Go file, other than one `inert_paths` matches,
  differs in the working tree from the map's record, or the working tree holds such a file, not ignored by git, that the record lacks.

Rationale: a map tells which tests reached code at the content it hashed; every case above is code or state the map cannot speak for.
A new plain function, a const outside such a group, a var with no initializer or a type with no methods is reached only through changed code,
which selects through D4 or misses here.
Bound: a test that reaches a function of a linked package it did not reach when the map was built, through code unchanged since, is not selected,
nor is a test that relies on the `init` of a standard-library package a changed file newly imports,
nor one whose unchanged code dot-imports a linked package and resolves a predeclared name to a declaration added there;
only the landing gate catches these.
A checkout whose `core.autocrlf` converts line endings rehashes every importing package's own files, so every importing package misses.

### D6 — Running and recording selected tests

A tier runs its changed packages and whole-run packages as one invocation over the package list, as today,
and its selected packages as a second invocation with an anchored `-run` pattern naming the union of their selected test names.
A name selected in one package also runs in another selected package that declares it; that only adds tests.
A pattern too long for the command budget runs the selected packages whole instead.
The selected invocation's green entry carries its `-run` arguments, so it never serves as a later gate's base and never stands in for a whole run.
`--stop-on-fail`, the failed-test rerun and the verify budget act on both invocations as on one tier run today.

Bound: a tier whose command builds or vets first does so in both invocations; the build cache serves the second.

### D7 — Scope report

The gate's scope output names each package of a coverage-scoped tier with one of: `changed` (whole), `whole: lym binary`,
`miss: <reason>` (whole), `whole: over budget`, `selected <n> of <m>`, or `none selected`.

### D8 — The gate only reads maps

The scope reads the hub's maps under `<BoltDir>/.lym/testcover` through `testcover.ReadMap` and never starts a build or a refresh.
A read that fails is a miss per D5.

### Added after Discussion (parent, 2026-11-11): D2 leaves testmap's staleness rule as it is

Decision: F4/Q10: no. The writer removes D2's staleness rule and states in D5 that a dependency-drift miss lasts until testmap rebuilds the map by its present rules (own files or the age limit). The dependency-drift rule D2 proposed goes into note testmap-upkeep as an input, not into this record.

Rationale: Note testmap-upkeep owns the whole staleness and refresh policy, dependency drift included, and has a research step of its own still to run; this record decides how the gate scopes by a map, not when a map goes stale.

### Added after Discussion (parent, 2026-11-11): This motif lands now; the build parts wait on the upkeep design

Decision: Q11: the motif lands its record now without depending on testmap-upkeep. The record states, under Open risks, that coverage narrows only while a map is fresh, that the fresh-map share is set by testmap-upkeep's cadence, and that the tasks raddle files from this record must be ordered after the task that builds testmap-upkeep's design.

Rationale: A motif lands no code, so landing the design costs nothing that the upkeep cadence could make wrong; the ordering belongs to the build tasks, which raddle files later.

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

- A larger coverage binary per package, and testmap rebuilding maps whose linked dependencies drifted, are acceptable testmap costs.

## Open risks

- Question for the parent: D2 changes testmap's staleness rule and build cost, over every linked package, covered or not,
  and note `testmap-upkeep` says it owns the whole staleness policy, dependency drift included.
  May D2 change that policy now, or should it wait for that note?
- Every existing map lacks D2's marker, so every package needs one rebuild before any selection;
  that also delays the full map set `redundancy-sweep-existing` waits on, at the pace note `testmap-backlog-drain` governs.
- Question for the parent: by the measured rate under Constraints, landings to `internal/fabricengine` still miss about twice a day.
  A package linking it narrows only while its map is younger than the last such landing,
  so `coverage` stays close to `diff` unless testmap rebuilds a map soon after each landing, a cadence note `testmap-upkeep` governs.
  Should this task land before that note sets the cadence, or wait for it?

## Acceptance criteria

- D1: a gate key with `scope: coverage` loads; the scope error for an unknown value names `coverage` among the allowed values.
- D1: under scoper `none` or `dotnet`, a `coverage` key computes the same targets as `diff`.
- D1: a package `diff` would not run never runs under `coverage`; watch skips, the whole-tree fallbacks and `inert_paths` act as under `diff`.
- D1: a `coverage` call with explicit targets, `tree_scope`'s included, runs those targets un-narrowed.
- D1: the template's `darn-write`, `webster-fix` and `code-burler-fix` keys hold `scope: coverage` and `pr-review` holds `scope: diff`.
- Review: D1's template comment on the gate keys names `coverage` and which keys default to it.
- D2: testmap on a fixture module records, for a test of package B that calls into package A, A's blocks keyed with A's import path,
  A's files and hashes in their own field, an embedded file in a subdirectory of A included,
  and the format marker holding the map's `Commit` and `BuiltAt`.
- D2: testmap records a test the spawn scan marks out-of-process as such.
- D2: testmap counts a map stale when the marker is missing or a linked package's recorded file changed at HEAD,
  a linked package no test covered a block of included, and not stale when only a package B's test binary does not link changed.
- D2: testmap leaves package B's map as it was while linked package A holds a modified or untracked file.
- D2: the redundancy report's verdict for a fixture test is the same whether the map holds cross-package blocks or not, and `mapDrift` reports no other package's file.
- D3: a changed package runs every test of the tier; an importing package runs at least the D3 union
  and nothing outside it but D6's same-name overlap and D6's over-budget whole run.
- D3: an importing package's guard test, out-of-process test and test absent from its map, each of the tier and not named by the watch skip,
  run even when D4 selects nothing.
- D3: an importing package with no test in the D3 union and no miss runs no test.
- D3: an importing package whose only union member is an unwatched watch-marked guard test runs no test, and the gate passes.
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
  a removed import, an added import of any other package, a const added to an `iota` group of A,
  or an added, removed or changed `//go:` directive line is a miss.
- D6: a selected-test pass leaves a green entry carrying its `-run` arguments, which a later gate never reads as the package passed or the tier whole.
- D6: an over-budget selected pattern runs the selected packages whole.
- D6: a failing selected test is rerun by the failed-test rerun and stops the gate under `--stop-on-fail`.
- D7: the scope output lists each package with `changed`, `whole: lym binary`, `miss: <reason>`, `whole: over budget`,
  `selected <n> of <m>` or `none selected`.
- D8: an unreadable map file runs that package whole; computing a scope starts no testmap build.

## Notes for the plan writer

- The scope is formed in `internal/gatecli/scope.go` (`computeScope`, and its gate-key switch) over `impactset.DeriveTargets`, whose result today is a flat package list; the scope value is validated in `internal/gateslot/config.go`.
- Map reading is `testcover.ReadMap`; testmap's tier list and staleness are in `internal/gatecli/testmap.go`; `testcover.Toolchain.BuildBinary` already takes a comma-separated coverpkg list.
- testmap lists a package's files through `fabricengine.Fabric.CodeDirAt`, which returns only a directory's direct files;
  D2's linked-package file set also needs the files in subdirectories outside nested package directories.
- The anchored `-run` form already exists in `gateslot.FailedTestsCall`; the green record's entries and the command budget are in `internal/gatecli/green.go`.
- The redundancy judgment builds its kept set from the map's `Blocks` keys; its own-package filter keeps the keys that carry no import path.
