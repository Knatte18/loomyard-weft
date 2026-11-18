# Card 5 — reach-step

**Create:**
- `plan:internal/gatecli#ReachReport` -> `type ReachReport struct`
- `plan:internal/gatecli#ReachFunction` -> `type ReachFunction struct`
- `plan:internal/gatecli#ReachReport.NoteLines` -> `func (r ReachReport) NoteLines(path string) []string`
- `plan:internal/gatecli#reachStep` -> `type reachStep struct`
- `plan:internal/gatecli#execution.reachStep` -> `func (e *execution) reachStep() reachStep`
- `plan:internal/gatecli#reachStep.run` -> `func (s reachStep) run(red, inHub bool)`
- `plan:internal/gatecli#changedFuncsAt` -> `func changedFuncsAt(head redundancyTree, fabric *fabricengine.Fabric, base string, module goModule, changed []string) ([]testreach.ChangedFunc, error)`
- `plan:internal/gatecli#TestReachReport_NoteLines` -> `func TestReachReport_NoteLines(t *testing.T)`
- `plan:internal/gatecli#TestGateTestReachStep` -> `func TestGateTestReachStep(t *testing.T)`

**Edit:**
- `internal/gatecli#runTest`
- `internal/gatecli#testMapRun`
- `internal/gatecli#testMapRun.mapPackages`
- `plan:cmd/lym#TestCoverageMapReaderBoundary`
- `PATTERN.md`
- `pattern/PATTERN-test-speed.md`
- `docs/overview.md`
- `internal/gatecli/doc.go`
- `internal/gateslot/template.yaml`
- `internal/gateslot/doc.go`
- `pattern/PATTERN-gate-slot.md`

**Uses:**
- `plan:internal/testreach#Open`
- `plan:internal/testreach#Reader.Maps`
- `plan:internal/testreach#ChangedFuncs`
- `plan:internal/testreach#Fresh`
- `plan:internal/testreach#ReadPackageMap`
- `plan:internal/testreach#Reader.TestsReaching`
- `plan:internal/testreach#Reader.SpawnMarked`
- `plan:internal/testreach#Reader.StaleLinkers`
- `plan:internal/gateslot#ReportReach`
- `internal/gatecli#parentMergeBase`
- `internal/gatecli#redundancyModule`
- `internal/gatecli#goFileHashes`
- `internal/gatecli#workingTree`
- `internal/gatecli#commitTree`
- `internal/gatecli#coverageMapsDir`
- `internal/gatecli#seedTestMapModule`
- `internal/gatecli#runTestMapStandIn`
- `internal/gatecli#readTestMap`
- `internal/hubgeom#IsPrime`
- `internal/impactset#ChangedPaths`
- `internal/testcover#Toolchain.LinkedPackages`

**Intent:**
Add the `reach` report step of `lym gate test` (D3), its report `reach.json` and the note it prints (D4).
Card 7 adds the report's reader and its HEAD match (D5), with their first caller.

Run order: in `runTest`, a key whose `reports` lists `gateslot.ReportReach` runs the step after its lints and before the redundancy step, under the same memory guard, in an uncancelled call.

`changedFuncsAt(head, fabric, base, module, changed)` is the one changed-function finder of gatecli (D2's callers):
for each package directory holding a changed production Go file, it puts every production Go file of the directory at `base` (through `commitTree`, so through fabric) and in `head` into two `testreach.FileSet`s,
then calls `testreach.ChangedFuncs`.

`reachStep` carries the call's fields as `redundancyStep` does, including `minuteUnit`, the length of one minute of `reach_budget_min`, so a test drives the budget with a hand-built step.

`reachStep.run(red, inHub)` writes `ReachReport` through `internal/state` as `reach.json` in the worktree's gate directory (`gateslot.WaitDir(location.AnchorPath())`; the name is a constant) and prints its note to stderr.
- A red tier, a call outside a hub and a scoper other than `go` give a report with `Note` set (and `RedGate` for red) that judges nothing and makes no lookup, as the redundancy step does;
  so does a task worktree with no parent branch. The printed line says which.
  Nothing the step meets changes the exit code; every failure is an `Errors` line, and a write failure is logged at Warn.
- The base is the merge base with the parent branch (`parentMergeBase`).
  In the prime (`hubgeom.IsPrime`), which has no parent branch, the base is HEAD, so the step answers for the working tree's uncommitted changes; the report records `Prime`.
  It then takes the changed paths (`impactset.ChangedPaths`) and the changed functions (`changedFuncsAt` over `workingTree`).
- Refresh (D3): only in a task worktree; a prime call refreshes nothing and answers from the shared maps.
  For each package holding a changed function, the step reads its worktree anchor map through `testreach.ReadPackageMap`.
  The package is refreshed only when that map is absent or `testreach.Fresh` is false for it,
  and only when neither it nor a linked in-module package has a modified or untracked file (`impactset.ChangedPaths` against HEAD, by package ownership).
  The linked packages are those the map's `Linked` records, or, for a package with no map, the in-module packages `testcover.Toolchain.LinkedPackages` lists for its test binary.
  The refresh runs the map run's measurement (`testMapRun` with `mapsDir` set to the worktree's anchor maps, `testcover.Dir(location.AnchorPath())`, and the packages as its jobs), so it holds a gate slot and passes the map run's own clean checks;
  it never writes the hub's shared maps.
  `testMapRun` gains a `deadline` field: `mapPackages` starts no package past it and lists each remaining one unmeasured with the budget reason; a zero deadline sets no bound, so `lym gate test --map` is unchanged.
  The step sets the deadline from `reach_budget_min` in `minuteUnit`s, counted from the slot's hold.
- Each package left unrefreshed gets an `Unrefreshed` line with its reason and its way forward:
  uncommitted files, naming the dirty package, its own or a linked one: commit and run the gate again;
  past `reach_budget_min`: run the gate again, since a package refreshed now costs nothing on the next call;
  the call runs in the prime: the prime keeps no worktree maps, and its shared maps are rebuilt with `lym gate test --map`.
- It then opens `testreach.Open` over the shared maps and, outside the prime, the anchor maps, and records per changed function a `ReachFunction`:
  `Package`, `File`, `Func`, `Hash`, `New`, its tests from `TestsReaching` (own package first, fewest functions first, at most five kept, with `MoreTests` counting the rest),
  and its caveats: for an unreached function whose package holds a tier2 test (by `testcover.ScanTestFuncs`) or a spawn-marked entry (`SpawnMarked`), that such a test may run it unseen;
  for an unreached function `New` since the base whose package `StaleLinkers` names packages for, that those packages' tests may reach it, naming them.
- A reader that read no map at all (`Maps()` is empty), as in a hub whose `lym gate test --map` never ran, gives a report that judges nothing:
  its `Note` says no coverage map exists and to build the maps with `lym gate test --map` from the prime, so no function reads as unreached for want of maps.
- The report records `GateKey`, `Written`, `Base`, `Prime`, `ModuleDir`, the `Maps` it read (package, commit, build time, worktree or shared), the `Unrefreshed` lines,
  and `FileHashes`: the Go files, test files included, of each package holding a changed function and of each package whose tests it names, by `goFileHashes`.
  Report names, the file name and the caps (five tests per function, the note's line cap) are named constants.

`ReachReport.NoteLines(path)` renders D4's note: per changed function its tests, own package first, then the count of the rest; the commit and age of each map read;
then the changed functions no mapped test reaches, with their caveats; then the `Unrefreshed` lines.
It is capped at the note's line cap, and a cut note ends with a line naming `path` for the rest.
A report with a `Note` renders that note alone.

The card's guard clause: `TestCoverageMapReaderBoundary` also fails when a non-test file outside `internal/testreach` calls `testreach.ChangedFuncs`, other than the gatecli file holding `changedFuncsAt`, which it pins,
so gatecli keeps one changed-function finder.

Tests:
- `TestReachReport_NoteLines` (tier0) is one table over built reports: own-package tests before other packages', fewer-functions first within each, at most five per function plus the count of the rest, each map's commit and age named;
  the tier2/spawn caveat and the stale-linker caveat on unreached functions; a judged-nothing report renders its note alone; a note over the cap ends naming the path.
- `TestGateTestReachStep` (tier1) reuses the map tests' hub fixture (`seedTestMapModule`, the stand-in `go` of `runTestMapStandIn`) in a task worktree with a parent branch:
  - a passing `darn-write` call listing `reach` writes `reach.json` and prints the note, and its exit code equals the same call's without the step;
  - rows for a red tier, a non-Go scoper, a call outside a hub and a task worktree with no parent branch each give a report with its note and no lookup, exit unchanged;
  - a package whose anchor map matches is not refreshed (the stand-in records no build of it);
  - a package whose anchor map is stale, and one with no anchor map, are refreshed: the stand-in records each build, the no-map package's linked packages come from the stand-in's dependency list,
    and each anchor map is written with hashes matching the worktree;
  - a package with an uncommitted file, and one whose linked package has one (named in its line), are answered from the existing maps with their lines and way forward;
  - a prime call with an uncommitted change refreshes nothing and answers from the shared maps with its line, and its report carries `Prime`;
  - a prime call in a hub with no shared maps gives a report that judges nothing, whose note names `lym gate test --map`;
  - a package holding a tier2 test file gets the caveat on its unreached function;
  - no row's refresh writes the shared maps (their bytes are unchanged);
  - The budget row drives a hand-built `reachStep`, as `stepAt` drives the redundancy step, with a `minuteUnit` so short the budget is spent at the hold:
    each package is answered from the existing maps with its budget line.

PATTERN changes (`PATTERN-test-speed` — changes): the overview line becomes

```markdown
- `PATTERN-test-speed` — Writing or running a test: via `lym gate test` by key; one `tiergate.Require` tier, lowest that fits; it pins behavior no other test covers, which the redundancy and reach reports check; skips, clocks, `t.Parallel`, `//lym:seam` and `//lym:watch` per the background. (test) — [background](pattern/PATTERN-test-speed.md)
```

`pattern/PATTERN-test-speed.md`: the coverage-map section's "built only on command" sentence changes to say that the hub's shared maps are built only on command,
while the `reach` step refreshes a task worktree's own anchor maps of the packages holding changed functions, under its budget and the map run's clean checks;
a new bullet describes the `reach` step: advisory, before the other report steps, its report and note, its bounds.
The "Tests run through the gate" bullet that counts "Three redundancy runs" names the runs without a count and adds the `reach` step's refresh.

`internal/gatecli/doc.go` gains a `# Reach report` section, and its map-run sentence "It runs only on command: no dobby, landing or gate starts it" says that the `reach` step also runs it, over a task worktree's anchor maps.
Its memory-guard sentence naming the runs the guard covers adds the reach step's refresh.
`docs/overview.md`'s gate passage names the `reach` step beside the `redundancy` step, and its sentences "No dobby builds the coverage maps: the operator builds them on command" and "only on command" say the same of the shared maps and name the reach step's refresh of a worktree's own.
`internal/gateslot/template.yaml`'s comments on `memory_quit_mb`, `redundancy_build_timeout_min`, `redundancy_test_timeout_sec` and `redundancy_test_memory_mb`, which name the runs they bound, add the reach step's refresh.

The refresh is a new place that holds gate slots (`PATTERN-gate-slot`):
- `pattern/PATTERN-gate-slot.md`'s Gate slot section gains, beside the map run of `lym gate test --map`, a refresh of a task worktree's own anchor maps,
  which runs the map run's measurement in the map run's slot ask; its lead bullet names the places without a count.
- `internal/gateslot/doc.go`'s bounds paragraph names the refresh where it says a map call holds the larger of tier0's and tier1's slot asks.
  Its sentences naming the runs that take the three `redundancy_*` bounds, and the runs that run fewer tests side by side when memory is short, add the reach step's refresh.
- `docs/overview.md`'s `lym gate test --map` sentence says the reach step's refresh runs the same measurement under the same build and test bounds and memory cap,
  and its memory-guard sentence names the refresh beside the redundancy report step.

**ImpactSummary:** The gate call gains one advisory step that never changes its exit; `testMapRun` gains a deadline whose zero value keeps `lym gate test --map` as it is; the PATTERN line, its background, the gate-slot pattern, the gatecli and gateslot docs, the overview and the template comments that say no gate builds maps or list the bounded runs and slot holders change in place.
