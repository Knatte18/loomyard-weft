# Card 3 — Retire the zoom bracket, the strand pane buttons and Alt+z

**Delete:**
- `internal/reedengine/zoom.go`
- `internal/reedengine/zoom_test.go`
- `internal/reedengine#zoomBindingCondition`
- `internal/reedengine#paneClickCommand`
- `internal/reedengine#strandButtonFormat`
- `internal/reedengine#TestBindings_AltZTogglesStrandAndSelvageZoomInTheStrandsWindowOnly`

**Create:**
- `plan:internal/reedengine#Engine.withBootOpLock` -> `func (e *Engine) withBootOpLock(fn func() error) error`

**Edit:**
- `internal/reedengine#Engine.AddStrandUnless`
- `internal/reedengine#Engine.UpdateStrand`
- `internal/reedengine#Engine.RemoveStrand`
- `internal/reedengine#Engine.ReplaceStrand`
- `internal/reedengine#Engine.Up`
- `internal/reedengine#Engine.EnsureSession`
- `internal/reedengine#Engine.Resume`
- `internal/reedengine#Engine.AttachArgv`
- `internal/reedengine#shellClickCommand`
- `internal/reedengine#statusClickBranches`
- `internal/reedengine#bindingArgvs`
- `internal/reedengine#shellButtonFormat`
- `internal/reedengine#statusFormatButtons`
- `internal/reedengine#consolePaneOption`
- `internal/vscode#keybindingsBlockEntries`
- `internal/reedengine#TestUp_BootValidation`
- `internal/reedengine#TestCalleeFailuresLogOncePerEpisode`
- `internal/reedengine#TestBindingArgvs_BindsTheFiveAltKeysAndTheStatusClickInTheRootTableOnly`
- `internal/reedengine#TestBindings_StatusBarClicksSelectAndZoomByRangeType`
- `internal/reedengine#TestBar_LineOneButtonsFollowTheWindowZoomAndWaitMarks`

**Intent:** D7: with no strand pane in the shell window, the per-strand pane buttons, the strand click, Alt+z and the zoom-keeping wrapper that guarded ops against a zoomed strand pane go.

`zoom.go` goes whole: `keepZoomLocked`, `withOpLockKeepingZoom`, `parseZoomState`, the zoom state format and the record flag.
`withBootOpLockKeepingZoom`'s boot refusals stay as `withBootOpLock`: under the op lock it runs `validateBootConfig` and `preflightBootHost`, then `fn`, with no zoom read; it lives beside `Up` in `lifecycle.go`.
`Up` and `Resume` take `withBootOpLock`; `EnsureSession`, `UpdateStrand`, `RemoveStrand` and `AttachArgv` take `withOpLock`; `AddStrandUnless` and `ReplaceStrand` run their bodies directly where they ran `keepZoomLocked`, so a refused call still reads no tmux state.

Bindings: `bindingArgvs` drops the `M-z` `bind-key` and adds `{"unbind-key", "-n", "M-z"}` ahead of the other argvs, since a `bind-key` re-pin never removes a binding; a server an older build pinned therefore loses Alt+z at the first stale-stamp re-pin, which `repinStaleBindingsLocked` already triggers.
The unbind removes only the root-table `M-z`, which only reed bound.
`statusClickBranches` drops its `pane` branch, so the re-bound click binding carries no strand-button dispatch; `shellClickCommand` selects the shell window and no longer unzooms it, and `paneClickCommand` and `zoomBindingCondition` go.
The `bindings.go` header names four Alt keys and drops the pane click, and `consolePaneOption`'s doc stops saying the console takes Alt+z.
`lyx ide spawn`'s VS Code block forwards the reed Alt keys to the integrated terminal, so `keybindingsBlockEntries` drops its `alt+z` entry; `TestSeedKeybindings` and `TestMergeKeybindings` build their expected block from that table and need no change.
Bar: `strandButtonFormat` goes, and `statusFormatButtons`'s `@lyx_strands` branch gives the shell button alone; `shellButtonFormat` is lit while its window is active, with no zoom term.
The `bar.go` header drops the strands' pane buttons from line 1's order.

Tests: `zoom_test.go` goes whole; `TestCalleeFailuresLogOncePerEpisode` drops its zoom step.
`TestUp_BootValidation` already asserts each boot refusal at `Up` and `Resume` before any tmux round trip, so it reaches `withBootOpLock`'s two refusals unchanged; its doc stops naming the zoom read.
`TestBindingArgvs_BindsTheFiveAltKeysAndTheStatusClickInTheRootTableOnly` asserts the `unbind-key -n M-z` argv, no `M-z` `bind-key`, and no `pane` branch in the click dispatch, so it fails if the unbind is dropped.
At the tmux tier, `TestBindings_AltZTogglesStrandAndSelvageZoomInTheStrandsWindowOnly` goes; `TestBindings_StatusBarClicksSelectAndZoomByRangeType` drops its pane-click case and asserts a shell click selects the shell window and leaves `window_zoomed_flag` unchanged, and that with `@lyx_strands` unset on every window a shell click leaves the current window where it is; `TestBar_LineOneButtonsFollowTheWindowZoomAndWaitMarks` asserts line 1 renders no `@strand` pane button, the shell button lit while the shell window is active.

**ImpactSummary:** eight engine ops lose the zoom bracket with their signatures unchanged, and the server-wide bindings lose Alt+z and the pane click; outside `internal/reedengine` only `internal/vscode`'s forwarded-key table changes, dropping `alt+z`.

**Commit:** 3: reed: retire the zoom bracket, strand pane buttons and Alt+z
