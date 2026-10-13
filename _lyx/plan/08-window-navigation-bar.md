# Card 8 — window-navigation-bar

**Create:**
- `plan:internal/reedengine#shellButtonFormat` -> `const shellButtonFormat = ""`
- `plan:internal/reedengine#shellClickCommand` -> `const shellClickCommand = ""`
- `plan:internal/reedengine#TestBindings_AltUpDownStepWindowsInIndexOrderPastHiddenOnes` -> `func TestBindings_AltUpDownStepWindowsInIndexOrderPastHiddenOnes(t *testing.T)`

**Delete:**
- `internal/reedengine#viewButtonFormat`
- `internal/reedengine#viewClickCommand`
- `internal/reedengine#stepCommand`
- `internal/reedengine#stepStopCondition`
- `internal/reedengine#stepBindingCondition`
- `internal/reedengine#TestBindings_AltUpDownCycleStrandPanesAndSelvageWrappingAndKeepingTheZoom`

**Edit:**
- `internal/reedengine#bindingArgvs`
- `internal/reedengine#statusClickBranches`
- `internal/reedengine#statusFormatButtons`
- `internal/reedengine#TestBindingArgvs_BindsTheFiveAltKeysAndTheStatusClickInTheRootTableOnly`
- `internal/reedengine#TestBindings_StatusBarClicksSelectAndZoomByRangeType`
- `internal/reedengine#TestBar_LineOneButtonsFollowTheWindowZoomAndWaitMarks`
- `internal/reedengine/doc.go`

**Intent:** Make windows the unit the operator steps between, in the order cards 5 and 6 fix, and give the shell window the button VIEW had (D6).

`bindingArgvs` binds `M-Down` to `next-window`, then (`\;`) `if-shell -F -t : '#{@lyx_hidden}' 'select-window -t :^'`, so a step that lands on a hidden window goes on to the session's first window, the overview.
It binds `M-Up` to `previous-window`, then `if-shell -F -t : '#{@lyx_hidden}'` running `selectStrandsWindowCommand`, so a step back onto a hidden window lands on the window carrying `@lyx_strands`, the shell.
Both wrap with tmux's own window wrap and work in every window: the `@lyx_strands` gate on the keys goes, and so do `stepCommand`, `stepStopCondition` and `stepBindingCondition`, since stepping between the shell window's panes is dropped; a click on a strand's pane button still reaches a pane.
Since hidden windows sit after the shell (card 5), one conditional step reaches a visible window; on psmux, where the order is best-effort, a step may land on a hidden window.
`M-z` keeps zooming only in the strands' window, unchanged.

The status bar's line 1 loses the leading VIEW button.
`statusFormatButtons`' window loop renders, for the window carrying `@lyx_strands`, `shellButtonFormat` followed by that window's strand pane buttons, so the window buttons follow window index order: the overview's first, agents in start order, the shell's last.
`shellButtonFormat` is a `range=user|shell` button labelled `shell`, lit (white, black text, bold) while that window is active and not zoomed and unlit otherwise, exactly as VIEW was lit; the placeholder declaration heads above stand for these format strings.
`statusClickBranches` dispatches the `shell` range to `shellClickCommand`, which keeps VIEW's action: select the strands' window and unzoom it when zoomed.
The overview window's button is an own-window button with no `@strand_color`, which renders as the plain window-button style.
The file comments of `bar.go` and `bindings.go`, and `internal/reedengine/doc.go`'s bindings and bar paragraphs, describe the window steps and the `shell` button and drop VIEW and pane stepping.

`TestBindingArgvs_BindsTheFiveAltKeysAndTheStatusClickInTheRootTableOnly` keeps its key list and replaces the step subtest with the exact `M-Up` and `M-Down` argvs above, and its range-type list becomes `shell`, `pane`, `window`, `session`.
`TestBindings_AltUpDownCycleStrandPanesAndSelvageWrappingAndKeepingTheZoom` is deleted with the behavior it pinned, and `TestBindings_AltUpDownStepWindowsInIndexOrderPastHiddenOnes` replaces it on `newBindingsFixture`: it adds an own-window strand and an overview strand and two windows carrying `@lyx_hidden` after the shell, each made by a plain `new-window -d -a` after the last window plus `set-option -w @lyx_hidden 1` as `newBarFixture` does, since the fixture's recorder `lyx` exits at once and would close an `OpenWindow` window, then drives Alt+Down from the overview through every visible window to the shell and on, past both hidden windows, to the overview, and Alt+Up from the overview, wrapping past both hidden windows, to the shell.
`TestBindings_StatusBarClicksSelectAndZoomByRangeType` clicks ` shell ` where it clicked ` VIEW `, with the same unzoom and return-to-the-strands'-window assertions.
`TestBar_LineOneButtonsFollowTheWindowZoomAndWaitMarks` replaces its VIEW lit and unlit strings with the `range=user|shell` ones, asserts line 1 holds no ` VIEW `, and asserts the shell button comes after the `extra` window's button and every own-window button, the last visible window; after its own-window step it adds an overview strand and asserts the `overview` window's button is the first button on line 1, in the plain unlit window style.
`internal/reedengine`'s budget row is unchanged: one top-level test leaves and one arrives.

**ImpactSummary:** The server-wide Alt+Up, Alt+Down and status-click bindings and line 1 of the status bar change for every reed session on the next pin; all their builders and constants live in `bar.go` and `bindings.go` and are read nowhere else.
