# Card 9 — kwinplace-planner

**Create:**
- `plan:internal/kwinplace#Grid` -> `type Grid struct`
- `plan:internal/kwinplace#Capacity` -> `func Capacity(screen Rect, minTileWidth, minTileHeight int) Grid`
- `plan:internal/kwinplace#GridFor` -> `func GridFor(windows int, room Grid) Grid`
- `plan:internal/kwinplace#Anchor` -> `func Anchor(snap Snapshot, places Places, parent string) (string, bool)`
- `plan:internal/kwinplace#PlanInput` -> `type PlanInput struct`
- `plan:internal/kwinplace#Placement` -> `type Placement struct`
- `plan:internal/kwinplace#Plan` -> `func Plan(in PlanInput) Placement`
- `plan:internal/kwinplace#TestCapacityAndGrid` -> `func TestCapacityAndGrid(t *testing.T)`
- `plan:internal/kwinplace#TestAnchor` -> `func TestAnchor(t *testing.T)`
- `plan:internal/kwinplace#TestPlan` -> `func TestPlan(t *testing.T)`

**Extends:**
- `internal/kwinplace/doc.go`

**Intent:** The pure planner of D3 step 2: snapshot, rules and places in, target desktop, grid and window order out (D4 to D8, D11).

`Capacity` gives `floor(width / minTileWidth)` columns capped at 5 and `floor(height / minTileHeight)` rows capped at 2, each at least 1 (D5).
`GridFor` gives 1 x `n` while `n` is at most the room's columns, else 2 rows of `ceil(n / 2)` columns (D6).

`Anchor` reads the parent's entry from `places`:
a settled entry gives the desktop of the parent's live `Window.Matches` in the snapshot, or its `LastDesktop` when it has no live window, and `true`;
a missing or unsettled entry gives `false`, which sends the flow back to the parent wait (D8).

`PlanInput` holds the snapshot, the places, the mode, the two tile sizes, the tiling output's name, the placed worktree, pid and title,
and the anchor desktop with whether it is D8's fallback.
`Plan`:
- finds the placed window by `Window.Matches`;
- counts a desktop's windows: normal, not minimized, on that one desktop alone, on the tiling output, the placed window left out (D4, D6);
- a desktop is full when its count reaches `Capacity` of the tiling output's geometry (D5);
- the start desktop is the entry's home when that desktop still exists, else the anchor for `same` and the desktop after it, by desktop order, for `next` (D8);
- a full start passes to the desktop after it, and so on; past the last desktop the target is `""` and the warning names the start desktop (D8);
- orders the target's windows: those homed there by order key, then the rest by current zone, untiled ones last in stacking order, ties keeping the current order;
  the placed window sorts by its own key when the target is its home, else last (D6);
  a window's current zone is `Snapshot.ZoneOf`, or for an untiled one the zone whose box inset by the tile's padding starts at its frame's top-left corner,
  as a clamped window's does; a root with no children holds no zones;
- returns the target, the grid `GridFor` gives for the counted windows plus the placed one, the zone order, whether it overflowed from home and whether it is homeless.
The order holds only windows on the target desktop and the placed window, so an apply touches no other desktop,
and a desktop at or over capacity is never a target (D11).
Mode `off` never reaches `Plan`: the flow records and moves nothing (card 10).
The prime's anchor is the flow's choice, not the planner's (card 10).

Tests, `tier0`, over hand-built snapshots and places, kwinplace's own types:
- `TestCapacityAndGrid`: 2560x1440 at 500x600 gives 2x5, 1536x960 gives 1x3, a 1x1 minimum caps at 2x5;
  3 windows with 5 columns give 1x3, 6 give 2x3, 7 give 2x4 (D5, D6).
- `TestAnchor`: a settled parent with a live window anchors on that window's desktop; a settled parent with no live window on its last desktop;
  an unsettled or missing parent answers false.
- `TestPlan` rows:
  - a window on another output is not counted (D4); capacity minus one windows plus the placed one is not full (D5);
  - windows keyed 2, 0 and 1 reopening in that order end in key order; a window on two desktops is not counted;
    a clamped, untiled window counts in the zone its frame starts at;
    untiled windows not homed there go last, in stacking order; two windows with one order key keep their current order;
    the placed window goes last when the target is not its home;
    on a desktop whose root tile has no children every window is untiled and ordered by stacking (D6);
  - `next` from desktop 1 with 2 full lands on 3; past the last desktop the target is empty and the warning names the start;
    a home wins over the mode; a home desktop the snapshot no longer holds falls to the anchor; a full home overflows from itself (D8);
  - the order holds no window of another desktop (D11).
Together they add under 0.2 seconds to one `tier0` run.
`doc.go` gains a paragraph on the planner.

**ImpactSummary:** Pure functions in the new package; `doc.go` gains a paragraph, and the flow on card 10 is the only caller.
