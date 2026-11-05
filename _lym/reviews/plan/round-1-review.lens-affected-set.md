```yaml
lens: affected-set
outcome: reported
```

Lens: affected-set. For each card's change I checked what reads or asserts it outside the card, and which cards cover that.

**Per change: what reads it, and which cards cover it**

- **Card 4 renames `BaselineModel` / `VerifyModel` / `VerifyInteractive` in `Config`.**
  - Readers: `config.go`, `seats.go`, `config_test.go`, `seats_test.go` (all card 4).
  - Also reads it: `verify_test.go:175` (in the `verifyFixture` table closure) and `verify_test.go:869` (in `TestDriftVerify`). **No card before 16 covers these.**
  - The key names `verify_interactive` / `verify_model` / `baseline_model`: `configreg_test.go` (card 4), `template.yaml` and `doc.go` (card 4), `PATTERN-parent-directive.md` and `parentdirective_test.go` (card 24), `drift-template-verify.md` (card 24), and `driftcli/cli.go:135,221`, help text of verbs card 22 removes. Covered.
- **`verify_max_runs` / `VerifyMaxRuns`** (card 26): `configcli_test` (26), `configreg_test` (26), the `driftcli` integration fixtures `driftYAML` and the anchor test (26), `accept_integration_test` (deleted, 22), `seats.go:275` in `VerifyTable` (deleted, 24). Covered.
- **Card 16 changes `DriftVerify`'s config to exactly one `judgement` gate.**
  - Readers: `driftVerifyEntry`, `TestDriftEntries` (`entries_drift_test.go:53`) and `engineMinimalConfig` (`build_engines_test.go:57`), all card 16.
  - Also reads it: `minimalEntries` in `internal/shedrecipe/conformance_test.go:91-96`, which builds DriftVerify with `gates: [{name: verify}]`. **Card 16 does not list it**; only card 23 edits `minimalEntries`, and only to drop the `DriftRecheck` row.
- **`Defer` re-sign** (card 19): callers `driftcli/cli.go:543` `deferCmd`, `verbs_test.go` `TestDeferWaive`, `TestDriftAwait`, `TestDriftBaseline`. All card 19. Covered.
- **`Verdict` re-sign** (card 20): `loomcli/wiring.go:809` (`wire`), `loomcli/landingdeps.go:244` (`landingDeps`), `verdict_test.go`. All card 20. Covered.
- **`statusAt` drops a parameter** (card 25): callers are `TestDriftAwait`, `newBaselineFixture`, `TestDriftBaseline`, `TestDeferWaive`, `TestDriftVerify` (card 25); `TestAcceptWithdraw` and `TestRecordScope` (deleted, 25); `verifyFixture.agentAccept` / `sessionAccept` (deleted, 16). Covered.
- **`Paths` field removals** (cards 24–26):
  - `AcceptLock` in `driftcli/cli.go:154-162`: helper removed by card 22.
  - `AcceptedTestsDir` / `AcceptLock` in `accept_integration_test.go`: deleted, 22.
  - `Runs` in `capture_integration_test.go:252-284`, `TestCapture_OutlivesCaller`: edited by card 22, which drops its compare steps.
  - `AcceptedDir` in `driftwiring_integration_test.go:203`: test deleted, 24.
  - Covered.
- **`drift-verify` gate key** (cards 16 and 27):
  - Card 27: `gateslot/template.yaml`, `goReadGates` (`tiers.go:113`), `templateDefaults` (`config_test.go:80`), `TestSeedAdoptGate` (`config_test.go:821`), envkit's fixed gate lists (`envkit.go:284`), `gatecli/doc.go:332`, `docs/dotnet-hub.md:96`.
  - Card 16: `entries_drift_test.go:53`, `build_engines_test.go:57`.
  - Card 24: the `drift-verify` role row in `stencils.go:785` (`roleOpeningStencils`).
  - Covered.
- **`DriftRecheck` engine** (cards 16 and 23):
  - Card 23: `registry`, `TestNames` (`registry_test.go:102`), `minimalEntries` (`conformance_test.go:90`), `TestDriftEntries`, both `doc.go` files.
  - Card 16: the `loomrecipe` tests and the recipes.
  - `reportengine` and `loomshed` read only the row-name constant `NameDriftRecheck`, which stays.
  - Covered.
- **Retired verbs' names in text** (D14: stencil and recipe-comment text):
  - Covered: `await.go:127-129` (26), `baseline.go:101` (18), `steprun.go:249-251` (25, through `missingDirRefusal`), `verify.go` (16), `seats.go` (24), `darn-recipe.yaml:54` (18).
  - Not covered: card 18 itself **adds** recipe-header text naming `lym drift accept`, `withdraw` and `waive`.

**Findings**

1. **BLOCKING**, class design, cards 04 and 16 (method).
   Both cards miss test members their own changes force, so each card's tier-0 gate fails from the state the cards before it leave.
   - Card 4 renames `Config.VerifyInteractive` but does not list the `driftengine` members that read it: `verifyFixture`'s table closure (`internal/driftengine/verify_test.go:175`) and `TestDriftVerify` (`verify_test.go:869`). The driftengine test package stops compiling until card 16 rewrites those members.
   - Card 16 makes `driftVerifyEntry` accept only one `judgement` gate but does not list `internal/shedrecipe#minimalEntries`, whose DriftVerify row is `{"subject": "decision-record", "gates": [{name: verify, attempts: 3}]}` (`internal/shedrecipe/conformance_test.go:91-96`). `TestEveryEntry_RejectsUnknownConfigKey` builds from that row.
   - Fix: add `internal/driftengine#verifyFixture` and `internal/driftengine#TestDriftVerify` to card 4's `Edit` list, renaming the field. Add `internal/shedrecipe#minimalEntries` to card 16's `Edit` list, its DriftVerify row becoming `gates: [{name: judgement, attempts: 3}]`.

2. **MEDIUM**, class decision, card 18.
   Card 18's Intent has both recipe headers add a note that a run parked at an old halt whose reason names `lym drift accept`, `withdraw` or `waive` goes on with `lym loom resume` (`_lym/plan/18-drift-baseline-await.md:46`).
   That is new recipe-comment text naming verbs card 22 removes. D14 removes "their stencil and recipe-comment text", and the Review criterion says "no recipe comment names a removed verb" (`decision-record.md:333`, `:449`).
   Fix: word the note without the removed verb names, for example "an old Drift-Verify halt from before this change".

Tool calls:
1. Grep for the renamed and removed config keys and fields across the repo: the pre-read had no readers of the `Config` fields.
2. Grep for `VerifyInteractive` lines, `Defer`/`Verdict` callers and the files calling `statusAt`: needed the call sites behind the re-signs and the rename.
3. Awk to map `statusAt`/`Defer` calls to their enclosing functions, plus a grep for `Paths` field readers outside driftengine: needed member-level coverage.
4. Enclosing tests for the `Runs`/`AcceptedDir` readers, plus readers of the `drift-verify` key and `DriftRecheck`: those were outside the pre-read.
5. Lines and enclosing tests for the `DriftRecheck` rows in conformance, registry, reportengine and loomshed: needed to tell an engine reference from a row-name reference.
6. `minimalEntries`' DriftVerify row in `conformance_test.go`: needed the config shape card 16 breaks.
