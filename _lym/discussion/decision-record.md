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
- This hub's own `gate.yaml`.

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
- Beside the own-package `Files`, the map records, per covered package other than its own, the set of its Go files and their content hashes, in a field of its own.
- Each test records whether the spawn scan finds it out-of-process or unresolved.
- The map carries a format marker for this shape; a failed-build map carries it too.
- Tests that cover no block of their own package but some block elsewhere now enter the map.

testmap counts a map stale, besides today's rules, when it lacks the marker or when a covered package's recorded file set or hashes differ at HEAD.
The redundancy report keeps its fresh builds on an own-package coverpkg and judges against kept tests' own-package blocks only, so its verdicts do not change;
`mapDrift` keeps comparing own-package files only.

Rationale: a map that holds only its own package's blocks cannot tell which test of an importing package reaches a changed package,
and a map whose dependency drift testmap never repairs would stay a scope miss until the age limit.

### D3 — Changed packages run whole; importing packages run selected tests

The scope derivation carries which packages the diff touched apart from those it added as importers.
A package the diff touches, test files included, runs every test of the tier, as today.
So does a package added only because its tests use the `lym` binary.
A package in the `diff` set only because it imports a changed package runs, per tier, the union of:

- every test D4 selects from its map;
- every test of the tier the package declares at the working tree that its map does not record;
- every test of the tier its map marks out-of-process or unresolved;
- every `//lym:guard` test of the package;
- the whole package on any miss of D5.

A package with no test in that union and no miss runs nothing for that tier.

### D4 — What a test is selected by

For each changed non-test Go file of a changed package, the scope parses the file at the tier's base and in the working tree with `go/parser`
and finds the changed functions: a function or method present in both whose body text differs, and one present only at the base.
A test of an importing package is selected when its map records a covered block of that file whose enclosing function is a changed function.
A file deleted since the base makes every function it held changed.
The function names come from the same enclosing-function naming the block keys use.

Rationale: block keys carry their enclosing function, so a function-level match needs no block boundaries from a coverage build,
and it selects a superset of the tests a block-level match would.

### D5 — Map misses widen, never narrow

Each case below runs the named package whole for the tier, with the reason in the scope report:

- The importing package's map is missing, unreadable, records a build failure, or lacks D2's format marker.
- For a covered package, the map's recorded file set or a recorded hash differs from that package's files at the tier's base.
- The tier is one testmap does not map, such as tier 2: every importing package with tests of that tier runs whole, as under `diff`.
- A changed file of a changed package has a change outside function bodies other than comments, formatting and named imports:
  every importing package of that changed package runs whole.
  A blank or dot import counts as such a change.
- A changed file adds a method or an `init` function the base lacks: every importing package of that changed package runs whole.
- A non-Go file changed inside a changed package's directory, other than one `inert_paths` matches:
  every importing package of that changed package runs whole.

Rationale: a map tells which tests reached code at the content it hashed; every case above is code or state the map cannot speak for.
A new plain function is reached only through a changed caller, whose own function selects through D4.
Bound: a test that reaches a function of a covered package it did not reach when the map was built, through code unchanged since, is not selected;
only the landing gate catches that.

### D6 — Running and recording selected tests

A tier runs its changed packages and whole-run packages as one invocation over the package list, as today,
and its selected packages as a second invocation with an anchored `-run` pattern naming the union of their selected test names.
A name selected in one package also runs in another selected package that declares it; that only adds tests.
A pattern too long for the command budget runs the selected packages whole instead.
The selected invocation's green entry carries its `-run` arguments, so it never serves as a later gate's base and never stands in for a whole run.
`--stop-on-fail`, the failed-test rerun and the verify budget act on both invocations as on one tier run today.

Bound: a tier whose command builds or vets first does so in both invocations; the build cache serves the second.

### D7 — Scope report

The gate's scope output names each package of a coverage-scoped tier with one of: `changed` (whole), `miss: <reason>` (whole),
`selected <n> of <m>`, or `none selected`.

### D8 — The gate only reads maps

The scope reads the hub's maps under `<BoltDir>/.lym/testcover` through `testcover.ReadMap` and never starts a build or a refresh.
A read that fails is a miss per D5.

## Constraints

- Go 1.26 (`go.mod`, toolchain go1.26.8): `go test -c -cover -coverpkg=<list>` instruments every listed package, and the set-mode profile lists blocks of each by file.
- PATTERN-config-layers: `coverage` is a new value, not a changed meaning of `diff`; `gateslot` stays a strict loader, so a machine on an older binary refuses a hub file holding `coverage` until it updates.
- PATTERN-tolerant-records: a map read never refuses an unknown key; D2's fields and marker are added fields.
- PATTERN-fabric-boundary: base file content for D4 is read through the path the impactset diff already uses, never a new raw git call.
- PATTERN-gate-slot: map builds stay in `lym gate testmap`'s held slots; the scope reads maps inside the gate's own slot.
- PATTERN-test-speed and PATTERN-test-isolation: each test takes the lowest tier that fits; fixtures use hermetic git and a fixture module.

## Auto-mode assumptions

- The cost the board names is tiers 0 and 1 across importing packages.
- A larger coverage binary per package, and testmap rebuilding maps whose covered dependencies drifted, are acceptable testmap costs.

## Open risks

- Question for the parent: D2 raises testmap's build cost and rebuild rate, the cost note `testmap-upkeep` calls too large and orders before this task.
  May this task change the map format and staleness now, or should D2 wait for that note's policy?
- Question for the parent: this hub's `_bolt/_lym/config/gate.yaml` writes `scope: diff` for `darn-write`, `webster-fix` and `code-burler-fix`,
  so the template default does not reach the gate the board entry names. Is the switch an operator step after landing, or should the parent record it for the run?
- Hit rate: a map built at a later main than the worktree's base reads as a miss for every covered package that changed since; with every map a miss the gate runs as `diff` does today.
- Every existing map lacks D2's marker, so every package needs one rebuild before any selection;
  that also delays the full map set `redundancy-sweep-existing` waits on, at the pace note `testmap-backlog-drain` governs.
- Tier 2 keeps the import scope; if tier 2 dominates a Darn-Write gate's time, this task does not cut it.

## Acceptance criteria

- D1: a gate key with `scope: coverage` loads; the scope error for an unknown value names `coverage` among the allowed values.
- D1: under scoper `none` or `dotnet`, a `coverage` key computes the same targets as `diff`.
- D1: a package `diff` would not run never runs under `coverage`; watch skips, the whole-tree fallbacks and `inert_paths` act as under `diff`.
- D1: a `coverage` call with explicit targets, `tree_scope`'s included, runs those targets un-narrowed.
- D1: the template's `darn-write`, `webster-fix` and `code-burler-fix` keys hold `scope: coverage` and `pr-review` holds `scope: diff`.
- Review: D1's template comment on the gate keys names `coverage` and which keys default to it.
- D2: testmap on a fixture module records, for a test of package B that calls into package A, A's blocks keyed with A's import path, A's file set and hashes in their own field, and the format marker.
- D2: testmap records a test the spawn scan marks out-of-process as such.
- D2: testmap counts a map stale when a covered package's file changed at HEAD or the marker is missing, and not stale when only an uncovered package changed.
- D2: the redundancy report's verdict for a fixture test is the same whether the map holds cross-package blocks or not, and `mapDrift` reports no other package's file.
- D3: a changed package runs every test of the tier; an importing package runs at least the D3 union and nothing outside it but D6's same-name overlap.
- D3: an importing package's guard test, out-of-process test and test absent from its map run even when D4 selects nothing.
- D3: an importing package with no test in the D3 union and no miss runs no test.
- D4: an edit inside function F of A selects B's tests that covered a block of F and no B test that covered only other functions.
- D4: deleting a file of A selects every B test that covered a block of it.
- D5: each listed miss case runs the importing package whole and names the reason, a new file in a covered package included.
- D5: an edit to a comment or a named import outside function bodies is no miss.
- D6: a selected-test pass leaves a green entry carrying its `-run` arguments, which a later gate never reads as the package passed or the tier whole.
- D6: an over-budget selected pattern runs the selected packages whole.
- D6: a failing selected test is rerun by the failed-test rerun and stops the gate under `--stop-on-fail`.
- D7: the scope output lists each package with `changed`, `miss: <reason>`, `selected <n> of <m>` or `none selected`.
- D8: an unreadable map file runs that package whole; computing a scope starts no testmap build.

## Notes for the plan writer

- The scope is formed in `internal/gatecli/scope.go` (`computeScope`, and its gate-key switch) over `impactset.DeriveTargets`, whose result today is a flat package list; the scope value is validated in `internal/gateslot/config.go`.
- Map reading is `testcover.ReadMap`; testmap's tier list and staleness are in `internal/gatecli/testmap.go`; `testcover.Toolchain.BuildBinary` already takes a comma-separated coverpkg list.
- The anchored `-run` form already exists in `gateslot.FailedTestsCall`; the green record's entries and the command budget are in `internal/gatecli/green.go`.
- The redundancy judgment's own-package filter can reuse `testcover.PackageBlocks`.
