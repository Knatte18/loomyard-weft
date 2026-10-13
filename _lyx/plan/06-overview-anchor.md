# Card 6 — overview-anchor

**Create:**
- `plan:internal/reedengine/render#AnchorOverview` -> `const AnchorOverview Anchor = "overview"`
- `plan:internal/reedengine#isOverview` -> `func isOverview(d render.Display) bool`
- `plan:internal/reedengine#overviewWindowInsertArgs` -> `func overviewWindowInsertArgs(windows []sessionWindow) []string`

**Edit:**
- `internal/reedengine/render#Anchor`
- `internal/reedengine/render#ValidateDisplay`
- `internal/reedengine/render#TestPartitionByAnchor`
- `internal/reedengine#isOwnWindow`
- `internal/reedengine#StrandStatus`
- `internal/reedengine#validateDisplay`
- `internal/reedengine#refuseWindowMove`
- `internal/reedengine#Engine.addStrandLocked`
- `internal/reedengine#Engine.openOwnWindowLocked`
- `internal/reedengine#Engine.killStrandPanes` -> `func (e *Engine) killStrandPanes(st *ReedState, paneIDs []string, selectOnEnd bool) (reapPIDs []int, leftWindows []string, endedCurrent bool)`
- `internal/reedengine#Engine.RemoveStrand`
- `internal/reedengine#Engine.ReplaceStrand`
- `internal/reedengine#TestAddStrandLocked`
- `internal/reedengine#TestUpdateStrandLocked`
- `plan:internal/reedengine#TestWindowOrder`
- `plan:internal/reedengine#TestWindowOrder_RealTmux`
- `internal/reedengine/doc.go`

**Intent:** Add the `overview` anchor: one strand per session in a window of its own that is always the session's first window (D1).

`render.AnchorOverview` joins the closed vocabulary; `render.Anchor`'s doc and the package doc in `types.go` name five values, and `ValidateDisplay` accepts an overview display whatever its size, as it does own-window and hidden, since the declaration is ignored.
`partitionByAnchor` already places nothing outside top and below-parent, and `TestPartitionByAnchor` gains the row "OverviewNotPlaced" pinning that.

In `reedengine`, `isOwnWindow` reports a strand that lives in a window of its own, now own-window or overview, so reconcile, `presentPanes`, `recordedPaneIDs` (the strands' window resolver's window-0 fallback, D3), `anyPlacedStrand` and the relaunch's old-pane kill treat an overview pane exactly as an own-window one; `isOverview` names the overview anchor alone.
`StrandStatus.OwnWindow`, which `Status` fills from `isOwnWindow`, now reports an overview strand too, and its field doc says "the strand lives in a window of its own, own-window or overview"; its readers outside reed, shuttle's stop and wait paths, only ever see shuttle runs' strands.
`validateDisplay` admits `overview` and its refusal text lists it.
`refuseWindowMove` also refuses an update between overview and own-window, with its existing way forward.
`addStrandLocked` refuses an overview add while another strand of the state already carries the overview anchor, live or dormant: "strand %q already holds the overview window; way forward: lyx reed remove --name <its role>, then add this strand again".
The input that reaches it after every upstream check passes is a valid overview display under a name no strand holds, added to a state that records an overview strand under another name; a second overview under the same name is refused earlier by the name check.
`ReplaceStrand` removes the old strand before its add, so replacing the overview strand itself, or replacing another strand with one, is never refused.

`openOwnWindowLocked` opens an overview strand's window named `overview` with `overviewWindowInsertArgs`, which is `-b -t <lowest-index window>` (the first window of the listing, hidden ones included), so the overview holds the lowest index whenever it is opened; with no listed window it opens plain, and the psmux fallback of card 5 applies.
The overview window carries the own marker like an own window, so reconcile ends it when no strand holds it, and sets neither `@lyx_unseen` nor `@strand_color`: the overview strands set no segment, so its button takes the plain window-button style.

`killStrandPanes` reports whether the session's current window was among the windows the kill ends, reading the current window once only when a window ends, and selects the strands' window for such a viewer only when `selectOnEnd` is set (card 7 replaces that choice).
`RemoveStrand` passes `true`.
`ReplaceStrand` passes `false` when the replacement is an overview strand and `true` otherwise, and when the kill ended the current window and the replacement is an overview strand it selects the new strand's window once the launch succeeded, a failure logged at Warn.
A replace whose old strand shared a window that stays open, such as a `top` strand in the shell window, ends no window and switches nothing.
`ReplaceStrand`'s doc comment and `internal/reedengine/doc.go`'s anchor section describe the overview anchor, its placement, its one-per-session refusal and the replace selection.

`TestAddStrandLocked` gains a subtest "SecondOverviewUnderAnotherNameRefused" with a state of its own, seeded with a dormant overview strand under another name (no pane, so nothing launches), asserting the error contains that strand's name and `lyx reed remove` and that no record was registered; the shared `rejected` table and its state stay as they are.
It also gains a subtest calling `validateDisplay` directly on an overview display with a size declared and wanting nil, which needs no tmux since an accepted overview add would launch a pane.
`TestUpdateStrandLocked` gains "OverviewToOwnWindowRejected" and "OwnWindowToOverviewRejected".
`TestWindowOrder` gains rows for `overviewWindowInsertArgs`: the lowest window is the target, a hidden lowest window included, and no window gives nothing.
`TestWindowOrder_RealTmux` gains rows: an overview strand added after two agent windows holds the lowest index, is named `overview` and carries no `@lyx_unseen` or `@strand_color`; replacing a `top` strand with an overview strand opens the overview first, removes the old pane from the shell window and leaves the shell window selected; replacing the overview strand while its window is viewed selects the new overview window; replacing the overview strand while agent windows are live keeps overview, agents in start order, shell.

**ImpactSummary:** One new anchor value admitted at both validation layers, a refusal on add, and the kill helper re-signed; `killStrandPanes`'s two callers, `RemoveStrand` and `ReplaceStrand`, are on this card, and nothing outside `reedengine` adds an overview strand until card 9.
