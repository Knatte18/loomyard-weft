# Card 4 — geometry-button-context

**Create:**
- `internal/reedengine#ButtonContext`
- `internal/reedengine#ButtonBeam`
- `internal/hubgeom#buttonContext`
- `internal/hubgeom#buttonBeam`

**Edit:**
- `internal/reedengine#Geometry`
- `internal/hubgeom#reedGeometry`
- `internal/hubgeom#TestReedGeometry`
- `internal/hubgeom_test#TestGeometryOverRealHub`

**Extends:**
- `internal/hubgeom/doc.go`
- `internal/reedengine/doc.go`

**Uses:**
- `internal/reedengine#ButtonPlace`
- `internal/hubgeom#ParentOf`
- `internal/hubgeom#hubWithOrigin`
- `internal/desktophook#LoadConfig`
- `internal/fabricengine#ReadOriginFor`
- `internal/fabricengine#PrimeName`
- `internal/fabricengine#ConfigDirs`
- `internal/lymcwd#NamedWorktree`

**Intent:**
Tell reed, lazily, the session's place, its placeholder values and the machine's buttons (D4), so the engine never resolves a location (`PATTERN-cwd-resolution`) and building a geometry stays free of reads.

reedengine (`geometry.go`, type declarations only):
- `ButtonContext` holds `Place ButtonPlace`; `Slug`, `Worktree` and `Session`, the three cheap placeholder values; `MachineButtons yaml.Node`, desktop's `buttons` node; and `Beam func() (ButtonBeam, error)`, which resolves `{beam}` and is called only when the render needs it (card 6).
- `ButtonBeam` holds `Anchor`, the beam's anchor path, empty when the beam's anchor directory is absent on this machine; `Name`, the beam's worktree name, which the render's Warn names; and `PrimeAnchor`, the prime's anchor path, a task's run directory when `Anchor` is empty (D7).
- `Geometry` gains `Buttons func() (ButtonContext, error)`, documented beside `SpawnOrder`: nil in standalone mode, where the row shows `follow` alone, and called by the status pins only, at boot and attach.
- The header comment of `geometry.go`, which says the file declares `Geometry` alone, names the two button types it now declares beside it.

hubgeom (a new file `buttons.go`):
- `reedGeometry` sets `Buttons: buttonContext(l, prime)`.
- The closure `buttonContext` returns, when called:
  - `Place`: `ButtonPlacePrime` when `prime`; otherwise the pair's own origin record, read with `fabricengine.ReadOriginFor(l, l.WorktreeName)` as `ResolveParent` reads it, gives `ButtonPlaceBeam` when it carries `Beam` and `ButtonPlaceTask` otherwise, an absent record included.
    A read error is returned, wrapped naming the worktree; the render then shows `follow` alone.
  - `Slug` `l.WorktreeName` (the prime's own name in the prime), `Worktree` `l.AnchorPath()`, which `PaneCwd` already equals, and `Session` `reedengine.SessionName(l.WorktreePath())`, the geometry's own session name.
  - `MachineButtons`: the `Buttons` node of `desktophook.LoadConfig(fabricengine.ConfigDirs(l))`; a failed load is logged at Warn naming the error and leaves the node zero, with the place and values intact (D3).
  - `Beam`: `buttonBeam(l, place)`.
- `buttonBeam` returns a closure that reads nothing until called.
  In the prime and in a beam session it returns the session's own anchor as `Anchor` and `PrimeAnchor` (a beam's `PrimeAnchor` is unused, since only a task's run directory falls back) and its own name.
  For a task it resolves the prime with `fabricengine.PrimeName(l)` and `lymcwd.NamedWorktree(l.HubPath, primeName)`, then the parent with `hubgeom.ParentOf(prime, l.WorktreeName)` (`PATTERN-pair-parent`):
  `""` gives the prime's anchor and name;
  a parent slug gives `lymcwd.NamedWorktree(l.HubPath, parent).AnchorPath()` as `Anchor` when that path is an existing directory, else an empty `Anchor` with `Name` the parent slug, since `ParentOf` names a beam absent on this machine and `NamedWorktree` builds its path without checking it.
  `PrimeAnchor` is the prime's anchor in every task case.
  A failed prime-name, parent or anchor read is returned wrapped; the render then shows `follow` alone.

`reedengine/doc.go` gains a sentence that the geometry tells the button context through `Geometry.Buttons`, nil in standalone mode.
`hubgeom/doc.go` gains a paragraph: ReedGeometry also tells reed a lazy button-context closure (buttons.go) carrying the place, the placeholder values and desktop's machine buttons, and a `{beam}` resolver that reads the parent through `ParentOf` over the prime's location only when called.

Tests:
- `TestReedGeometry` (`tier0`, internal package) asserts on each row that `Buttons` is non-nil, and calls it.
  Every row asserts `Worktree` equals the fixture's anchor path, so the subpath-anchored task row, whose anchor differs from its worktree root, fails if `l.WorktreePath()` is told instead; it also reads `ButtonPlaceTask`, since that fixture has no origin record.
  The prime row reads `ButtonPlacePrime` with `Slug` `some-worktree` and `Session` `reedengine.SessionName(worktreeRoot)`, and its `Beam` closure returns the anchor path as `Anchor` with no git spawn.
  New rows over `hubWithOrigin`: the record `{"parent_branch":"main","beam":true}` reads `ButtonPlaceBeam`, `{"parent_branch":"main"}` reads `ButtonPlaceTask`, and the unparseable record `{` makes the closure return an error.
  A row writing `buttons: {task: [{label: x, run: y}]}` to the board's `desktop.machine.yaml` (`configengine.MachineFile`) under an anchor carrying `_lym/` gets that node as `MachineButtons`.
  A subtest that does not call `t.Parallel`, since it reads the process-global logger output through `logcapture.CaptureVerbose` and says so in a comment, writes `buttons: {task: {}}` there and gets a zero node, the place intact and the Warn logged.
- `TestGeometryOverRealHub` (`tier1`, external package, over the hubforge hub) gains a step after the task-parent step: for a task pair whose origin names the prime, `Beam` returns the prime's anchor as `Anchor` and `PrimeAnchor`; with its origin's `parent_worktree` set to another live pair and `parent_branch` to that pair's own branch it returns that pair's anchor; with `parent_worktree` set to `gone-beam`, absent on this machine, and `parent_branch` to a branch no worktree holds, it returns an empty `Anchor`, `Name` `gone-beam` and the prime's `PrimeAnchor`; with its origin record then overwritten by the unparseable `{`, the `Beam` closure of the context read before the overwrite, called after it, returns an error wrapping the parent read (`ParentOf`'s origin read).
  The live-pair and `gone-beam` rows set `parent_branch` off the prime's branch because `ParentOf` answers the prime whenever the origin's `parent_branch` equals the prime's checked-out branch, before it returns `parent_worktree`, so a row keeping hubforge's prime branch never reaches the parent-anchor or empty-`Anchor` branch.

**ImpactSummary:** `Geometry` gains one field every existing constructor leaves nil (standalonegeom and the reedengine tests keep compiling with no row); `reedGeometry` sets it; no reader exists until card 6.

**Commit:** 4: Tell reed the session's button place and placeholder values
