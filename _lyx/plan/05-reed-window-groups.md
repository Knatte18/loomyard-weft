# Card 5 — reed shares one window per declared group up to window_share_max

**Create:**
- `plan:internal/reedengine#WindowGroup` -> `type WindowGroup struct`
- `plan:internal/reedengine#ownWindowPlacement` -> `type ownWindowPlacement int`
- `plan:internal/reedengine#placeApart` -> `const placeApart ownWindowPlacement = iota`
- `plan:internal/reedengine#placeNewShared` -> `const placeNewShared ownWindowPlacement = 1`
- `plan:internal/reedengine#placeJoinShared` -> `const placeJoinShared ownWindowPlacement = 2`
- `plan:internal/reedengine#planOwnWindowPlacement` -> `func planOwnWindowPlacement(groups map[string]WindowGroup, d render.Display, shareMax, liveCount int, groupWindow string) (ownWindowPlacement, WindowGroup, error)`
- `plan:internal/reedengine#groupLiveCount` -> `func groupLiveCount(strands []Strand, group, self string, windows []windowPane) (count int, groupWindow string)`
- `plan:internal/reedengine#pruneWindowGroups` -> `func pruneWindowGroups(st *ReedState)`
- `plan:internal/reedengine#Engine.joinGroupWindowLocked` -> `func (e *Engine) joinGroupWindowLocked(groupWindow string, windows []windowPane, splitFlag string) (string, error)`
- `plan:internal/reedengine#TestPlanOwnWindowPlacement` -> `func TestPlanOwnWindowPlacement(t *testing.T)`
- `plan:internal/reedengine#TestJoinGroupWindowLocked_RefusesASilentlyReusedPane` -> `func TestJoinGroupWindowLocked_RefusesASilentlyReusedPane(t *testing.T)`

**Edit:**
- `internal/reedengine#ReedState`
- `internal/reedengine#Engine.launchStrandLocked`
- `internal/reedengine#Engine.removeStrandLocked`
- `internal/reedengine#dropStrands`
- `internal/reedengine/doc.go`
- `internal/reedengine#TestRemoveStrandLocked`
- `plan:internal/reedengine#TestOwnWindow_RealTmux`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/reedengine#windowSplitLayout`
- `plan:internal/reedengine#Engine.openOwnWindowLocked`
- `internal/reedengine#windowPane`
- `internal/reedengine#TmuxCmd.listWindowPanes`
- `internal/reedengine#isOwnWindow`
- `internal/reedengine#validateSplitCreatedNewPane`
- `internal/reedengine#strandLabel`

**Intent:**
D2 and D3's layout: a strand declaring a group joins the group's shared window while the group's declared size is at most `window_share_max`, and every other own-window strand gets a window to itself.

`ReedState` gains `WindowGroups map[string]WindowGroup \`json:"windowGroups,omitempty"\``, keyed by group name; reed.json stays tolerant of unknown keys and an older file decodes with none.
`WindowGroup` (in `ownwindow.go`) holds `Shared bool \`json:"shared"\`` (the group's first strand chose one shared window) and `Size int \`json:"size"\`` (the group size it declared, 0 read as 1).
Its doc comment says the record is reed's own, written by the group's first strand and dropped with its last.

`planOwnWindowPlacement(groups, d, shareMax, liveCount, groupWindow)`, pure:

- `d.Window` empty: `placeApart`, zero record, nil.
- The declared size is `max(d.WindowSize, 1)`.
- A record for `d.Window` whose `Size` differs: an error `window group %q is live with size %d, and this strand declares size %d; way forward: give the strand the group's size, or another group name`, naming both sizes.
- No record: a new one, `Shared: size <= shareMax`, so a later `window_share_max` edit never splits a group across shared and own windows; with a record, the recorded `Shared` decides, never `shareMax`.
- Not shared: `placeApart`.
- Shared with `liveCount >= Size` (a re-stepped seat added before its predecessor is removed): `placeApart`, leaving the shared window as it is.
- Shared with a non-empty `groupWindow`: `placeJoinShared`; otherwise `placeNewShared`.
- It returns the record it decided on, so the caller stores it.

`groupLiveCount(strands, group, self, windows)` counts the strands of `group` other than `self` (by guid) whose binding names a listed pane, alive or dead, since a dead pane still occupies the shared window; and returns the id of the window whose `Group` is `group` and that carries `Own`, found by the `@lyx_group` window option and never by name, empty when none is listed.

`launchStrandLocked`'s own-window branch:

- For `d.Window` empty it calls `openOwnWindowLocked(s, strandLabel(s.Name), "")` as before.
- Otherwise, before `groupLiveCount`, it re-lists the session's panes through `tmux.listWindowPanes(e.SessionName())` into `windows`, after any head reconcile that ended an own window, which card 3 keeps out of `killed`; so the placement and `joinGroupWindowLocked` read the current windows, never a group window or a pane that has gone.
- It then calls `planOwnWindowPlacement(st.WindowGroups, s.Display, e.cfg.WindowShareMax, count, window)` with `groupLiveCount(st.Strands, s.Display.Window, s.GUID, windows)`.
  A refusal returns before anything is created, so `addStrandLocked` drops the record and its launch script as for any failed launch, and the refusal reaches the add's caller.
- It stores the returned record in `st.WindowGroups` (creating the map), which the add's `SaveState` persists.
- `placeApart`: `openOwnWindowLocked(s, strandLabel(s.Name), "")`, a window named by the strand's role.
- `placeNewShared`: `openOwnWindowLocked(s, s.Display.Window, s.Display.Window)`, a window named by the group and carrying `@lyx_group`.
- `placeJoinShared`: `joinGroupWindowLocked(window, windows, splitFlag)`, with `splitFlag` and `layout` from `windowSplitLayout(e.cfg.WindowSplit)`, validated already at add and resume (card 2).
- After a join or a new shared window it runs `select-layout -t <window> <layout>` (`even-horizontal` for `width`, `even-vertical` for `height`); a failure is logged at Warn and never fails the launch.
  A window to itself gets no layout, and no size pins, resize hooks or Selvage live in any own window.

`joinGroupWindowLocked(groupWindow, windows, splitFlag)` splits the group window's first listed pane with `split-window -d <splitFlag> -t <pane> -c <PaneCwd> -P -F '#{pane_id}'`, where `-d` keeps the group window's active pane where it is (D4: a joiner never takes focus from an operator viewing that window), and runs `validateSplitCreatedNewPane` against the group window's listed panes (mapped with `livePane()`), so a silently reused pane is refused as in window 0 (D7).

`pruneWindowGroups(st)` deletes every record whose group no strand of `st.Strands` still names, so a later instance under the same name records afresh.
`removeStrandLocked` and `dropStrands` call it after they shrink `st.Strands`.

`internal/reedengine/doc.go`'s "Own windows" paragraph gains the grouping: the shared-or-apart choice, its record, the overflow rule, the size refusal and the even layout in the `window_split` direction.

Tests:

- New `TestPlanOwnWindowPlacement` in `internal/reedengine/ownwindow_test.go` (package `reedengine`), rows over `groups`, `d`, `shareMax`, `liveCount` and `groupWindow`, each asserting the placement, the record and the error:
  - `UngroupedIsApartWithNoRecord`.
  - `FirstOfAGroupOfTwoUnderTheCapStartsTheSharedWindow`: size 2, cap 2: `placeNewShared`, record `{Shared: true, Size: 2}`.
  - `SecondOfAGroupOfTwoJoins`: the record and a group window `@4`, live count 1: `placeJoinShared`.
  - `EitherArrivalOrderEndsShared`: the same group seen first from each strand's side gives the same record, then the same join.
  - `GroupOfFiveIsApart`: size 5, cap 2: `placeApart`, record `{Shared: false, Size: 5}`.
  - `ShareMaxZeroIsApart`: size 2, cap 0: `placeApart`.
  - `ZeroSizeReadsAsOne`: size 0, cap 2: `placeNewShared` with `Size: 1`.
  - `SizeMismatchIsRefusedNamingBothSizes`: record size 2, strand size 3: an error containing `size 2` and `size 3`.
  - `RecordedChoiceBeatsAChangedCap`: record `{Shared: true, Size: 2}`, cap 0, live count 1, window `@4`: `placeJoinShared`.
  - `OverflowPastTheRecordedSizeIsApart`: record `{Shared: true, Size: 2}`, live count 2: `placeApart`.
  - `SharedRecordWithNoLiveWindowStartsIt`: record shared, live count 0, no window: `placeNewShared`.
- New `TestJoinGroupWindowLocked_RefusesASilentlyReusedPane` in `ownwindow_test.go`, over `installFakeTmux`: with the group window `@4` listing `%5` and `split-window` answering `%5`, `joinGroupWindowLocked("@4", windows, "-h")` returns the split-guard error and no layout runs; with `split-window` answering `%6`, it returns `%6` and the recorded `split-window` argv carries `-d`, `-h` and `-t %5`.
- `TestRemoveStrandLocked` gains a group check: two strands of group `g` with a record; removing one keeps the record, removing the second drops it.
- `TestOwnWindow_RealTmux` gains subtests, each with fresh group names:
  - `GroupOfTwoSharesOneWindowSideBySide`: two `AddStrand`s with `Window: "pair"`, `WindowSize: 2` land in one window named `pair` carrying `@lyx_group` `pair`, its two panes side by side (equal `pane_top`, different `pane_left`); with that window selected before the second join, its active pane is still the first strand's pane after it, and the session's current window is the same.
  - `ADeadMemberStillCountsTowardTheGroup`: group `dying` (size 2) whose first strand runs `exit` and goes dead; the second joins its window (live count 1), and a third, while the dead pane is still bound, lands in a window of its own (live count 2).
  - `HeightSplitStacks`: with `e.cfg.WindowSplit = "height"`, a second group of two stacks (equal `pane_left`, different `pane_top`).
  - `GroupOfFiveGivesEachItsOwnWindow`: three strands of a size-5 group land in three windows, each named by its role.
  - `OverflowGetsAWindowOfItsOwn`: a third strand of the full `pair` group lands in a window of its own, and `pair`'s window keeps its two panes.
  - `ShareMaxEditBetweenJoinsDoesNotMoveTheSecondJoiner`: the first strand of group `late` joins with the cap at 2, the cap is set to 0, and the second still joins `late`'s window.
  - `SizeMismatchIsRefused`: a strand declaring size 3 for `pair` fails naming both sizes, and no window or pane is added.
  - `RelaunchOfADeadGroupMemberRejoinsItsWindow`: the first strand of a live group of two goes dead and `Resume` relaunches it into the group window beside its live partner, and its old dead pane stays in that window, unbound and never killed.
    With its partner removed first, the group window stays, since the dead pane is still bound until the relaunch, and the relaunch joins it beside that pane; a later `RemoveStrand` of the relaunched strand then ends the window, the unbound dead pane with it.
- `cmd/lyx/testdata/test-budget.yaml`: raise `internal/reedengine` by two, for `TestPlanOwnWindowPlacement` and `TestJoinGroupWindowLocked_RefusesASilentlyReusedPane`.

**ImpactSummary:** Adds one persisted `ReedState` field, a pure placement plan and the group branch of the own-window launch; window 0 strands and ungrouped own-window strands take the same path as after card 4.

**Commit:** 5: reed shares one window per declared group up to window_share_max
