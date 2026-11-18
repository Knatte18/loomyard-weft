# Card 4 — reed-title-names-own-window

**Edit:**
- `internal/reedengine#Engine.createOwnWindowPaneLocked`
- `internal/reedengine#Engine.moveShellWindowStrandsLocked`
- `internal/reedengine#placeApart`
- `internal/reedengine/render#Display`
- `internal/reedengine#TestOwnWindow_RealTmux`
- `internal/reedengine#TestListReconcilePanesLocked_MovesShellWindowStrands`
- `internal/reedengine/doc.go`

**Create:**
- `plan:internal/reedengine#ownWindowName` -> `func ownWindowName(d render.Display, role string) string`

**Uses:**
- `internal/reedengine#shellWindowName`
- `internal/reedengine#overviewWindowName`
- `internal/reedengine#strandLabel`

**Intent:**
`Display.Title` also names the window of a strand reed places in a window of its own, so the checklist's window reads `pawl` (D3).

`ownWindowName` returns `d.Title`, or role when the title is empty or equals a window name reed reserves, `shellWindowName` or `overviewWindowName`.
Every path that names a window of its own for one strand calls it with `strandLabel(s.Name)` as role:
- `createOwnWindowPaneLocked`'s ungrouped open, and its `placeApart` open for a grouped strand whose group does not share a window or whose shared window is full;
- `moveShellWindowStrandsLocked`'s `break-pane -n`, which reads the strand's display before it rewrites the anchor and keeps the display's `Title`, so a moved strand's window and its later relaunch take the same name.

A strand that opens or joins its group's shared window ignores `Title`, and the shared window keeps its group's name.
An overview strand keeps `overviewWindowTitle`, which `openOwnWindowLocked` applies over the name it is passed; `openOwnWindowLocked`'s comment, "named name", stays true and is not changed.

Bound: the title only names a window.
tmux allows two windows of one name, reed addresses an own window by its window id and finds the shell window by its console pane, and `OpenWindow`'s same-name lookup skips windows carrying `@lym_own`, so a title two windows share gives two buttons of one name and nothing else.
The reserved-name fallback keeps the shell's and the overview's names unambiguous to the operator.

Comments: `Display.Title`'s comment says it names the overview window, and the window of an own-window strand placed in a window of its own, empty or a reserved name reading as the role, and that a group's shared window ignores it.
`placeApart`'s and `createOwnWindowPaneLocked`'s comments say the window is named by the title, else the role.
`internal/reedengine/doc.go`'s overview-anchor paragraph keeps its overview naming and gains the own-window naming rule beside it, naming the checklist window a pawl run gets beside a beam's runs strand.

Tests:
- `TestOwnWindow_RealTmux` (tier2) gains one subtest, "ATitleNamesAWindowOfItsOwn".
  It adds through `e.AddStrand`, reading names with the test's `display(window, "#{window_name}")`:
  - an ungrouped own-window strand titled `pawl`, which wants window name `pawl`;
  - ungrouped strands titled `shell` and `overview`, which want their roles;
  - a grouped strand of a size-5 group, placed apart, titled `apart`, which wants `apart`;
  - the first strand of a new size-2 group titled `ignored`, which wants the group's name.
  The untitled cases stay covered by the existing `OpensDetachedNamedByRoleWithoutMovingTheViewer`, `GroupOfFiveGivesEachItsOwnWindow` and `GroupOfTwoSharesOneWindowSideBySide` subtests.
- `TestListReconcilePanesLocked_MovesShellWindowStrands` (tier0) gets a `wantNames` row field naming the `-n` value of each `break-pane`, asserted beside the existing argv shape.
  The existing rows that move a strand set `wantNames` to the moved strands' names, which the `bound` closure sets to the guid, `strandLabel` answering the whole name when it does not parse.
  Two new rows break out a strand written as a literal: `bound`'s display with a `Title` added.
  One titled `pawl` wants `pawl`; one titled `shell` wants the strand's name.

**ImpactSummary:** Local to reed's window naming: two call paths in `internal/reedengine` change their `-n` argument; no strand today sets `Title` outside the overview anchor, so every existing window keeps its name until card 3's checklist spec sets one.

**Commit:** 4: Reed names a window of its own by its strand's title
