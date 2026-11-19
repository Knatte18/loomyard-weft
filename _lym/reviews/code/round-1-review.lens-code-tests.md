```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. For each behaviour the diff adds or changes, the test that pins it, or none.

**Pinned (test named)**
- `--copies` flag refusals (negative, no `--gate`, no targets): `TestGateTest_RefusesBeforeResolvingTheHub`.
- `--copies` key refusals (tier 3 or more than one tier): the `gateKeyRefusal` rows in `TestRunTierCopies`.
- `--copies` lease size, clamp line, one-copy-for-a-`slots: all`-tier, red copy and refusal inside a held slot: `TestRunTierCopies`.
- `HubSetsGate`: `TestHubSetsGate`.
- The started-line round trip: `TestStartedLines`.
- The three gate-key template defaults: `templateDefaults` in `internal/gateslot/config_test.go`.
- `LoadConfig` and `CheckValues`: `TestLoadConfig`.
- Guard and CategoryNone key classification: `wantLocalKeys` in `TestModules_EveryTemplateKeyClassified`, plus the `auto_accept` refusal row in `TestPrepareRunSettings`.
- Every `CheckCampaign` refusal and its order: `TestCheckCampaign`.
- `Rotate`: `TestRotate`.
- Live gate verdicts (red, green, no binary started, copies never fail the gate, `llm_driving` skips copies, refused or errored calls write no report): `TestLiveGate`.
- Gate attempts and `PassOnCap` (`clean` 3, `verify` the campaign's attempts, `live` the live attempts): `TestHardenFixEntry`.
- `harden-burler-fix` verify key and tree scope: `TestHardenFixEntry_VerifyAsksTheHardenKey`.
- Reap behaviour (recorded versus unrecorded processes, ancestors spared, gone-hub server stopped, hub-list pruning, interrupted mark, no process lister): `TestReap`.
- Round row order, models file, commits file, error joining and archive: `TestWrapRow` and `TestArchiveRound`.
- Facts rendering, live-red read and halt: `TestCampaignRoundFacts`, `TestCampaignLiveRed` and `TestCampaignHalt`.
- Sandbox build, list and deploy-when-stale: `TestNewSandbox`.
- The sandbox verbs end to end: `TestSandboxVerbs`.
- Bouncer `OnHalt`: `TestBouncer_HaltRunsTheHaltSeam`.
- Bouncer `LiveRed` on both accept paths: the `liveRed` rows in `TestBouncer_AutoAccept` and `TestBouncer_SoftCapRule`.
- The facts section: `TestWriteRoundFacts_HardenSection`.
- `ReviewRound` and `FixedRound`: `TestReviewRound`.
- The `harden` commit seam: `TestBouncerEntry_CommitSeam`.
- The `harden_round` key on the review row, and `HardenFix` refusals: `TestHardenRoundKey` and `TestHardenFixEntry`.
- The segmented-build rule (two build rows, no Bouncer, unreachable build row, Bouncer before the record row): `ringTests` in `TestBuild_LoomContract`.
- The recipe shape and its loom contract: `TestHardenerRecipe`.
- Reviewer rotation for rounds 1 to 9, hardener tree scope and attempts surviving a `darn.yaml` edit, `landing.Reap` set: `TestWire_HardenerRecipe`.
- `--wait` live-call arguments and result reading, and the Harden-Review budget: `TestHardenLiveArgs` and `TestReviewBudget_HardenReview`.
- Hard cap: `TestPerchHardCap`.
- Finalize reap and its refusal: `TestFinalize_MergeMarkPushOrderAndVerdict` and `TestFinalize_StuckReasons`.
- Teardown clear, ordering and kept worktree: `TestTeardown_CallSequence`.
- Preflight hardener check: `TestPreflight_HardenCheck`.
- `StopServer`: `TestStopServer`.
- `SeedLocalBares`: `TestSeedLocalBares`.
- Retired-role reinterpretation and way forward: `TestDropRetiredRoles` and `TestLoad_RefusesUnusableContent`.
- Recipe lists (batten, report runs and timing, shedcli, shedrun, loomrecipe): `TestChildRecipeSupported`, `TestCompute_SelectsRuns`, `TestComputeTiming`, `TestRecipes_KeySet` and `TestRecipeNames`.

**Findings (behaviour with no pinning test)**

1. MEDIUM, design. `internal/shedrecipe/entries_bouncer.go:361-365` (the `harden_round` seam wiring). Nothing asserts that `bouncerCfg.LiveRed` and `bouncerCfg.OnHalt` are filled from `env.Harden`. `BouncerRowCarriesTheCampaignsFactsInItsFactsFile` covers `RoundFacts` only. Deleting either line passes every test, which silently removes the "never auto-accept a red live gate" bound and the halt reap. Fix: add a `TestHardenRoundKey` row that builds a Bouncer with `harden_round`, drives a spent-budget auto-accept over a red live report and expects an escalation, and drives a halt and expects `Reap` to be called at `ReapHalt`.

2. MEDIUM, design. `internal/hardenshed/round.go:103-109` and `internal/hardenshed/round_test.go` (`FixRowWritesTheCommitsReapsAndArchives`). The D7 order is end-of-row reap first, then archive, and the acceptance criterion says a stray left by the row appears in the archived stray report. The test only checks that `harden-strays-1.json` exists in the archive. It does not read its entries. Moving the archive before the end reap would pass. Fix: read the archived stray report in that row and assert it holds the `ReapEnd` entry.

3. LOW, design. `internal/gatecli/run.go:156` (`exe.green` skipped when `copies > 0`). No test shows a copies call writes or carries no green record. Fix: add an assertion in `TestRunTierCopies` that `exe.green` is untouched.

4. LOW, design. `internal/gatecli/detachfront.go:376` (`words()` forwarding `--copies`). `TestTestRequestWords_MainPackages` does not assert it. A detached re-run would drop the flag. Fix: add a `copies` row to the words test.

5. LOW, design. `internal/gatecli/copies.go:139-155`. The prebuilt-`lym` path (built once for all copies, `PrebuiltLym` cleared) and `mergeCopies`' totals have no row. `TestRunTierCopies` uses a plain `exit` command. Fix: add a copies row with `PrebuiltLym` and one asserting merged `Launches` and `Passed` counts.

6. LOW, design. `internal/loomcli/hardenwiring.go:570-588` (`hardenLiveKey`, where `Tested` adds each tier's `Covers`) and `hardenPreflight`. Only `CheckCampaign` is tested, with fake `Tested` values. The covers expansion and the absent-gate-config path are unpinned. Fix: a `loomcli` test over a gate config with `covers`.

7. LOW, design. `internal/loomcli/hardenwiring.go:326-366` (`hardenSegmentSettings`). Fixer rotation for rounds 4 and later, the `judge` and `seed` overrides, and the unresolvable-model refusal naming the `hardener.yaml` key have no row. `TestWire_HardenerRecipe` checks reviewer rotation only. Fix: extend it to the fixer list, and add rows for judge and seed and for a bad spec.

8. LOW, design. `internal/hardenshed/config.go` (empty `reviewer` or `fixer` list). `LoadConfig` and `CheckCampaign` accept an empty list, and `Rotate` then returns `[]` with no test of the outcome. Fix: either refuse it with a row in `TestCheckCampaign` and `TestLoadConfig`, or pin the intended behaviour.

9. LOW, design. `internal/hardencli/cli.go:269-282` (`commitSandboxConfig`). `TestSandboxVerbs` asserts `friction: ""` only. The record and acceptance criterion also name `selfreport: false`, and `loom.yaml` has no such key (the handler already knows this). The committed override's coverage of that key is unpinned. Fix: settle which key is meant, then assert it.

10. LOW, design. `internal/pairteardown/teardown.go:506-533` (`clearWorktreeSandbox`, the real spawn). The root-exists check and the spawn through `selfexec.LymBinary` are untested. The fake replaces them in `TestTeardown_CallSequence`. Fix: add a test with a stub `lym` and a root directory.

11. LOW, design. `internal/loomcli/hardenwiring.go:430-465` (`HardenReapDeps`) and `internal/hardenshed/reap.go:224-249` (`namesRoot`). No test reaps a real orphan process through `/proc`. `TestSandboxVerbs` runs `clear` with no processes. Fix: one tier-1 test that starts a child with its cwd under the root and expects `clear` to end it.

12. LOW, design. The hardener recipe's `reviewer_fan_setting`, `fixer_fan_setting` and `auto_accept_setting` values (`hardener.*`). No test shows each resolves to `hardener.yaml` and not `loom.yaml`. This is the D9 and D6 guard path, and I found no test in the diff. Fix: a `configreg` or loomcli test resolving `hardener.auto_accept` from a hub `hardener.yaml`.

**Tool calls:** none. The manifest in my inherited context covered everything.
