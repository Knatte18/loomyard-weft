```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. For each behaviour the diff adds or changes, the test that pins it, or none.

**Per behaviour**
- `RerunMark` and `WriterOnly` (`shuttleengine/gate.go`): pinned through `TestWrapRowGate` rows and `TestWithReachTurn`. There is no direct unit test, and none is needed.
- `WithReachTurn` (`verifytree/reach.go`):
  - Pinned by `TestWithReachTurn` for: a function listed once, one failure short of the budget, a pass resetting the count, the re-run adding nothing, an unreadable listed file, and a pending error.
  - No row pins the `!result.Pending && !result.Terminal` exclusion from the failure count (finding F1).
- `WrapRowGate` writer-only bypass and re-run mark: pinned by the `WriterOnly...` rows, `TheJudgingRunAfterAFixerRunsUnderTheEntrysMark` and the composed turn rows in `rowgate_test.go`.
- `testreach` `ChangedFuncs`, `ChangedLines`, `Fresh`, `Open`, the index, `FuncsReachedBy` and `StaleLinkers`:
  - Pinned by `TestChangedFuncs`, `TestChangedLines`, `TestFresh`, `TestReader` and `TestReaderIndex`.
  - No row pins `ChangedFuncs` matching a name defined in two files on one side only (the ambiguous-name branch).
  - No row pins `Open` skipping a map that names a block or function index outside its tables when it comes from the worktree directory.
  - Neither gap is serious enough to file.
- `PlanLineMutants` (`testcover/linemutant.go`): `TestPlanLineMutants` pins one mutant per line, the negate-over-constant order, string, embedded-file and logger exclusions, and uncovered blocks. Four gaps remain (finding F2):
  - the delete-operand operator in the order;
  - a logger alias import;
  - a local variable shadowing the logger package name;
  - a shifted rune constant.
- `gateslot` config keys and the `reach` report name: pinned by `TestLoadConfig` rows (zero values, `reach` accepted, `mutation` refused, template defaults for `batch` and `darn-write`).
- `TriageLines`, `RenderTriageSection`, and `FailedTests` unchanged by triage lines: pinned by `TestTriageLines` and the new `TestFailedTests` row. `RenderTriageSection` is pinned only through the loomshed and webster rows, which is enough.
- `renderFailureReport` triage budget (dropped first, never causes a cut, a failure's lines kept whole): pinned by the new subtest in `TestRenderFailureReport`.
- `triageLookup.lines` (`gatecli/triage.go`): pinned by `TestTriageLookup_Lines` for the reaching, count, none-plus-command, subtest, unmapped, no-parent, merge-base-failed, changed-read-failed and open-failed cases.
- `execution.addTriage` wiring (`gatecli/triage.go`): `test_integration_test.go` pins only the unmapped line, in a worktree. The prime-has-no-change-set branch (the `prime` closure returning no parent) has no test (finding F3).
- Triage in fixer reports: pinned by `TestFailureReport` (weaver), `TestVerifyGate_FailedEvaluationWritesReport`, `TestRenderVerifyGateFindings` and `TestVerifyFailureFindings`.
- Reach report step (refresh, unrefreshed reasons, budget, prime, red/no-hub/non-go notes, exit unchanged): pinned by `TestGateTestReachStep`.
- `ReachReportMatches` and `ReachPendingOf`: pinned by their subtests in `TestGateTestReachStep`.
- `NoteLines` cap and caveats: pinned by `TestReachReport_NoteLines`.
- `lym gate reach` forms, refusals, cap, landings and the non-go note: pinned by `TestGateReach`. `FirstParentCommits` is pinned through it. The root-commit skip (`commit.Parent == ""`) has no row (finding F4).
- `lym gate mutate`:
  - `TestGateMutate` pins refusals, the non-go note, kill, survive and unjudged, the cap, the uncovered block, an uncommitted file, the stale-map refresh, the mutation budget and the reach budget.
  - Unpinned: the "diff changes no function" early note, `reasonSharedMapStale` and `reasonNoMapToMutate` in the prime, and the unbuildable-mutant count (finding F5).
- Map run writes the index only from the prime with HEAD still: pinned by `TestGateTestMap` and `TestGateTestMapInATaskWorktreeKeepsItsOwnMaps`, including the failed-write row.
- `reach_report` marker (`shedadapters`, `shedrecipe`, `contracts/stencils`): pinned by `TestRenderReachReportNote`, `TestBouncerEntry_HappyPath`, `TestBurlerReviewEntry_RubricStencil` and `TestLoomRubrics`.
- `resolveGateEntry` wrapping the `verify` element with the turn: only `Rerun != nil` is asserted. The `ListedPath` (`row+" gate"`) and the `gateKey` passed to `VerifyReach` are unasserted, but the weaver test covers the batch analogue.
- Pending-source wiring in `loomcli/wiring.go` (`VerifyReach`, `weaverGeom.ReachPending`): `TestWire_ReviewKeysReachTheEnv` asserts `ReachReport` only (finding F6).
- `PATTERN-coverage-map-reader` boundary: pinned by `TestCoverageMapReaderBoundary`.

**Findings**
- F1 — LOW, design. Location: `internal/verifytree/reach.go:118`. Summary: no `TestWithReachTurn` row covers an inner failure that is `Pending` or `Terminal`, which must not add to the failing count. Evidence: `reach_test.go` rows use `innerFailure` only.
- F2 — LOW, design. Location: `internal/testcover/linemutant.go:74` and `internal/testcover/mutant.go:795` (`calleePackage`, `importedPackages`). Summary: D9's operator order is pinned only for negate-over-constant and drop-over-nothing, with no `&&`/`||` operand row. The logger exclusion has no aliased-import or shadowed-local row, and there is no rune-constant row. Evidence: `linemutant_test.go` fixture has no `&&` or `||`, and imports `logger` unaliased.
- F3 — MEDIUM, design. Location: `internal/gatecli/triage.go:306`. Summary: D7 and the added entry "Triage in the prime follows D7" require the prime to get the no-change-set line naming the missing parent branch, with no `lym gate reach` command. Only the pure lookup is tested with a no-parent callback; nothing drives `addTriage` in the prime, so the prime branch could revert to HEAD-as-base unnoticed. Evidence: `test_integration_test.go:1447-1451` and `triage_test.go` (no prime row).
- F4 — LOW, design. Location: `internal/gatecli/reachverb.go:341`. Summary: the root-commit skip (`commit.Parent == ""`) has no row. Evidence: `reachverb_integration_test.go` ranges start at a non-root seed.
- F5 — LOW, design. Location: `internal/gatecli/mutate.go:205` and `:292-309`. Summary: no row for the no-changed-function note, the prime's stale-shared-map reason, the prime's no-map reason, or the unbuildable-mutant count. Evidence: `TestGateMutate` rows are listed above; the prime appears only with `--base` and fresh maps.
- F6 — LOW, design. Location: `internal/loomcli/wiring.go:826` and `:920`. Summary: the wiring of `gatecli.ReachPendingOf` into `weaverGeom.ReachPending` and `Env.VerifyReach` is unasserted, so a dropped assignment silently disables D3's reach turn for the loomshed verify gate and the weaver batch gate. `envkit` lists both as nil-legal. Evidence: `wiring_test.go:949-956` asserts `ReachReport` only.

Tool calls: none; my context already held every diff and test I needed.
