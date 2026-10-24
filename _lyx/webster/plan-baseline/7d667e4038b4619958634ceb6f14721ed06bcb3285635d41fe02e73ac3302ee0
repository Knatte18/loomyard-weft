# Card 12 — the shared build step

**Create:**
- `plan:internal/gatecli#execution.runBuild` -> `func (e *execution) runBuild(run tierRun, env []string) (tierResult, bool)`
- `plan:internal/gatecli#execution.reportBlocked` -> `func (e *execution) reportBlocked(run tierRun) runEnd`

**Edit:**
- `internal/gatecli#execution`
- `internal/gatecli#execution.runListed`
- `internal/gatecli#execution.runGreenAware`
- `internal/gatecli#tierRun`
- `internal/gatecli#planTierRuns`
- `internal/gatecli#runEnd`
- `internal/gatecli#runTest`
- `internal/gatecli#reportGroup`
- `internal/gatecli#runReport`
- `internal/gatecli#reportCmd`
- `internal/gatecli#runStandIn`
- `internal/gatecli#standInRecord`
- `internal/gatecli#helperRoleStandIn`
- `internal/gatecli#summaryLogPathOf`
- `internal/gatecli#newCommand`
- `internal/gatecli#TestGateTest_Scenario`
- `internal/gatecli#TestGateTest_Scopes`
- `internal/gatecli#TestPlanTierRuns`
- `internal/gatecli#TestGateReport`
- `pattern/PATTERN-gate-slot.md`
- `internal/gatecli/doc.go`

**Uses:**
- `internal/gatecli#scenarioConfig`
- `internal/gatecli#scopesConfigWith`
- `internal/gatecli#runRecordOf`
- `internal/gatecli#runningGate.records`
- `internal/gateslot#BuildStep`
- `internal/gateslot#BuildSummaryLine`
- `internal/gateslot#BuildTimeoutLine`

**Intent:**
Run the hub's `build` command once per `lyx gate test` call, before the opted-in tiers, inside held slots (Decision 6, Decision 7, Constraint `PATTERN-gate-slot`).

- `tierRun` gains `buildOnly`, which `planTierRuns` sets on a `shared_build` tier whose share is `noTests` (card 9) and that is left with no invocation, so that tier acquires its slots to run the build and runs no test command.
  A `NoCarry` tier keeps its whole invocation under a `noTests` share (card 10), so it never takes `buildOnly`; an opted-in one runs the build, when no earlier opted-in tier ran it, and then its own whole command.
- `execution` gains `buildDone` and `buildFailure`, the failed build's log path.
  The first opted-in tier in plan order whose run will acquire slots, one with invocations left after the green record's subtraction or a `buildOnly` one, runs `runBuild` inside its `runWithSlots` body, before `runTier`, with the slot environment.
  A call whose opted-in tiers are all carried, or whose shares are inert-only, all-inert-card or empty-diff empty scopes, runs no build; an empty `dotnet` diff with a listed `carry: false` opted-in tier runs the build before that tier, which card 10 runs whole.
- `runBuild` runs the build's `command` through the platform shell as a tied child (`runTiedChild`, a logged spawn) in its `dir`, with the call's environment, the slot's inherited variable and the build's `env`; the words after `--` never reach it, and lyx adds no MSBuild or compiler-server switch (Decision 7).
  Its output passes to the call's stdout and into its log, it is bounded by its own `timeout_min` counted from the hold, and the tier's own `timeout_min` starts after it, since `runTier` starts its clock when called.
  A timeout kills its tree, exits 1 and prints and logs `gateslot.BuildTimeoutLine`.
  A build whose shell cannot start, its process nil while `ctx` is not done, is refused as `runInvocations` refuses a tier, naming the build's `dir` and the way forward `make build.dir an existing directory and put the platform shell on PATH, then re-run the same command`.
  The refusal travels in `tierResult.refusal`, so it halts the call with exit 1 as a tier refusal does: no later tier runs, no tier is reported blocked, and no build run record is written, since the build never ran.
  It writes its run record, `Build` set, `Tier` the tier it precedes and scope form `build`, through `gateslot.WriteRunRecord`, and prints `gateslot.BuildSummaryLine`.
- A failed or timed-out build is a failed run counted before the tier it precedes: `runEnd` gains the build's code, `runTest` puts `build (exit <code>)` first among that tier's failures, and the call exits with the first failed run's code in plan order.
  Every opted-in tier from there on runs no command: `reportBlocked` writes its run record, outcome failed with the build's exit code and `BlockedBy` the build's log path, prints `gate test: <tierN> did not run, since the build failed; see <log>` on stderr and no `tier<N>` summary line, and the closing line names it as blocked by the build.
  Tiers not opted in still run.
- `runReport` totals records with `Build` apart: a row gains `build_runs` and `build_seconds`, and such a record adds nothing to `tier_runs` or `wall_seconds`; `reportCmd`'s help text names the two fields.
- `runStandIn` records an invocation whose first argument is `sharedbuild` as a build, exiting with `helperBuildExitEnv`, and holds it under `helperHoldEnv`, so the scenario can stand in for a build command; the comments of `standInRecord.Build` and `helperRoleStandIn` name that build beside the prebuild's.
- `newCommand`'s Long help gains one sentence naming the build step, its own summary line and its run record, and that `--guards` is refused under the `none` and `dotnet` scopers.
- `summaryLogPathOf` also finds the log path of the `gate test: build before tier<N>` line, so a row reads the build's run record and log.
- `PATTERN-gate-slot.md`'s line that every gate build keeps `-trimpath` is narrowed to every Go gate build, since a `dotnet` build takes no Go flags.
- `doc.go` gains a Build step section with these rules and its Bound: a `shared_build` tier whose command lacks `--no-build` builds again inside its own run, since lyx does not read the command; its empty-scope sentence, that such a tier takes no slot, is rewritten for the build-only tier.

Tests:
`TestPlanTierRuns` gains rows: a `noTests` share gives a `shared_build` tier a `buildOnly` run, and gives a `shared_build` `NoCarry` tier its whole invocation with `buildOnly` unset.
`TestGateTest_Scenario` gains rows with a `build: {command: "<stand-in> sharedbuild", dir: sub, env: {BUILD_MARK: "1"}, timeout_min: 1}` and `shared_build` on tier0 and tier6 through `scenarioConfig`'s mutate:
the build runs once, before tier0's invocation, in `sub`, with tier0's slot and `BUILD_MARK` in its environment and none of the call's words after `--` in its arguments, and tier6 runs without a second build; tier0's run record starts no earlier than the build's record ends, so the tier's `timeout_min` is counted after the build;
a build exiting 3 fails tier0 and tier6 without running their commands, prints no failed `tier0` summary line, prints each blocked tier's line naming the build's log, writes their run records with outcome failed, exit 3 and `blocked_by` naming the build's log, still runs a listed tier2 that does not opt in, and exits 3 with the closing line naming the build first;
a build held past a shortened `timeout_min` prints the build timeout line and exits 1, and the row kills the held sleeper from `runningGate.records`, since `tierRecords` leaves out a build;
the build's run record and log, read through `summaryLogPathOf`, are named with `-build-`;
a build whose `dir` is a missing directory is refused with the exact text naming that `dir` and the `make build.dir an existing directory` way forward, exits 1 and runs no tier command, the listed tier2 that does not opt in included;
and a second `--all` over the same clean tree carries both tiers and runs no build.
`TestGateTest_Scopes` gains `dotnet` rows with a stand-in build and `shared_build` on tier0, tier1 not opted in and no longer covering it, sitting before the rows that commit to the pair:
an edit to `cs/Lib/Unused.cs` from card 9's tree, whose type no test class names, runs the build once inside tier0's slots, runs no tier0 command, and runs tier1 whole with the `selected no test class` line; with the build exiting 3 the call exits 3;
an inert-only edit under `inert_paths: ["*.md"]` and an empty diff each run no build;
and an empty diff with a listed `carry: false` tier opting in runs the build before that tier's whole command.
`TestGateReport` gains a build record and asserts it counts in `build_runs` and `build_seconds` only.

**ImpactSummary:** the run loop gains a build path entered only for a hub whose `build` is declared and whose tiers opt in; for every other hub the plan and run loop are unchanged.
