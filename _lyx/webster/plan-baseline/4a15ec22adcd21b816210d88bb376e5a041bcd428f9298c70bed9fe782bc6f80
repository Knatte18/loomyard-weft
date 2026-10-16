# Card 5 — Retire the below-parent and top anchors

**Delete:**
- `internal/reedengine/render#AnchorTop`
- `internal/reedengine/render#AnchorBelowParent`
- `internal/reedengine#Engine.splitWindowZeroPaneLocked`
- `internal/reedengine#planPaneTarget`
- `internal/reedengine#paneSplit`
- `internal/reedengine#planTopPaneTarget`
- `internal/reedengine#topContentDisplay`
- `internal/reedengine#TestPlanPaneTarget`

**Create:**
- `plan:internal/reedengine/render#Anchor.UnmarshalJSON` -> `func (a *Anchor) UnmarshalJSON(data []byte) error`
- `plan:internal/reedcli#TestAddCmd_RefusesAnAnchorOutsideTheVocabulary` -> `func TestAddCmd_RefusesAnAnchorOutsideTheVocabulary(t *testing.T)`
- `plan:internal/reedengine#TestWriteOps_RefuseARetiredAnchor` -> `func TestWriteOps_RefuseARetiredAnchor(t *testing.T)`

**Edit:**
- `internal/reedengine/render#Anchor`
- `internal/reedengine/render#ValidateDisplay`
- `internal/reedengine/render#TestDisplayDecodesRetiredKeysIgnored`
- `internal/reedengine#validateDisplay`
- `internal/reedengine#refuseWindowMove`
- `internal/reedengine#Engine.launchStrandLocked`
- `internal/reedengine#recordedPaneIDs`
- `internal/reedengine#shellWindowOf`
- `internal/reedcli#reedCLI.addCmd`
- `internal/shuttleengine#Spec`
- `internal/shuttleengine#normalizeAttachSpec`
- `internal/reedengine#retiredKeysState`
- `internal/reedengine#currentKeysState`
- `internal/reedengine#TestLoadState_RetiredDisplayKeysIgnoredAndLayoutUnchanged`
- `internal/reedengine#TestSaveState_ThenLoadState_RoundTrips`
- `internal/reedengine#TestAddStrandLocked`
- `internal/reedengine#TestUpdateStrandLocked`
- `internal/reedengine#TestAddStrand_IfAbsent_NoOps`
- `internal/reedengine#TestAddStrandUnless_NamedStrandSkips`
- `internal/reedengine#TestAddStrandUnless_NoNamedStrandAdds`
- `internal/reedengine#TestClassifyIfAbsent`
- `internal/reedengine#TestNeedsLaunchOnAdd`
- `internal/reedengine#TestNeedsLaunchOnSurface`
- `internal/reedengine#TestRemovalEmptiedSession`
- `internal/reedengine#TestRemoveStrand_NeverKillsAPaneOutsideThisSession`
- `internal/reedengine#TestCapturePaneArgv`
- `internal/reedengine#TestPasteSubmit`
- `internal/reedengine#TestResolveLivePaneID`
- `internal/reedengine#TestSetWaitMark`
- `internal/reedengine#TestPlanResumeLaunches_ThreeLifecycleStates`
- `internal/reedengine#TestPlanUpLaunches_NeverLaunchesAnyStrand`
- `internal/reedengine#repairStrand`
- `internal/reedengine#TestStrandWindowTarget`
- `internal/reedengine/render#TestDisplayMarshalsWithoutRetiredKeys`
- `internal/reedengine#goodAttachStrands`
- `internal/reedengine#TestAttachArgv_EveryOtherDegradedPathYieldsBareArgv`
- `internal/reedengine#setupAttachGeometryFixture`
- `internal/reedengine#clearConflictingPaneBindings`
- `internal/reedengine#Engine.AddStrandUnless`
- `internal/reedengine#Engine.ReplaceStrand`
- `internal/reedcli#TestStrandNamingAndList`
- `internal/reedengine#launchFake`
- `internal/reedengine#TestLaunchStrandLocked_ReapsUntrackedPanesBeforeChoosingASplitTarget`
- `internal/reedengine#TestLaunchStrandLocked_SkipsTheRedundantReEnumerationWhenNothingIsReaped`
- `internal/reedengine#TestLaunchStrandLocked_SplitWindowAndPaneTitleCalls`
- `internal/reedengine#TestOperatorActive_ReadsAnAttachedClientsKeystroke`
- `internal/reedengine#newBarFixture`
- `internal/reedengine#newBindingsFixture`
- `internal/reedengine#TestMultiplexerContract`
- `internal/reedengine#TestRemoveStrand_SoleStrandEmptiesSessionSucceeds`
- `internal/reedengine#TestDeadSelvagePaneIsHealedByUpWithoutCorruptingLayout`
- `internal/reedengine#TestStrandOpsIgnoreASecondCurrentWindow`
- `internal/reedengine#TestAddStrand_LogsAttributionOnlyOnColdBoot`
- `internal/reedengine#TestLaunchScript_SourcedScriptRunsInThePaneShellScope`
- `internal/reedengine#addEnvProbeStrand`
- `internal/reedengine#TestOwnWindow_RealTmux`
- `internal/reedengine#TestPasteSubmit_DeliversTextAndLeavesTheBufferStackAlone`
- `internal/reedengine#TestStrandOps_RealTmux`
- `internal/reedengine#TestWindowOrder_RealTmux`
- `internal/shuttleengine#TestSpec_Validate`
- `internal/shuttleengine#TestRun_Wait_Classification`
- `internal/shuttleengine#TestRunner_Start_HappyPath`
- `internal/shuttleengine#TestAttach_RunningRecordUnsatisfiedFileContract_RespawnsOrErrors`
- `internal/shuttleengine#TestAttach_UntrackedTerminalRecord_RespawnEligibleRegardlessOfAge`
- `internal/pairteardown_test#addLivePair`
- `internal/standalonegeom_test#TestNaming_StandaloneStrandIsShortnameAndRoleWithNoParent`
- `internal/reedcli#TestWatchdogReap`
- `internal/reedcli#listPaneLines`
- `internal/reedcli#TestSmokeLifecycle`
- `internal/reedcli#TestSmokeRemoveLastStrandThenAddRunsTheNewCommand`
- `internal/reedcli#TestSmokeCrashRecovery`
- `internal/reedcli#TestSmokeStateRecovery`
- `internal/reedcli#TestSmokeSelvage`

**Intent:** D1, D2's `lyx reed add` half and D3's write refusal and read-time remap: the anchor vocabulary becomes `own-window`, `overview` and `hidden`, so no strand reed places lands in the shell window.

render: `AnchorTop` and `AnchorBelowParent` go and `Anchor`'s doc names the three-value closed set.
`Anchor.UnmarshalJSON` decodes a JSON string and maps the retired spellings `"below-parent"` and `"top"` to `AnchorOwnWindow`, keeping every other value as written, so a `reed.json` strand an older binary recorded loads and reads as `own-window` (PATTERN-tolerant-records); a non-string value is a decode error as before.
Because the remap happens at decode, `internal/state`'s rewrite sees the old element decode equal to the new one and keeps its unknown `size` and `contentRows` keys, and the rewritten record reads `anchor: own-window`.
`ValidateDisplay`'s doc drops the retired anchors.

reedengine: `validateDisplay` accepts `hidden`, `own-window` and `overview`; a display naming `below-parent` or `top` is refused with `anchor %q is retired; way forward: use own-window`, and any other value with `invalid anchor %q; want own-window|hidden|overview`.
`addStrandLocked` and `updateStrandLocked` run it, and `AddStrand` reaches it through `addStrandLocked` before any pane is launched.
`AddStrandUnless` also runs it at its head, beside `validateIfAbsent` and before the session pre-flight, so a refused add boots no server; `ReplaceStrand` runs it at its head, beside `validateNaming`, because its body removes the old strand and kills its pane before it reaches `addStrandLocked`, and a refusal there would leave the old strand gone.
`UpdateStrand` reaches it through `updateStrandLocked` before any launch or save.
So each of the four writes refuses before any pane or state changes.
The input reaching the refusal is a Go-built `render.Display{Anchor: render.Anchor("below-parent")}`, which no decode remaps; `TestWriteOps_RefuseARetiredAnchor` reaches it.
`clearConflictingPaneBindings`'s doc stops citing `planPaneTarget` and `planLayout`.
`refuseWindowMove`'s doc stops naming window 0.
`launchStrandLocked` creates every pane through `createOwnWindowPaneLocked`, since every non-hidden display is own-window or overview; `splitWindowZeroPaneLocked`, `planPaneTarget`, `paneSplit` and `planTopPaneTarget` go.
`recordedPaneIDs` lists only the recorded Selvage pane, and its and `shellWindowOf`'s docs stop naming top or below-parent strands.

`lyx reed add`: `--anchor` defaults to `own-window` and its help reads `placement: own-window|hidden`; any other value, a retired one and `overview` included, exits non-zero with `invalid --anchor %q; want own-window|hidden` before the engine is called, so nothing is added.
The `Long` text says a strand gets a window of its own unless `--anchor hidden`, not a live pane, and the CLI-vocabulary comment drops the top-anchor note.

shuttleengine: `Spec`'s `Display` comment and `normalizeAttachSpec`'s comment stop naming `render.AnchorBelowParent` and window 0.

Tests: `TestAddCmd_RefusesAnAnchorOutsideTheVocabulary` drives `(&reedCLI{}).addCmd()` directly, as `TestRunCLI_Run_FlagValidation` drives shuttle's run, with rows `below-parent`, `top`, `overview`, `sideways` and an empty `--anchor=`, each with `--cmd`; it asserts one error envelope whose message names the value and `own-window`, and the nil engine proves nothing was added.
At the tmux tier `TestStrandNamingAndList` replaces its `--anchor sideways` check with `--anchor below-parent`, keeping the non-zero exit assertion the unit test cannot make, and gains steps that pin the new default: `add --cmd` with no `--anchor` and no `--parent` opens the strand in a window carrying `@lyx_own`, and the shell window still lists only the Selvage pane; `add --parent <that guid> --cmd` with no `--anchor` opens the child in a window of its own; and `remove --recursive <parent guid>` removes both.
`TestWriteOps_RefuseARetiredAnchor` runs on the fake tmux with a persisted strand: rows call `AddStrand`, `AddStrandUnless`, `UpdateStrand` on that strand and `ReplaceStrand` of it, each with `below-parent` and with `top`, and assert an error naming `own-window`, an unchanged `reed.json`, and no `split-window`, `new-window` or `kill-pane` issued (`mustNotCall`); the `ReplaceStrand` rows fail if the check stays only in `addStrandLocked`.
`TestDisplayDecodesRetiredKeysIgnored` gains rows decoding `"below-parent"` and `"top"` as `own-window`, `"hidden"` unchanged, and a non-string anchor (`1`) as a decode error.
`TestStrandWindowTarget` drops its `StrandPaneResolvesWithoutSelvage` row, which pinned the strand-pane fallback `recordedPaneIDs` no longer feeds.
`TestDisplayMarshalsWithoutRetiredKeys`, `goodAttachStrands`, `TestAttachArgv_EveryOtherDegradedPathYieldsBareArgv` and `setupAttachGeometryFixture` move their displays to `own-window`; `TestDisplayMarshalsWithoutRetiredKeys` round-trips an `own-window` display with `KeepViewer` set, since a `top` value now decodes as `own-window` and would no longer round-trip equal.
`retiredKeysState` holds strands with `anchor: below-parent` and `anchor: top`, a `size` object and `contentRows`; `currentKeysState` holds the same strands as `own-window`, and `TestLoadState_RetiredDisplayKeysIgnoredAndLayoutUnchanged` asserts the retired file loads to the current displays, and that a `SaveState` of the loaded state writes `anchor: own-window` and keeps the `size` and `contentRows` keys in the file.
Every other test listed used `below-parent` or `top` only as a convenient anchor, or relied on `lyx reed add`'s old default, and moves to `own-window` (D8); `TestPlanPaneTarget` and `topContentDisplay` pin only the retired split planning and go.
The launch tests assert a `new-window` where they asserted a `split-window`; the reedcli smokes that read the session's current window to find a strand pane read the strand's own window, `listPaneLines` listing session-wide.
The hidden-strand rows in `TestNeedsLaunchOnAdd` and `TestNeedsLaunchOnSurface` keep pinning that a hidden strand owns no pane.
`TestMultiplexerContract` builds no display; its step (b) comment stops describing the raw `split-window` as the path `launchStrandLocked` uses, since `launchStrandLocked` now creates every pane through `new-window`.
`TestSmokeLifecycle`'s `StackedAddsKeepEverySessionPane` subtest is renamed `AddsKeepEverySessionPane`, and its comment is rewritten to say that successive default adds each open a window of their own and yield one live pane per strand plus Selvage in the session-wide listing.

**ImpactSummary:** removes two exported render constants whose only remaining users are the tests and comments listed here, across `internal/reedengine`, `internal/reedcli`, `internal/shuttleengine`, `internal/pairteardown` and `internal/standalonegeom`; no production spawner named either.

**Commit:** 5: reed: retire the below-parent and top anchors
