# Card 2 — Retire the shell window's stacked layout and its resize hook

**Delete:**
- `internal/reedengine/render/checksum.go`
- `internal/reedengine/render/checksum_test.go`
- `internal/reedengine/render/focus.go`
- `internal/reedengine/render/height.go`
- `internal/reedengine/render/height_test.go`
- `internal/reedengine/render/layout.go`
- `internal/reedengine/render/rules.go`
- `internal/reedengine/render/rules_test.go`
- `internal/reedengine/render/pins_test.go`
- `internal/reedengine/render#SizeKind`
- `internal/reedengine/render#SizePositional`
- `internal/reedengine/render#SizeContent`
- `internal/reedengine/render#SizeSharing`
- `internal/reedengine/render#Size`
- `internal/reedengine/render#Strand`
- `internal/reedengine/render#Box`
- `internal/reedengine/render#Selvage`
- `internal/reedengine/render#Params`
- `internal/reedengine/render#partitionByAnchor`
- `internal/reedengine/render#removeDuplicatePaneCells`
- `internal/reedengine/render#breakCycles`
- `internal/reedengine/render#severParent`
- `internal/reedengine/render#orderStack`
- `internal/reedengine/render/policy_test.go`
- `internal/reedengine/apply_test.go`
- `internal/reedengine/zoom_integration_test.go`
- `internal/reedengine#paneIDsByTop`
- `internal/reedengine#renderInputs`
- `internal/reedengine#Engine.toRenderInputs`
- `internal/reedengine#Engine.planLayout`
- `internal/reedengine#Engine.fixedHeightPins`
- `internal/reedengine#anyPlacedStrand`
- `internal/reedengine#applyOpts`
- `internal/reedengine#applyResult`
- `internal/reedengine#Engine.applyLayoutLockedOpts`
- `internal/reedengine#Engine.applyLayoutLocked`
- `internal/reedengine#toRenderStrands`
- `internal/reedengine#TestToRenderStrands_MapsFieldsAndSetsLiveFromPaneSet`
- `internal/reedengine#Engine.SetContentRows`
- `internal/reedengine#TestSetContentRows`
- `internal/reedengine#Engine.selvageRenderParams`
- `internal/reedengine#TestSelvageRenderParams`
- `internal/reedengine#parseWindowSize`
- `internal/reedengine#Engine.liveBoxLocked`
- `internal/reedengine#reservedRowsFromStatus`
- `internal/reedengine#windowSizeAllowsChain`
- `internal/reedengine#Engine.readStatusRowsLocked`
- `internal/reedengine#Engine.readWindowSizeLatestLocked`
- `internal/reedengine#Engine.readBorderTitleRowLocked`
- `internal/reedengine#pinsForContent`
- `internal/reedengine#Engine.contentPinsLocked`
- `internal/reedengine#Engine.runResizePinsLocked`
- `internal/reedengine#resizePinHookArgvs`
- `internal/reedengine#Engine.resizeSignalHookCommand`
- `internal/reedengine#Engine.installResizePinsLocked`
- `internal/reedengine#Engine.removeResizeSignalFileLocked`
- `internal/reedengine#TestParseWindowSize`
- `internal/reedengine#TestLiveBoxLocked`
- `internal/reedengine#TestReservedRowsFromStatus`
- `internal/reedengine#TestWindowSizeAllowsChain`
- `internal/reedengine#TestReadbacksLocked`
- `internal/reedengine#TestResizePinHookArgvs`
- `internal/reedengine#assertResizePinHookArgvsWellFormed`
- `internal/reedengine#TestResizeSignalHookCommand`
- `internal/reedengine#TestInstallResizePinsLocked_IssuesTheSignalEntryLast`
- `internal/reedengine#TestPinsForContent`
- `internal/reedengine#resizeSignalFileName`
- `internal/reedengine#Engine.resizeSignalPath`
- `internal/reedengine#tmuxQuoteValue`
- `internal/reedengine#resizeHookCommand`
- `internal/reedengine#TestResizeHookCommand`
- `internal/reedengine#TestTmuxQuoteValue`
- `internal/reedengine#TestResizeSignalPath`
- `internal/reedengine#TestPinBootOptionsLocked_InstallsTheSignalEntry`
- `internal/reedengine#errAttachChainSuppressed`
- `internal/reedengine#errAttachNothingToLayOut`
- `internal/reedengine#errNoClientSize`
- `internal/reedengine#chainedAttachArgv`
- `internal/reedengine#TestAttachArgv_ChainedArgv`
- `internal/reedengine#TestAttachArgv_ChainGate`
- `internal/reedengine#TestAttachArgv_ChainCarriesTheAdjustedPinsAfterSelectLayout`
- `internal/reedengine#TestAttachArgv_PreflightListsPanesOnTheUnzoomedWindow`
- `internal/reedengine#wantChainedAttachArgv`
- `internal/reedengine#assertChainedArgv`
- `internal/reedengine#wantZoomBracketedChain`
- `internal/reedengine#zoomRecordHookBody`
- `internal/reedengine#zoomRestoreHookBody`
- `internal/reedengine#zoomRecordChainArgv`
- `internal/reedengine#zoomRestoreChainArgv`
- `internal/reedengine#paneBorderFormat`
- `internal/reedengine#windowSizeNow`
- `internal/reedengine#windowLayoutNow`
- `internal/reedengine#plannedLayoutOfChain`
- `internal/reedengine#TestAttachGeometry_ExactLayoutAndRowBudgets`
- `internal/reedengine#TestAttachGeometry_StaleLayoutRaceIsSafe`
- `internal/reedengine#assertAttachGeometryRowBudgets`
- `internal/reedengine#TestAttachGeometry_ResizeAfterAttachHoldsRowBudgets`
- `internal/reedengine#TestAttachGeometry_BareAttachFromTallClientStillHoldsSelvageBudget`
- `internal/reedengine#TestAttachGeometry_DeadStripPinDoesNotBreakSelvagePin`
- `internal/reedengine#applyLayoutNow`
- `internal/reedengine#strandWindowCells`
- `internal/reedengine#assertPlannedCells`
- `internal/reedengine#resizeClientAndWait`
- `internal/reedengine#TestAttachGeometry_TitleRowCellsHoldTheirPlannedHeights`
- `internal/reedengine#TestAttachGeometry_PinsKeepTheirCellHeightWithoutATopTitleRow`
- `internal/reedengine#TestAttachGeometry_OneRowTopCellEndsOneRowTallerRightAfterTheApply`
- `internal/reedengine#TestSelvageNeverGetsZeroHeightLayoutCell`
- `internal/reedengine#TestEnsureSelvagePaneLocked_RecoversWhenTheBottomPaneIsTooSmallToSplit`
- `internal/reedcli/smoke_dotfill_test.go`
- `internal/reedcli/smoke_dotfill_measure_test.go`

**Edit:**
- `internal/reedengine/render#Display`
- `internal/reedengine/render#ValidateDisplay`
- `internal/reedengine/render#TestDisplayDecodesRetiredKeysIgnored`
- `internal/reedengine/render#TestDisplayMarshalsWithoutRetiredKeys`
- `internal/reedengine#Strand`
- `internal/reedengine#Engine.reconcileApplyPersistLocked`
- `internal/reedengine#Engine.pinGeometryOptionsLocked`
- `internal/reedengine#Engine.pinBootOptionsLocked`
- `internal/reedengine#Engine.ensureServerAndSessionLocked`
- `internal/reedengine#Engine.AttachArgv`
- `internal/reedengine#reapPolicy.exemptFromDeadKill`
- `internal/reedengine#Engine.splitSelvagePaneAtBottomLocked`
- `internal/reedengine#Engine.splitPaneBelowLocked`
- `internal/reedengine#TestPinGeometryOptionsLocked`
- `internal/reedengine#TestSaveState_ThenLoadState_RoundTrips`
- `internal/reedengine#TestLoadState_RetiredDisplayKeysIgnoredAndLayoutUnchanged`
- `internal/reedengine#TestCalleeFailuresLogOncePerEpisode`
- `internal/reedengine#goodAttachStrands`
- `internal/reedengine#goodAttachLive`
- `internal/reedengine#TestEnsureSelvagePaneLocked_SplitsOnceBelowTheBottommostPane`
- `internal/reedcli#TestSmokeLifecycle`
- `internal/reedengine#assertBareArgv`
- `internal/reedengine#TestAttachArgv_EveryOtherDegradedPathYieldsBareArgv`
- `internal/reedengine#TestAttachArgv_PreflightOnAKnownGoodSession`
- `internal/reedengine#TestAttachArgv_SelectsTheRelevantWindow`
- `internal/reedengine#TestAttachArgv_MultiClientWarning`
- `internal/reedengine#setupAttachGeometryFixture`
- `internal/reedengine#TestAttachGeometry_DegradedPathStillAttaches`
- `internal/reedengine#TestDeadSelvagePaneIsHealedByUpWithoutCorruptingLayout`
- `internal/reedengine#TestStrandOps_RealTmux`
- `internal/reedengine#TestBar_LineTwoListsSessionsInIDOrderAndTheBorderNamesStrands`
- `internal/reedcli#TestSmokeWarmPath`
- `internal/reedengine#topContentDisplay`
- `internal/reedengine#TestAddStrandLocked`
- `internal/reedengine#newBindingsFixture`
- `internal/reedengine#TestWindowOrder_RealTmux`
- `internal/reedengine#TestUpdateStrandLocked`
- `internal/reedengine#TestMultiplexerContract`

**Create:**
- `internal/reedengine#TestPinBootOptionsLocked_ClearsTheWindowResizedHook`

**Intent:** D4, and D6's hook half: reed stops laying out the shell window, so nothing computes, applies, pins or re-applies a stacked strand layout, and every boot and attach clears a `window-resized` hook an older binary pinned.

render keeps only the closed display vocabulary spawners name: `Anchor` and its constants, `Display` and `ValidateDisplay`.
The `internal/reedengine/render/policy.go` file header is rewritten to describe only the display check `ValidateDisplay` makes, dropping the placed-cell filtering, stack ordering, cycle repair, the below-parent and top wording and the policy-versus-mechanics note.
Every layout file goes (`Rules`, `FixedHeightPins`, `Pin`, the cell planner, the height rules, the band clamp, the focus pick, the layout string and its checksum), and so do `Strand`, `Box`, `Selvage`, `Params`, `Size` and the `SizeKind` constants.
`Display` loses its `Size` field; `ValidateDisplay` keeps the own-window `WindowSize` refusal and returns nil for every other anchor, with no size branch.
`AnchorTop` and `AnchorBelowParent` stay for card 5, which retires them; their doc comments stop describing a stacked region.
`TestDisplayDecodesRetiredKeysIgnored` gains rows decoding a display carrying the old `size` object (content and sharing kinds) with it ignored, and `TestDisplayMarshalsWithoutRetiredKeys` asserts no `size` key is written; `PATTERN-tolerant-records` holds since `size` decodes as an unknown key.

In reedengine, `apply.go` keeps `liveIDSet` and `aliveIDSet` and loses everything else; its header names only those two sets.
`reconcileApplyPersistLocked` becomes list, reconcile, persist: it issues no `select-layout`, `select-pane` or `resize-pane`, and keeps its re-list after a kill because callers read the returned panes.
`Strand.ContentRows` goes, so `contentRows` becomes an unknown key `internal/state` keeps on a rewrite; `SetContentRows` goes with it, since no production caller reports content rows.
`selvageRenderParams` goes, and the doc comments of `exemptFromDeadKill`, `splitSelvagePaneAtBottomLocked` and `splitPaneBelowLocked` drop their `render.Rules` and band-cell reasoning while keeping the bottom split and the sole-corpse ordering.
`splitSelvagePaneAtBottomLocked`'s `select-layout even-vertical` retry goes, since D4 has reed issue no `select-layout` on the shell window; instead `splitPaneBelowLocked` splits with `-f`, a full-width split below the bottom-most pane that takes its rows from the whole window, so a short bottom pane an older binary left (the case the retry existed for) can no longer refuse the split.
A split that still fails returns its error, which the next `up` or `resume` retries.
The `TestEnsureSelvagePaneLocked_SplitsOnceBelowTheBottommostPane` comment drops its `render.Rules` band-cell reason, and the test asserts the `-f` flag on the split.

The window-size readbacks, the title-row readback, the pins and the whole `window-resized` array writer go: `resizePinHookArgvs`, `installResizePinsLocked`, `resizeSignalHookCommand`, the resize signal file and its hook command, and the zoom record and restore bodies only the array and the attach chain used.
`windowResizedHookName` stays.
`pinGeometryOptionsLocked` drops the `pane-border-status`/`pane-border-format` pins (and `paneBorderFormat`) and its watchdog block, and ends, everywhere but Windows where psmux has no `set-hook`, with two unconditional unpins, `set-hook -u -w -t <target> window-resized`, the window-scoped clear the array rebuild always issued first, and `set-hook -u -t <target> window-resized`, the session-scoped form the old watchdog-off path cleared, so an entry an older binary pinned in either scope goes; each failure is logged and ignored like its other pins.
This unpin clears only that one window hook on the shell window, which only reed ever pinned, so it can remove nothing an operator set elsewhere.
`pinBootOptionsLocked` drops its zero-pin install, so boot clears the hook through `pinGeometryOptionsLocked` and installs nothing; `ensureServerAndSessionLocked`'s comment about the install reading the watchdog key goes.

`AttachArgv` keeps its signature and its pre-flight (session check, the mismatched-client warning, the state read, `ensureShellWindowLocked`, the window selection, the strand-window marker, the option re-assert, the Selvage marker and `pinGeometryOptionsLocked`) and always returns the bare attach argv with the selection appended; it chains no `select-layout` and no pins, and needs no window-size or status readback.
A degrade path (no lock, no session, no state) still returns the bare argv, without the selection where it returns before the state read.

Tests: the render layout tests, `render/policy_test.go` and `apply_test.go` go whole; `zoom_integration_test.go` goes whole here because both its tests drive `SetContentRows` and the stack, and card 3 retires the bracket they pin.
`TestPinBootOptionsLocked_ClearsTheWindowResizedHook` replaces `TestPinBootOptionsLocked_InstallsTheSignalEntry` on its fake with one assertion: boot issues both unpins and no `set-hook` that installs a `window-resized` entry.
`TestPinGeometryOptionsLocked` asserts both unpins on both watchdog values, and no `pane-border-status` pin, and gains a row where the fake fails the window-scoped `set-hook -u` and the session-scoped unpin and the binding pins still follow; `TestAttachArgv_PreflightOnAKnownGoodSession` asserts the attach pre-flight issues it, so both acceptance paths (boot, attach) are pinned.
`TestAttachArgv_EveryOtherDegradedPathYieldsBareArgv`, `TestAttachArgv_SelectsTheRelevantWindow`, `TestAttachArgv_MultiClientWarning` and their helpers expect the bare argv plus selection on the known-good path too.
`TestSaveState_ThenLoadState_RoundTrips` and `TestLoadState_RetiredDisplayKeysIgnoredAndLayoutUnchanged` drop `ContentRows`, `Size` and the `render.Rules` comparison, the latter comparing the decoded displays alone.
`TestCalleeFailuresLogOncePerEpisode` drops the live-box, border-title and pin steps.
At the tmux tier, the layout-height attach geometry tests go; `setupAttachGeometryFixture` and `TestAttachGeometry_DegradedPathStillAttaches` drop the layout config, and the client-exit test stays as it is.
`TestSelvageNeverGetsZeroHeightLayoutCell` goes, and so does `TestEnsureSelvagePaneLocked_RecoversWhenTheBottomPaneIsTooSmallToSplit`, which pins only the retired retry.
The doc comment of `goodAttachLive` and `TestSmokeLifecycle`'s comments stop justifying their fixtures by `planLayout`, `applyLayoutLocked` and the Selvage band.
`TestDeadSelvagePaneIsHealedByUpWithoutCorruptingLayout` asserts that after a fresh `up`, after an `up` healing a Selvage corpse that is the window's only pane, and after a `resize-window` of the session run with no client attached (the fixture attaches none, so `window-size latest` has no client to follow), the shell window lists exactly one pane, the alive Selvage, whose height equals the window's height.
`TestStrandOps_RealTmux` drops its `ReplaceKeepsTopSlotAtCollapsedRows` height assertion and its `paneIDsByTop` use, keeping the slot and pane-survival assertions.
`TestBar_LineTwoListsSessionsInIDOrderAndTheBorderNamesStrands` drops its border-title half.
`TestSmokeWarmPath` counts attach's round trips without the chain's readbacks.
`topContentDisplay`, `TestAddStrandLocked`'s content and sharing rows, `newBindingsFixture` and `TestWindowOrder_RealTmux` stop building a `Size`: the size refusal rows go, and the displays keep their anchors for card 5.
`TestUpdateStrandLocked` drops its `TopPositionalDisplayRejected` row, which pinned the content-sized refusal this card removes, and keeps every other row for card 5.
`TestMultiplexerContract`'s step (c) comment stops citing `apply_test.go` and `render.Rules`, which this card deletes.
The two reedcli dot-fill smokes measured the resize artifact the pins and the hook caused, and go whole.

**ImpactSummary:** removes render's layout surface and reed's shell-window apply, pins, hook array and attach chain; the only cross-package callers are `AttachArgv`'s, in `internal/reedcli` and `internal/loomcli`, whose signature is unchanged.

**Commit:** 2: reed: retire the shell window's stacked layout and resize hook
