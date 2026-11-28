# Card 10 — kwinplace-planner

**Create:**
- `internal/kwinplace#Grid`
- `internal/kwinplace#Capacity`
- `internal/kwinplace#GridFor`
- `internal/kwinplace#Anchor`
- `internal/kwinplace#CurrentOrder`
- `internal/kwinplace#PlanInput`
- `internal/kwinplace#Placement`
- `internal/kwinplace#Plan`
- `internal/kwinplace#TestCapacityAndGrid`
- `internal/kwinplace#TestAnchor`
- `internal/kwinplace#TestCurrentOrder`
- `internal/kwinplace#TestPlan`

**Extends:**
- `internal/kwinplace/doc.go`

**Intent:** The pure planner of D3 step 2: snapshot, rules and places in, target desktop, grid and window order out (D4 to D8, D11).

`Capacity` gives `floor(width / minTileWidth)` columns capped at 5 and `floor(height / minTileHeight)` rows capped at 2, each at least 1 (D5).
`GridFor` gives 1 x `n` while `n` is at most the room's columns, else 2 rows of `ceil(n / 2)` columns (D6).

`Anchor` reads the parent's entry from `places`:
a settled entry gives the desktop of the parent's live `Window.Matches` in the snapshot, or its `LastDesktop` when it has no live window, and `true`;
a missing or unsettled entry gives `false`, which sends the flow back to the parent wait (D8).

`CurrentOrder` is D6's current-order rule, the one both `Plan` and the flow's report update read.
Given one desktop's leaf tiles and the windows sitting on it, it returns their internal ids in current order:
- by current zone, the leaf whose `Windows` lists the window, else, for an untiled window,
  the zone whose box inset by the tile's padding starts at its frame's top-left corner, as a clamped window's does;
- untiled windows in no zone go last, in stacking order, and windows sharing a zone keep their stacking order;
- a root with no children holds no zones, so on such a desktop every window is ordered by stacking.

`PlanInput` holds the snapshot, the places, the mode, the two tile sizes, the tiling output's name, the placed worktree, pid and title,
`Prime` (the placed window is the prime's own), and the anchor desktop with whether it is D8's fallback.
`Plan`:
- finds the placed window by `Window.Matches`;
- counts a desktop's windows: normal, not minimized, on that one desktop alone, on the tiling output, the placed window left out (D4, D6);
- a desktop is full when its count reaches `Capacity` of the tiling output's geometry (D5);
- the start desktop is the entry's home when that desktop still exists;
  else, for the prime's own window, the anchor under both `same` and `next`;
  else the anchor for `same` and the desktop after it, by desktop order, for `next` (D8);
- a full start passes to the desktop after it, and so on; past the last desktop the target is `""` and the warning names the start desktop (D8);
- orders the target's windows: those homed there by order key, then the rest in `CurrentOrder`, ties in order key keeping the current order;
  the placed window sorts by its own key when the target is its home, else last (D6);
- returns the target, the grid `GridFor` gives for the counted windows plus the placed one, the zone order, whether it overflowed from home and whether it is homeless.
The order holds only windows on the target desktop and the placed window, so an apply touches no other desktop,
and a desktop at or over capacity is never a target (D11).
Mode `off` never reaches `Plan`: the flow records and moves nothing (card 11).
The prime's anchor, the current desktop, is the flow's choice (card 11); `Prime` keeps `next` from starting past it.

Tests, `tier0`, over hand-built snapshots, tiles and places, kwinplace's own types:
- `TestCapacityAndGrid`: 2560x1440 at 500x600 gives 2x5, 1536x960 gives 1x3, a 1x1 minimum caps at 2x5;
  3 windows with 5 columns give 1x3, 6 give 2x3, 7 give 2x4 (D5, D6).
- `TestAnchor`: a settled parent with a live window anchors on that window's desktop; a settled parent with no live window on its last desktop;
  an unsettled or missing parent answers false.
- `TestCurrentOrder` rows: tiled windows come in the order of the leaves that list them;
  a clamped, untiled window counts in the zone its frame starts at;
  untiled windows in no zone go last, in stacking order;
  on a desktop whose root tile has no children every window is untiled and ordered by stacking (D6).
- `TestPlan` rows:
  - a window on another output is not counted (D4); capacity minus one windows plus the placed one is not full (D5);
  - windows keyed 2, 0 and 1 reopening in that order end in key order; a window on two desktops is not counted;
    windows not homed there follow the homed ones in current order; two windows with one order key keep their current order;
    the placed window goes last when the target is not its home (D6);
  - `next` from desktop 1 with 2 full lands on 3; past the last desktop the target is empty and the warning names the start;
    a home wins over the mode; a home desktop the snapshot no longer holds falls to the anchor; a full home overflows from itself;
    the prime's own homeless window under `next`, with room on the current desktop it is anchored on, lands on the current desktop (D8);
  - the order holds no window of another desktop (D11).
Together they add under 0.2 seconds to one `tier0` run.
`doc.go` gains a paragraph on the planner.

**ImpactSummary:** Pure functions in the new package; `doc.go` gains a paragraph, and the flow on card 11 is the only caller.
