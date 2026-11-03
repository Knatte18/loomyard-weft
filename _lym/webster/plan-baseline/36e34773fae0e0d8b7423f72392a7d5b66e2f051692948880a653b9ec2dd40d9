# Card 1 — reed-overview-order-key

**Create:**
- `plan:internal/reedengine#overviewWindowKeys` -> `func overviewWindowKeys(strands []Strand, windowByPane map[string]string) map[string]int`

**Edit:**
- `internal/reedengine/render#Display`
- `internal/reedengine/render#AnchorOverview`
- `internal/reedengine#overviewWindowInsertArgs` -> `func overviewWindowInsertArgs(windows []sessionWindow, keys map[string]int, order int) []string`
- `internal/reedengine#Engine.openOwnWindowLocked`
- `internal/reedengine#Engine.addStrandLocked`
- `internal/reedengine#Engine.ReplaceStrand`
- `internal/reedengine#refuseWindowMove`
- `internal/reedengine#isOverview`
- `internal/reedengine#bindingArgvs`
- `internal/reedengine/doc.go`
- `internal/reedengine/bar.go`
- `internal/reedengine/render#TestDisplayDecodesRetiredKeysIgnored`
- `internal/reedengine/render#TestDisplayMarshalsWithoutRetiredKeys`
- `internal/reedengine#TestWindowOrder`
- `internal/reedengine#TestAddStrandLocked`
- `internal/reedengine#TestUpdateStrandLocked`
- `internal/reedengine#TestWindowOrder_RealTmux`

**Uses:**
- `internal/reedengine#parseSessionWindows`
- `internal/reedengine#windowOfPane`
- `internal/reedengine#Strand`

**Intent:**
D1: the overview anchor holds several strands, ordered by a display order key.

`render.Display` gains `Order int` with the JSON tag `order,omitempty`, documented as the overview strand's place among the session's overview windows, read only for an overview strand, its zero value the first place.
A display an older lym recorded carries no `order` key and so decodes as key zero; that is D5's no-migration rule.
`AnchorOverview`'s comment drops "the session's one overview strand" and says an overview strand gets a window of its own among the session's first windows, in ascending order key.
`isOverview`'s doc comment says it reports whether d places a strand in an overview window, one of the session's first windows; only the comment changes.

Placement (`windoworder.go`):
- `overviewWindowKeys` maps the window id of each overview strand's bound pane to that strand's `Display.Order`, from the strands and the session's pane-to-window map; a strand with no pane, a pane the map lacks and a non-overview strand are left out.
- `overviewWindowInsertArgs` takes those keys and the new window's own order key.
  Among the listed windows the keys name whose key is not higher than `order`, it returns `-a -t <id>` after the highest-index one; with none, it returns `-b -t <id>` before the lowest-index window, hidden ones included, as today; with no window listed it returns nothing, the plain form.
  So equal keys sit in the order they last opened, and a window with a lower key opens before a higher one.
- `openOwnWindowLocked` builds the keys with `overviewWindowKeys(st.Strands, windowByPane)` from the pane map it already lists (an empty map when that listing failed, which places before the lowest window as today) and passes `s.Display.Order`.
  A relaunch's old window carries the strand's own key, so a relaunched overview window opens after it and the old window then ends, keeping its place among distinct keys.
  Its doc comment says the overview window opens after the last overview window whose key is not higher, else before the lowest-index window.

Refusals:
- `addStrandLocked` drops the loop that refused an overview add while another strand holds the anchor.
  That removes `ReplaceStrand`'s refusal while a third strand holds the overview too, since `ReplaceStrand` reached it only through `addStrandLocked`; `ReplaceStrand`'s doc comment drops its one-per-session sentence and keeps the select-on-replace rule unchanged.
  Bound: reed no longer caps a session's overview strands. `lym reed add` refuses the overview anchor as engine-only (`internal/reedcli/add.go`), and no shuttle caller sets it, so the runs strand and the loom status strand are the only overview spawners; both add under an explicit name, which `strandNameLocked` refuses while another strand holds it, so neither adds a second copy of itself.
- `refuseWindowMove` refuses an update of an overview strand whose `next.Order` differs from `current.Order`: `strand %q cannot change its overview order; way forward: lym reed remove --name %s, then have its spawner add it again with the new order`.
  Input reaching it: `UpdateStrand` on an overview strand with an overview display of another `Order`, which `validateDisplay` and the anchor-move check both pass; the way forward succeeds because the add no longer refuses a second overview.
  The add half names the spawner, not `lym reed add`, since `lym reed add` refuses the overview anchor as engine-only (`internal/reedcli/add.go`); the spawners are engine callers, the runs strand's `lym beam start` and the status strand's `lym loom start`, which reed does not name.

`doc.go`: the "Overview anchor" paragraph says the anchor places any number of strands, each alone in a window of its own at the session's lowest indexes in ascending order key, a new, relaunched or replacement one opening after the last overview window whose key is not higher, else before the lowest-index window; it drops the one-per-session refusal sentences and adds the `UpdateStrand` order refusal; "the one-per-session refusal" leaves the `isOverview` list.
The "Window order" paragraphs say the overview windows, in ascending order key, come first, and state the new `-a`/`-b` placement.
The status-bar sentence ("the overview's button is first") and the Alt+Up/Alt+Down sentence ("the session's first window, the overview") say the overview windows' buttons come first and the step wraps forward to the session's first window, the lowest-index overview.
`bindingArgvs`'s doc comment says the same of the step's forward target: the session's first window, the lowest-index overview; only the comment changes.
`windoworder.go`'s file header says the overview windows come first in ascending order key, then the agent windows in start order.
`bar.go`'s header says line 1 reads the overview windows first, then the agents in start order, then the shell button, and that it reads own windows, the overview windows among them.

Tests:
- `TestDisplayDecodesRetiredKeysIgnored` (tier0) gains the row `OverviewWithNoOrderKeyReadsKeyZero`: `{"anchor":"overview","focus":false,"title":"beam: x"}` decodes to `Display{Anchor: AnchorOverview, Title: "beam: x"}`, order zero.
- `TestDisplayMarshalsWithoutRetiredKeys` (tier0) asserts a zero `Order` marshals with no `order` key, and that `Display{Anchor: AnchorOverview, Title: "pawl", Order: 1}` round-trips equal.
- `TestWindowOrder` (tier0) gains a `keys` and an `order` field on its overview rows, the listing parsed by `parseSessionWindows`: a key-1 window listed at index 0 with a new key 0 gives `-b -t` the lowest window; a key-0 window at index 0 and a key-1 at index 1 with a new key 1 gives `-a -t` the key-1 window (equal keys after the last opened); the same listing with a new key 0 gives `-a -t` the key-0 window; the existing two overview rows keep their answers with no keys.
  A new table `overviewWindowKeys` subtest pins that an overview strand's bound pane maps its window id to the strand's `Order` (two overview strands with keys 0 and 1 in two windows give both entries), and that a non-overview strand, a strand with no pane and a pane the map lacks are left out.
- `TestAddStrandLocked` loses the `SecondOverviewUnderAnotherNameRefused` subtest; acceptance is pinned at tier2 below.
- `TestUpdateStrandLocked` (tier0) gains `OverviewOrderChangeRejected`: an overview strand with `Order: 0` updated to an overview display with `Order: 1` is refused, display unchanged; the row's error must contain both `way forward: lym reed remove` and `add it again with the new order`, so a message that drops the add half fails; and `OverviewFocusOnlyChangeAccepted`: the same order with `Focus` set is accepted.
  The test's doc comment adds the overview order change to the refusals it lists.
- `TestWindowOrder_RealTmux` (tier2) gains subtests appended after its last one, where the session holds the overview strand `status` titled `beam: prime` with key 0, no agent, the visible non-own `plain` window an earlier subtest opened, the shell and the hidden `batten:*` windows.
  Each subtest computes its want from the window listing, never as a literal: the expected overview titles in order, then every other visible window but the shell in listing order (the `plain` window and the agents), then `shell`, then the hidden windows.
  The existing `agentNames` helper excludes only the name `overview`, so these subtests compare the overview prefix by the explicit titles and take the rest of the visible windows with those titles excluded.
  - `AnOverviewWithAHigherKeyOpensAfterTheFirstThenAnAgentAfterBoth`: with the shell window selected, add `pawl` (`Order: 1`, `Title: "pawl"`, cmd `exit`), want the titles `beam: prime, pawl` first and the rest as computed; the `pawl` window's marks `[#{@lym_own}][#{@lym_unseen}][#{@strand_color}][#{@lym_overview}]` read `[1][][][1]` and the current window is still the shell window, so the second overview window keeps every property of the first; add an own-window agent, want it after `pawl` and before `shell`, hidden windows still last.
  - `RelaunchingOrReplacingEitherOverviewKeepsTheOrder`: wait for `pawl`'s pane to die, relaunch it with `AddStrand` `IfAbsent`, which relaunches its stored command `exit`, so the row asserts only the order after the relaunch; replace `pawl` with `ReplaceStrand`; select `beam: prime`'s window and replace it with `ReplaceStrand`; after each the names keep `beam: prime, pawl` first, and after the last the current window is the new `beam: prime` window.
  - `AddedInTheOtherOrderTheOverviewsEndInTheSameOrder`: remove `beam: prime`, add it again with key 0, want it before `pawl`.
  - `EqualKeysSitInTheOrderTheyLastOpened`: add `third` (`Order: 1`), want `beam: prime, pawl, third`; replace `pawl`, want `beam: prime, third, pawl`.
  Strand variables later subtests read stay as they are, since these run last.

**ImpactSummary:** reed alone: one display field, the overview placement with its one caller `openOwnWindowLocked`, the removed add refusal and a new update refusal; no caller outside reed changes, and the runs and status strands keep key zero until cards 3 and 4.

**Commit:** 1: reed: let a session hold several overview windows ordered by a display order key
