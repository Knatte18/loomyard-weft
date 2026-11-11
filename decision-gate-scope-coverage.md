## Goal

A coverage gate scope runs changed packages whole and importing packages' selected tests

Takes: coverage-test-select's selector and the diff scope as impactset.DeriveTargets and computeScope derive it
Delivers: scope: coverage in gateslot and the template's darn-write, webster-fix and code-burler-fix keys; DeriveTargets keeping changed and lym-binary packages apart from importers; the D3 union per importing package and tier; a second tier invocation with an anchored -run in the run's own args, its green entry carrying them, over-budget fallback to whole; the per-package scope report; maps read only, never built
Exports: the coverage scope value and impactset.Targets' split of changed packages from importers

## Scope

The part changes these packages:

- internal/gateslot (about 80 changed lines)
- internal/impactset (about 150 changed lines)
- internal/gatecli (about 750 changed lines)

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

The union holds only tests of the tier declared in a test file of the package that build constraints admit on the gate's host,
admitted as `holdsTierTest` in `internal/gatecli/tierrun.go` admits one, through `go/build` `Context.MatchFile` with the tier's tag;
it drops the package's tests the diff's watch skip names.
A package whose union is then empty, with no miss, counts as `none selected` and runs nothing for that tier;
D6's selected invocation leaves it out.

Bound: a test that runs changed code in a child process the spawn scan does not see is mapped as in-process and is not selected;
only the landing gate catches it.
The scan does not follow a call into another in-module package that is not a testkit,
gives every test in a tier0 file an empty verdict,
and does not see a hook or alias that makes git run module code.

### D6 — Running and recording selected tests

A tier runs its changed packages and whole-run packages as one invocation over the package list, as today,
and its selected packages as a second invocation with an anchored `-run` pattern naming the union of their selected test names.
A name selected in one package also runs in another selected package that declares it; that only adds tests.
A pattern too long for the command budget runs the selected packages whole instead.
The selected invocation's green entry carries its `-run` arguments, so it never serves as a later gate's base and never stands in for a whole run.
The selected invocation's `-run` arguments sit in the run's own args, not the call's,
because a filter among the call's args exempts the whole tier run from the empty-tier check.
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

## Acceptance criteria

- D1: a gate key with `scope: coverage` loads; the scope error for an unknown value names `coverage` among the allowed values.
- D1: under scoper `none` or `dotnet`, a `coverage` key computes the same targets as `diff`.
- D1: a package `diff` would not run never runs under `coverage`; watch skips, the whole-tree fallbacks and `inert_paths` act as under `diff`.
- D1: a `coverage` call with explicit targets, `tree_scope`'s included, runs those targets un-narrowed.
- D1: the template's `darn-write`, `webster-fix` and `code-burler-fix` keys hold `scope: coverage` and `pr-review` holds `scope: diff`.
- Review: D1's template comment on the gate keys names `coverage` and which keys default to it.
- D3: a changed package runs every test of the tier; an importing package runs at least the D3 union
  and nothing outside it but D6's same-name overlap and D6's over-budget whole run.
- D3: an importing package's guard test, out-of-process test and test absent from its map, each of the tier and not named by the watch skip,
  run even when D4 selects nothing.
- D3: an importing package with no test in the D3 union and no miss runs no test.
- D3: an importing package whose only union member is an unwatched watch-marked guard test runs no test, and the gate passes.
- D3: an importing package whose only unrecorded declared test sits in a test file constrained to another GOOS runs no test,
  and the gate passes.
- D6: a selected-test pass leaves a green entry carrying its `-run` arguments, which a later gate never reads as the package passed or the tier whole.
- D6: an over-budget selected pattern runs the selected packages whole.
- D6: a failing selected test is rerun by the failed-test rerun and stops the gate under `--stop-on-fail`.
- D7: the scope output lists each package with `changed`, `whole: lym binary`, `miss: <reason>`, `whole: over budget`,
  `selected <n> of <m>` or `none selected`.
- D8: an unreadable map file runs that package whole; computing a scope starts no testmap build.
