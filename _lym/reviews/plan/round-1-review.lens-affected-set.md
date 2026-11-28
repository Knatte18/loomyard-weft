```yaml
lens: affected-set
outcome: reported
```

Task: affected-set lens. For each change a card makes, find everything that reads or asserts it and check which cards cover that set.

**Per change: the set found, and the cards that cover it**

- **New nested Deps structs** (`Deps.Backstop landingqueue.BackstopSeams` in card 7, `Deps.Detach DetachDeps` in card 8).
  - Set: every test carrying `//lym:seam internal/landingshed#Deps` (`TestLandingDeps_EveryFieldPopulated`, `TestWire_BeamRecipe`, `TestWire_DarnRecipe`, `TestWire_SamplerRecipe`, loomcli's and battencli's `TestWire_EverySeamFilled`, `TestFullEnv_FillsEverySeam`, `TestNilSeams_ReportsExactPaths`, `TestNilSeams_SkipsNilLegalPaths`), plus `envkit.LandingDeps` and battencli's `intentionallyNil`.
  - Covered by cards 7 and 8, except the two `TestNilSeams_*` tests: both cards list them only under Uses. **Partial.**
- **New Deps func fields** (`Queue`, `RecordsHead`, `NewestLanding`, `MergesClean`, `MemberDeps`, `WriteLedgerLanding`, `RecordsToHold`, `BuildChain`, `FirstParent`, `DetachState`).
  - Set: `landingDeps`, `envkit.LandingDeps`, `intentionallyNil`, and the reflective tests that walk them unchanged. Covered by cards 7, 8 and 10.
- **`parentMerger` gains `MoveCodeTo`.**
  - Set: `*fabricengine.Fabric` and `recordingParentMerger` (finalize_test.go:100-104), the only fake. Covered by cards 3 and 8.
- **`recordLanding`, `landedDiffEmpty` and `recordLandedSize` re-signed.**
  - Set: finalize.go:572, :974 and :978 only; no test calls them directly. Covered by card 7.
- **`TaskHead` switched to resolve `refs/heads/<taskBranch>`.**
  - Set: the Finalize readers (`rebaseAfterParentMerge`, `runFixer`, `judgeFixerChange`, `endSpent`, `mergeParent`, the failure recorder's head) and Parent-Align, which shares the same `landingDeps`. These read the same value while the worktree is on its branch. Covered by card 8; no change is forced.
- **Detach marker honoured by `readLocalBranch`, `RefusedPushStands`, `pushClassified` and `parentHoldsTaskWorkAt`.**
  - Set: `RunCommitsAhead` (runpusher `targetAhead`, tendercli deps.go:143); `TipsOnOrigin` (beamcli dobbydeps.go:807, battencli wire.go:1837); `TargetRefused`; `unpushedUnder`; `PushPairAnchored`, `pushSideDetached` and `landing.go`'s prime push; the `ParentHoldsTaskWork` and `OriginParentHoldsTaskWork` callers (prepush.go:78, pairteardown runread.go:160/167, battencli cancelcmd.go:490/492 and wire.go:1330, runpusher parentseams.go:159). Covered by card 4 through the shared functions.
- **`gateUnlocked`, `gatedTip` and `parentMoved` removed.**
  - Set: finalize.go only. The `parentMoved` at finalize_test.go:915 is an unrelated local. Covered by card 5.
- **`landing.finalize_lost_races` and `FinalizeLostRaces` removed.**
  - Set:
    - config.go:79, :82, :184-185, :207 and :261-262;
    - keynames.go:24;
    - template.yaml:34;
    - config_test.go:231;
    - finalize_owner_test.go:201;
    - finalize_unlocked_test.go:88, :174, :223, :400 and :437;
    - landlocally_integration_test.go:64;
    - the four recipe yaml comments;
    - docs/overview.md:1402 and :1406;
    - doc.go:45;
    - finalize.go.
  - Covered by cards 5, 6 and 12.
- **`RetiredKeys` gains `finalize_lost_races`.**
  - Set: landing's `ConfigSpec` and configengine's layer loader. configreg's run_settings refusal reads `RetiredWayForward`, which no card touches and which is consistent with D9. Covered by card 6.
- **`Shed.Restore` and `ErrRestoreRefused` added** (card 10).
  - Set: `Run` and `Step`, loomcli `newShed`, and the shedverbs drivers that classify the locked-check refusal (shedverbs step.go:410, spec.go:211). Card 10 covers the first three; the shedverbs classification is not covered. **Partial.**
- **New lymdirs names** (`LandingQueueDirName`, `DetachMarkerName`).
  - Set: internal/lymdirs has no test file enumerating names. Covered by cards 1 and 4.
- **Glossary line** (card 12).
  - Set: heddleengine shipped.go reads the file; no test asserts its lines. Covered by card 12.

**Findings**

1. **BLOCKING · consistency · cards 7 and 8** (`internal/testkit/envkit#TestNilSeams_ReportsExactPaths`, `#TestNilSeams_SkipsNilLegalPaths`)
   - Summary: both tests reach `envkit.NilSeams` and carry `//lym:seam internal/landingshed#Deps`. The new nested structs `landingqueue#BackstopSeams` (card 7) and `landingshed#DetachDeps` (card 8) must be marked on them too. The cards list both tests under Uses, and their Intent names only `TestFullEnv_FillsEverySeam` among the envkit tests, so the tier0 scan refuses the two unmarked tests.
   - The method fault: the card's search, `grep -rln`, lists files rather than test functions, so it missed the two other marked tests in envkit_test.go.
   - Evidence: envkit_test.go:115-131 and :167-193 (the markers, and `NilSeams(env)` called in both); pattern/PATTERN-test-speed.md:174-181 ("a test reaching `envkit.NilSeams` whose markers miss a struct the `Env` set holds" is refused); loom-plan-checks-spec.md:271 ("It passes a test listed under `Uses` that the card breaks"), so the gate does not catch this.
   - Fix: move both tests to Edit on cards 7 and 8, each gaining the same marker those cards add.
2. **MEDIUM · consistency · card 10** (`internal/shedengine#ErrRestoreRefused`, `Shed.Run`, `Shed.Step`)
   - Summary: card 10 wraps a restore refusal in a new sentinel, `ErrRestoreRefused`, beside `ErrLockedCheckRefused`. The step driver maps only `ErrLockedCheckRefused` onto its refusal kind, so a refused resume restore reaches the loop as an unclassified error rather than a drive refusal.
   - Neither shedverbs step.go nor spec.go is a target, so the affected set is partly covered.
   - Evidence: internal/shedverbs/step.go:410 (`errors.Is(err, shedengine.ErrLockedCheckRefused)`); internal/shedverbs/spec.go:211 (`StepBusyKind` maps `ErrShedBusy` and `ErrLockedCheckRefused`); shedengine/errors.go:15.
   - Fix: wrap the restore refusal in `ErrLockedCheckRefused` as well, or add shedverbs step.go's classification to card 10's targets.

Tool calls:
1. Grepped for `parentMerger` implementers and direct callers of the re-signed `recordLanding` helpers, plus leftover `gateUnlocked`, `gatedTip` and `parentMoved` references: the manifest had the definitions but not every fake or caller.
2. Searched the repository for "lock-free" and "lost race" wording outside finalize.go, and read shedengine/doc.go:20-35: the manifest held only the `finalize_lost_races` key references.
3. Listed internal/lymdirs, and searched for glossary readers and `RetiredKeys` readers: the readers of the new names, the glossary line and the retired key were not in context.
4. Read landing's `ConfigSpec` `RetiredKeys` block and configreg's run_settings retired-key refusal: to check whether a retired key also needs a `RetiredWayForward` entry.
5. Grepped for callers of `ErrLockedCheckRefused` and `ErrShedBusy` outside shedengine: to find who classifies the refusal card 10 adds beside it.
