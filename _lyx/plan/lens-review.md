# Lens review

Every lens fork returned a report: coverage, row-validity, forced-edits, guard-reach and way-forward.
Duplicates across lenses are merged into one line, naming each lens that raised it.

## coverage

- coverage, MEDIUM, guard-step pass carry and the `all` run after it unreached: fixed as proposed; card 14's `TestGateTest_Scenario` gains a guard-group repeat that carries and an `all` run after the guard pass that runs the guard packages' tests.
- coverage, MEDIUM, carried summary line never fed to `FailedTier`: fixed as proposed; card 7 lists `internal/gateslot#testFailedTier` and adds a row parsing a carried line beside a failing one.
- coverage, MEDIUM, merge fixer runs its row's key with no row: fixed as proposed; card 5 adds a `TestMergeIn_Fixer` row asserting the fixer's turn gate calls `VerifyCommand` with the spawning row's key.
- coverage + row-validity + way-forward, MEDIUM, `gate_key` override and non-string refusal unreached and without a way forward on the verify rows (card 4) and the landing entries (card 5): fixed differently; the row-validity lens rightly placed card 5's row where the parse cannot be reached, so the rows go where the parse lives instead: `TestResolveGateSpec_ValidOneEntryPerVocabularyName` (default `code-burler-fix` and `gate_key: custom`) and `TestResolveGateSpec_Rejections` (`gate_key: 3`) on card 4, `TestSimpleEntries_HappyPath` (defaults, `custom`, `3`) on card 5, and both refusals now name the way forward of setting `gate_key` to a `gate.yaml` key name or removing it.
- coverage, MEDIUM, `--gate` with an unknown key and the guard group's undeclared-tier and missing-placeholder refusals have no row: fixed as proposed; card 2 adds a `--gate nosuch` row to `TestGateTest_Scopes`, card 14 adds both guard-group refusal rows to `TestPlanTierRuns`.
- coverage, MEDIUM, a resumed run's caller label would land on a `skipped` entry through `persistMarked`'s `PendingBy` hand-off: fixed as proposed; card 17 states a skip write carries the pending label through and adds a `TestStep_EntryGate` row that the label lands on the next verdict.
- coverage, MEDIUM, reportengine timing readers treat a skip as a call: fixed differently; read `internal/reportengine/timingstops.go:125-170` and `timingwaits.go:83-96`, whose stops and halts read only Stuck and Awaiting, so only `internal/reportengine#mergeRows` builds a timed row per entry; card 18 lists `mergeRows` and `TestComputeTiming`, leaving the stop and wait readers unchanged with the reason stated.
- coverage, MEDIUM, no row builds batten's recipe and shows the three rows skip when off: fixed as proposed; card 20 adds a `TestNew_OnDoneChainAndLoadBearingEdges` row over the built Open-Window, Seed-IDE and Open-IDE entry gates.
- coverage, LOW, guard group carried whole unreached: fixed as proposed; card 14 adds the `TestPlanTierRuns` row.
- coverage, LOW, `none` scoper carrying only a whole entry unreached: fixed as proposed; card 7's `TestGreenRecord` pins it.
- coverage, LOW, a whole entry written after carry plus remainder unasserted: fixed as proposed; card 7 adds a third `all` run that carries whole.
- coverage, LOW, exact non-empty args match carrying unreached: fixed as proposed; card 7's `TestGreenRecord` args rule names `-- -count=1` matching and carrying.
- coverage, LOW, card 8's dirty fallback and same-module-directory condition unreached: fixed as proposed; card 8 adds both `TestGateTest_Scopes` rows.
- coverage, LOW, `--slug` and `--since` filtering unreached: fixed as proposed; card 10 adds both `TestGateReport` rows.
- coverage, LOW, Webster's end gate, `lyx webster verify` and the `webster-fix` call unasserted: fixed as proposed; card 3 adds rows to `TestGateParity_VerifyGate` and `internal/websterengine_test#TestFixVerifySpawnOrAttach`.
- coverage, LOW, a skip of a row with an empty `on_done` finishing the run unreached: fixed as proposed; card 17 adds the row.
- coverage, LOW, no check that the board holds the D13 follow-up: fixed as proposed; card 33 names the slug `drop-transitional-tag-spellings` and its `**Verify:**` line runs `lyx board find` for it, checked against the compact `"slug":"…"` output of `lyx board find gate-runs-once`.
- coverage, LOW, empty-targets darn answer unreached: fixed as proposed; card 6 adds a docs-only diff row to `TestDarnTreeScope_ClassifiesAtEachCall`.
- coverage + row-validity, LOW, card 15's untagged-`TestMain` row self-contradictory and a repeat of the first row: fixed differently; the zero count of a package whose events come from an untagged `TestMain` alone moves into a `TestCounters` row, and the scenario row runs a `tier1` test file beside an untagged `TestMain` file, instead of keeping a `TestEmptyTierFailure` row that only repeats the zero-count case.

## row-validity

- row-validity, MEDIUM, card 4's round scenario rows must be rewritten, not dropped, and their fixture is unlisted: fixed as proposed; card 4 lists `internal/loomshed#roundCall` and `internal/loomshed#roundGateConfig` and states the told command is the site's own `--gate code-burler-fix` call through `lyxbin.PutFirstOnPath`, with each lint row's fail-if-wrong input.
- row-validity, MEDIUM, card 7's mid-run commit row has no fixture and the no-slot row cannot fail: fixed as proposed; card 7 lists `internal/gatecli#scenarioConfig` for a tier command that edits and commits a tracked file, and the carried run is made while the row holds every slot through the scenario's `acquire` helper.
- row-validity, MEDIUM, card 8's derive-once row is unobservable: fixed differently; rather than a test-only counting seam, each derivation prints one stderr line naming its base and tiers, which a caller can read in a log, and the row counts those lines.
- row-validity, MEDIUM, card 20's reopen row repeats `SwitchOff` and leaves `SwitchOffAndOpenTerminalForced` driven by the deleted field: fixed as proposed; the repeat row is dropped and the forced row is rewritten onto `Env.Setting`.
- row-validity, MEDIUM, card 21's darn row needs a stepped Shed the test does not have: fixed differently; the row builds `NewDarn` and calls the built Code-Bouncer directly as the loom row does, and Describe's skip rests on `TestNewDarn_Routing`'s false gate and card 17's engine rows, rather than adding a stepped-Shed fixture to a test that has none.
- row-validity, LOW, card 20's new test has no tier or fixture: fixed as proposed; it is an `integration`-tagged hubforge test, retagged `tier1` by card 28.
- row-validity, LOW, card 10's `--since` refusal unreachable from `runReport`: fixed as proposed; the date parses before hub resolution and the row drives `RunCLI`.
- row-validity, LOW, card 1's keep-value branch and card 2's ignored `--card` have no row: fixed as proposed; both rows are added.
- row-validity, LOW, card 17's goto and relaunch rows assert what their tests never do: fixed as proposed; the goto row moves to `TestGoto_StepResumesAtTarget`, which steps, and `TestRelaunch` asserts the written `entered_row`.
- row-validity + guard-reach, LOW and MEDIUM, card 24's Verify line passes vacuously on a `go vet`-only tag and on an empty or refused `--resolved` output: fixed as proposed; the awk checks the `go test` step's `-tags` value and fails when it reads no tier, run against today's hub (fails, as expected before the Constraints' change), a sample of the Constraints' commands (passes) and empty input (fails).

## forced-edits

- forced-edits, BLOCKING, card 1 breaks unlisted test fixtures and seeds: fixed as proposed; card 1 lists `internal/websterengine#gateTestConfig`, `internal/websterengine_test#testGateConfig`, `internal/websterengine_test#TestFixVerifySpawnOrAttach`, `internal/websterengine_test#gateScenario.restart`, both `verifyGateConfig` seeds, `internal/webstercli#TestGateParity_VerifyGate`, `internal/configcli#TestDispatchLayers`, `internal/configcli#TestConfigOverRealHub`, `internal/loomcli#TestGateListSource`, `internal/loomcli#gateConfigWithTwoTiers`, `internal/loomcli#TestLandingDeps_VerifyCommandReadsPlanAtCallTime`, `internal/landingshed#fixedGateTierLists`, `internal/landingshed#TestMergeIn_PlanVerifyFailureRecord` and `internal/loomshed#roundGateConfig`, each moved onto the new shape and names.
- forced-edits, BLOCKING, card 5's re-signed `Deps.VerifyCommand` leaves callers unlisted: fixed as proposed; card 5 lists `internal/landingshed#newGateScratch`, `internal/loomcli#TestDarnTreeScope_ClassifiesAtEachCall` and `internal/testkit/envkit#LandingDeps`.
- forced-edits, BLOCKING, card 20's removal of `CommandDeps.Forced` leaves uses unlisted: fixed as proposed; card 20 lists `internal/battencli#TestWire_MarkerAndReportSeamsRefuseAnAbsentTaskWorktree` and `internal/battencli#stepCommandRows_FollowTheTasksLocalSwitch`.
- forced-edits, MEDIUM, the tmux isolation guard stops seeing retagged packages: fixed as proposed; card 16 lists `cmd/lyx#isolationTags`, `cmd/lyx#isolationTagSets` and `cmd/lyx#TestTmuxIsolation_FixtureTrees`.
- forced-edits, LOW, `-redundancy`'s default tags compile no retagged test: fixed as proposed; card 13 lists `cmd/testtiming#defaultRedundancyTags` and `cmd/testtiming#TestRedundancyTags`.
- forced-edits, LOW, doc files under `**Extends:**` that the cards rewrite: fixed as proposed; `internal/gatecli/doc.go` moves under `**Edit:**` on cards 2, 7, 8 and 14, `internal/loomshed/doc.go` on card 4, `internal/landingshed/doc.go` and `internal/verifytree/doc.go` on card 5, `internal/battencli/cli.go` and `internal/shedadapters/doc.go` on card 20; cards 9, 10 and 15 keep `Extends`, since they only add sections.
- forced-edits, LOW, Webster docs naming `final`, `round` and `--tier <n>...` unlisted: fixed differently; `internal/websterengine/doc.go` joins card 3's `**Edit:**`, but `internal/webstercli/verify.go`'s file comment goes on card 22's `**Prosa:**`, because listing that file beside card 3's `internal/webstercli#websterCLI.runVerify` is refused by `redundant-file-target`.
- forced-edits, LOW, `resolveGateSpec`'s doc names `env.Round`: fixed as proposed; card 4 lists `internal/shedrecipe#resolveGateSpec`.
- forced-edits, LOW, `internal/darnengine/config_test.go`'s header names the deleted `TreeScopedCommand`: fixed differently; the file goes on card 22's `**Prosa:**`, since a `_test.go` file under `**Edit:**` is refused by `edit-test-file` and the header holds no member to name.
- forced-edits, LOW, the gateslot doc is stale from card 1 to card 22: fixed as proposed; `internal/gateslot/doc.go` joins card 1 (keys, scopes, list-shape refusal, migration), card 3 (rendering) and card 14 (retired `guard_tier`), and card 22 adds the tier plan only.

## guard-reach

- guard-reach, MEDIUM, card 17's clearing of `EnteredRow` has no reliable site and a direct self-route never clears it: fixed as proposed; `internal/shedengine#Shed.persistMarked` is re-signed with an explicit `clearEntered` argument, a new `Shed.persistRouted` passes it at every routed call site, `internal/shedengine#Shed.rerouteStuck` included, and a self-bounce row is added.
- guard-reach, MEDIUM, card 15's guard-step non-exemption can be lost because `runGuardStep` appends `-run` to the run's args: fixed as proposed; `emptyTierFailure` reads the call's own `testRequest.args`, and a `TestEmptyTierFailure` row runs a guard-step run that is not exempt.
- guard-reach + way-forward, LOW, nil `Env.SettingKnown` or `Env.Setting` reaches a nil func call: fixed as proposed; card 19's `Build` refuses it as a fail-loud wiring guard modelled on `settingRef`, with a `TestBuild_Errors` row.
- guard-reach, LOW, the retired-`landing` reading of `LandingFailure.Kind` has no reader: fixed differently; the premise holds (`internal/landingshed/fixerrun.go:276` and `internal/landingshed/mergein.go:388` read only `LogPath`), so card 5 states the reading is the record's documented meaning and adds no branch, rather than adding a reader D1 does not ask for.
- guard-reach, LOW, card 14 leaves the guard scan's directory undefined once `guard_tier` goes: fixed as proposed; card 14 scans in `tier0`'s `dir` (or `-C` with explicit targets) and runs the scan before planning the groups.

## way-forward

- way-forward, MEDIUM, a list-shaped `gates` entry after a deploy is told to use `lyx config gate`, which cannot migrate it: fixed as proposed; card 1 refuses the list shape naming `lyx fabric reconcile` or `lyx config reconcile` from the prime, with a `TestLoadConfig` row.
- way-forward, MEDIUM, a leftover `switch` key's way forward still lists `switch` as accepted: fixed as proposed; card 20 rewrites `internal/shedrecipe#commandKeyWayForward` to name `entry_gate`, and `TestCommandEntry` asserts the text.
- way-forward, LOW, the non-bool evaluation error's `lyx config <module>` cannot fix a `run_settings` value: fixed as proposed; card 19's way forward names `lyx config <module> --resolved` to find the layer, then the prime or the task's `run_settings`, then the resume.
- way-forward, LOW, a skip chain longer than the list has no way forward or failed write: fixed as proposed; card 17 writes failed naming the rows and the way forward of fixing the gates' settings or `lyx shed goto`, then resuming.
- way-forward, LOW, card 7's record I/O failures have no stated outcome: fixed as proposed; each is logged and decides nothing.
- way-forward, LOW, `lyx gate report` on a machine with no run log directory: fixed as proposed; `ReadRunRecords` reads a missing directory as no records, with a row.
