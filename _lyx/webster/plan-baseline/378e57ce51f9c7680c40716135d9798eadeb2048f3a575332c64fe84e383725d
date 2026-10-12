# Card 3 — reed-content-rows-op

**Create:**
- `internal/reedengine#Engine.SetContentRows`
- `internal/reedengine#TestSetContentRows`

**Edit:**
- `internal/reedengine#applyOpts`
- `internal/reedengine#Engine.applyLayoutLockedOpts`
- `internal/reedengine#TestZoomBracket_ZoomedStrandSurvivesEveryReedStep`
- `internal/reedengine/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/reedengine#strandNamed`
- `internal/reedengine#installIfAbsentTmux`

**Intent:**
Add D4's one engine operation through which a content strand's pane process reports its line count, in `strand.go` beside `UpdateStrand`.

`SetContentRows(name, paneID, rows)` refuses `rows < 1` before taking any lock, then runs under `withOpLockKeepingZoom` (the zoom bracket, `keepZoomLocked`): it requires the session (`requireSessionLocked`), loads the state, finds the strand whose full name equals `name` (`strandNamed`), and refuses, each with an error naming the strand and the reason, an unknown name, a strand whose `Display.Size.Kind` is not `render.SizeContent`, and a strand whose recorded `PaneID` is not `paneID`.
A count equal to the recorded `ContentRows` returns nil and touches neither reed.json nor tmux.
Otherwise it sets that strand's `ContentRows`, persists the state, lists the strands' window's panes and re-applies the layout through `applyLayoutLockedOpts` with `SkipFocus` (an operator's cursor never moves when a run starts) and the new `InstallPins`.
It changes nothing but that strand's `ContentRows` and the layout and pins derived from it.
Bound: a hand-run pane process or a replaced one is refused by the name and pane checks (its `$TMUX_PANE` is not the recorded pane), so it can resize no strand; the pane process (card 5) retries at its next poll after any refusal, which is how a report made before `AddStrand` has recorded the pane succeeds once it has.

`applyOpts` gains `InstallPins bool`: with `SkipFocus` it still installs the window-resized hook array (`installResizePinsLocked`), because a changed content height changes the pins the hook replays; the watchdog's re-apply passes neither and keeps today's behaviour.
`applyLayoutLockedOpts`' doc comment states the new field.

`doc.go` names `SetContentRows` among the zoom bracket's call sites and adds a paragraph stating the operation, its refusals and its bound.

Tests:
- `TestSetContentRows` (new, `strand_test.go`), table-driven over `installIfAbsentTmux` answering pane lines for both `%1` and `%2` (an apply with fewer than two live panes issues nothing), with a state on disk holding a top content strand on `%1` and a positional strand on `%2`; that fake answers every `display-message` with its generation string, so the live box, title-row and zoom reads take their fallbacks (the configured box, no title row, not zoomed): `rows` 0 is refused and issues no tmux call; an unknown name, the positional strand's name, and the content strand's name with pane `%2` are each refused, leaving reed.json's `ContentRows` unchanged and issuing no `select-layout`; the content strand's name, `%1` and a new count persists that count and issues `select-layout`, the `resize-pane` pins and the `set-hook` rebuild with no `select-pane`; the same count again issues no `select-layout`.
- `TestZoomBracket_ZoomedStrandSurvivesEveryReedStep` (tmux tier) gains steps after its resume: add a top content strand (share 1/3, min 3) and assert the child is still the zoomed pane; report 5 rows for it through `SetContentRows` with its pane id and assert the child is still zoomed; zoom Selvage, report 6 rows and assert Selvage is still the zoomed pane; then unzoom and assert the content strand's `#{pane_height}` is 6, its title row taken at window row 0.
- `test-budget.yaml`: `internal/reedengine` 255 → 256.

**ImpactSummary:** one new exported reedengine method and one `applyOpts` field; the watchdog's and every existing caller's apply is unchanged.
