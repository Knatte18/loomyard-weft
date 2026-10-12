# Card 4 — reed-selvage-cycle

**Create:**
- `internal/reedengine#consolePaneOption`
- `internal/reedengine#Engine.markSelvagePaneLocked`
- `internal/reedengine#TestBindings_AltZTogglesStrandAndSelvageZoomInTheStrandsWindowOnly`
- `internal/reedengine#TestBindings_AltUpDownCycleStrandPanesAndSelvageWrappingAndKeepingTheZoom`

**Delete:**
- `internal/reedengine#TestBindings_AltZTogglesAStrandsZoomAndLeavesSelvageAndOtherWindowsAlone`
- `internal/reedengine#TestBindings_AltUpDownStepThroughStrandPanesOnlyWrappingAndKeepingTheZoom`

**Edit:**
- `internal/reedengine#zoomBindingCondition`
- `internal/reedengine#stepCommand`
- `internal/reedengine#Engine.ensureSelvagePaneLocked`
- `internal/reedengine#Engine.AttachArgv`
- `internal/reedengine#newBindingsFixture`
- `internal/reedengine#TestBindingArgvs_BindsTheFiveAltKeysAndTheStatusClickInTheRootTableOnly`
- `internal/reedengine#TestEnsureSelvagePaneLocked_SplitsOnceBelowTheBottommostPane`
- `internal/reedengine/doc.go`

**Intent:**
Build D9: Selvage joins the Alt+Up/Alt+Down cycle and Alt+z zooms it, in the strands' window only.

The bindings find Selvage through a display-only pane option that no Go decision reads, as `PATTERN-wait-mark-display-only` holds for the wait mark.
`consolePaneOption`, declared in `paneoptions.go` beside the other display options, names it; the option's literal and its Go name carry no "selvage", because `selvagepane_enforcement_test.go` confines Selvage-named identifiers to `selvagepane.go`.
`markSelvagePaneLocked`, in `selvagepane.go`, sets it (`set-option -p -t <Selvage pane> @lyx_console 1`) on `st.SelvagePaneID` when one is recorded, logging a failure and never failing the step.
`ensureSelvagePaneLocked` calls it right after recording a newly split Selvage, and `AttachArgv`'s pre-flight calls it beside `reassertStrandPaneOptionsLocked`, so a session booted by an older lyx gets the marker at its next attach.
Selvage still never gets `@strand` or `@strand_color`.

Bindings, in `bindings.go`:
- `zoomBindingCondition` becomes true only in the window carrying `@lyx_strands` and there on a strand pane, on the marked Selvage pane, or in a zoomed window; a hidden batten window's pane carries no `@lyx_strands`, so Alt+z does nothing there.
- `stepCommand` steps once more the same way only when the landed pane carries neither `@strand` nor the marker, so the cycle runs over every strand pane and Selvage in physical order, Selvage last (it is the bottom-most pane), wrapping, with `select-pane -Z` keeping the zoom, so a zoomed step onto Selvage shows it full screen.
  `stepBindingCondition` already limits the steps to the strands' window.
- The doc comments of both, and `doc.go`'s navigation-bindings paragraph and its pane-options paragraph, state the new cycle, the Alt+z scope and the marker.
A zoomed Selvage stays zoomed through every layout re-apply: `keepZoomLocked` re-zooms whatever pane was zoomed, which card 3's zoom test pins for Selvage.

Tests:
- `TestBindingArgvs_BindsTheFiveAltKeysAndTheStatusClickInTheRootTableOnly` pins the new M-z condition string and the M-Up/M-Down command strings, each naming `consolePaneOption`.
- `TestEnsureSelvagePaneLocked_SplitsOnceBelowTheBottommostPane` asserts the marker's `set-option -p` call on the newly split pane, issued after its split.
- `newBindingsFixture` (tmux tier) adds `alpha` as a top content strand (share 1/3, min 3) and `beta` positional, so the cycle, VIEW and the strand buttons run over a top strand placed by card 2 above a below-parent strand, the prime's `runs`-above-orch shape; it leaves its second window as it is, since that window carries no `@lyx_strands` exactly as a hidden batten window does (both binding conditions read only `@lyx_strands`, `@strand` and the marker), and marking it `@lyx_hidden` would drop the window button `TestBindings_StatusBarClicksSelectAndZoomByRangeType` clicks; and it unsets the marker on Selvage before attaching, so every bindings test passes only when the attach pre-flight re-asserts it.
- `TestBindings_AltZTogglesStrandAndSelvageZoomInTheStrandsWindowOnly` replaces the Alt+z test: Alt+z zooms and unzooms a strand, zooms and unzooms Selvage, and in the second window zooms neither that window nor the strands' window.
- `TestBindings_AltUpDownCycleStrandPanesAndSelvageWrappingAndKeepingTheZoom` replaces the step test: from `alpha`, Alt+Down reaches `beta`, then Selvage, then wraps to `alpha`; Alt+Up from `alpha` reaches Selvage; a zoomed Alt+Down from `beta` lands on Selvage with the zoom flag still 1; Alt+Up in the second window moves nothing.
- `TestBindings_StatusBarClicksSelectAndZoomByRangeType` runs unchanged on the new fixture: VIEW shows both strands and a strand button zooms its pane alone.

**ImpactSummary:** server-wide bindings and one Selvage pane option; two tmux-tier tests replaced one-for-one, so the budget row is unchanged.
