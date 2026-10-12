---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/reedengine/spawn.go:278
    summary: Relaunching an own-window strand whose pane is dead rebinds it to a new pane and never kills the old pane, so a dead pane is left behind in a shared group window
    fix: In launchStrandLocked, record the strand's previous s.PaneID before createOwnWindowPaneLocked runs. After the new pane is created and bound, when the previous id is non-empty, differs from the new id and is listed in the session listing as a pane of an own window, run kill-pane on it. Kill it after the new pane exists so a shared window that held only that pane is never ended mid-launch. Add a case to TestOwnWindow_RealTmux where a member of a shared group dies, the strand is relaunched through Resume or add --if-absent, and the group window holds one pane per strand with no dead pane.
  - id: F2
    severity: NIT
    class: consistency
    location: internal/reedengine/ownwindow.go:106
    summary: The ownWindowPlacement constants mix iota for placeApart with literal values 1 and 2 for the other two
    fix: Declare placeNewShared and placeJoinShared with bare names after placeApart = iota, dropping the explicit "= 1" and "= 2" and the repeated type, so the block reads as one iota sequence.
---

### [MEDIUM] Relaunch of a dead own-window strand leaves its old pane in a shared window

**Location:** internal/reedengine/spawn.go:278 (`launchStrandLocked`), with `internal/reedengine/ownwindow.go` `ownWindowsHoldingNoStrand` and `planReconcile` in `internal/reedengine/reconcile.go`.
**Issue:** An own-window strand whose pane is dead stays bound and counts as present, so reconcile never kills it, and `Resume` and `add --if-absent` both relaunch it.
`launchStrandLocked` then creates a new pane and overwrites `s.PaneID`, so the old pane is no longer bound to any strand.
For a strand in its own window, the next reconcile ends that window because it carries `@lyx_own` and holds no bound pane, and the old pane goes with it.
For a strand in a shared group window the window still holds other bound panes, so `ownWindowsHoldingNoStrand` never selects it, and nothing kills an unbound dead pane inside an own window.
The dead pane stays in the shared window, takes a share of the even layout, and outlives the strand until the whole group ends.
No test covers a relaunch inside a shared window.
Window 0 does not have this problem because reconcile kills its dead panes.

### [NIT] Mixed iota and literal values in the placement constants

**Location:** internal/reedengine/ownwindow.go:106
**Issue:** `placeApart` uses `iota` while `placeNewShared` and `placeJoinShared` repeat the type with explicit `1` and `2`, so the block reads as two conventions.

## Checks that found nothing

- `@lyx_own` and `@lyx_group` are set by window id in `openOwnWindowLocked`. Reconcile ends only a window carrying `@lyx_own` that holds no bound pane, and groups are found through `groupLiveCount` by `@lyx_group`, never by name. `@lyx_unseen` and `@strand_color` are read only by the status-bar format, not by Go.
- `listStrandPanes` is unchanged. Own-window panes enter only through `presentPanes` over the session-wide listing, and `planReconcile` keeps its kill decisions and survival rule on `live`. `recordedPaneIDs` skips own-window panes, so no layout or Selvage decision sees them.
- `windowSplitLayout` reads an empty `window_split` as `width`, and `validateWindowConfig` runs at boot (`up` and `resume`), at add, at replace, and at removal, where it degrades to a `warning`.
- `killStrandPanes` moves a viewer on an ending window to the strands' window before the kill and returns the surviving windows for relayout. Shuttle's stop paths remove dead own-window strands through `strandRemovable`, and `removeDiedOwnWindowStrand` writes `died-capture.txt` before the removal.
- Card 8's Delete group holds: `removeStrandIfLive` has no remaining reference in Go code.
- `lyx shuttle open` refuses an empty role and a covered pause, and reed numbers a held role rather than refusing it. `OrElse` joins two single pipelines, and `Open` composes the pane line through `shell.Chain`, so the `||` fallback and the trailing statements compose.
- `lyx gate test` over the touched packages passes, and `lyx loom lint-comments` reports ok.
- The new top-level tests each pin behavior no existing test covers, and the test budget rows were raised to match.
