# Card 8 — Reconcile migrates markerless wefts and retires the Bolt marker

**Edit:**
- `internal/fabricengine#Topology.Reconcile`
- `internal/fabricengine#Topology.add`
- `internal/fabricengine#ReconcileResult`
- `internal/fabriccli#runReconcile`
- `internal/fabricengine_test#TestReconcile_RefusesEmptyAnchorMarkerInsteadOfWiringAtTheRoot`
- `internal/fabricengine/doc.go`

**Delete:**
- `internal/fabricengine#refuseEmptyAnchorMarker`

**Create:**
- `plan:internal/fabricengine#committedMarker` -> `func committedMarker(repoDir, rev string) (marker string, found bool, err error)`
- `plan:internal/fabricengine#treeAnchor` -> `func treeAnchor(repoDir, rev string, names []string, fallback string) (string, error)`
- `plan:internal/fabricengine#retireBoltAnchorMarker` -> `func retireBoltAnchorMarker(rec *Mutations, l *lyxcwd.Location) (string, error)`
- `plan:internal/fabricengine_test#TestReconcile_MigratesMarkerlessWefts` -> `func TestReconcile_MigratesMarkerlessWefts(t *testing.T)`
- `plan:internal/fabricengine#TreeAnchorForTest` -> `func TreeAnchorForTest(repoDir, rev string, names []string, fallback string) (string, error)`
- `plan:internal/fabricengine_test#TestTreeAnchor` -> `func TestTreeAnchor(t *testing.T)`

**Uses:**
- `internal/fabricengine#refuseEmptyOwnMarker`
- `internal/fabricengine#commitAnchorMarker`
- `internal/lyxcwd#NamedWorktree`
- `internal/hubforge#ReanchorPair`
- `internal/fabricengine#ReconcilePairResult`
- `internal/fabricengine#Topology.repairPairWiring`
- `internal/fabricengine#SwapPushRecorderForTest`

**Intent:**
D8, and D5's markerless value.
`committedMarker` reads `.lyx-anchor` at the root of `rev`'s tree in `repoDir` through `git show`/`ls-tree`; an absent file is `found` false, and a value that is empty after trimming is returned as `""` with `found` true, so each caller decides (the census counts it as none, D2's refusals refuse it).
`treeAnchor` is D5's one rule: the committed non-empty marker at `rev`, else the shallowest directory of `rev`'s tree that holds one of the wired `names` (`.` when the root holds one; ties at one depth broken by byte order), else `fallback`, which callers fill with D2's resolution (`lyxcwd.NamedWorktree(...).AnchorRel`).
Every later card that compares or writes a markerless branch's anchor calls `treeAnchor`.

`Topology.Reconcile` drops the pass-wide `refuseEmptyAnchorMarker` (deleted) and judges markers per pair, the prime's weft first, before the pair loop's wiring repair:
a present weft whose own marker is empty is reported failed through `refuseEmptyOwnMarker`'s message in `ReconcilePairResult.Error`, its wiring untouched, and the pass goes on;
a present weft with no committed marker at `HEAD` gets `treeAnchor(weft, "HEAD", names, fallback)` committed with `commitAnchorMarker` (subject `fabric reconcile: record anchor <value>`), a detail line naming the value, and its detached push through `SpawnPairPush` of the pair's Location, as other weft writers spawn it; its wiring repair then runs at that anchor, read back through `NamedWorktree`.
After the pair loop `retireBoltAnchorMarker` removes `_board/.lyx-anchor`, and a leftover `.fabric-anchor` beside it, in one board commit and push through `Bolt`, only when the census holds: the prime's weft and every local pair's weft carry a committed non-empty marker at `HEAD`, and every weft branch whose warp sibling is on origin carries one on its already-fetched remote-tracking ref; a weft branch with no warp sibling on origin is an orphan and is not counted.
`retireBoltAnchorMarker` lives in card 6's `internal/fabricengine/anchormarker.go` and deletes both files through an `os.Root` opened at the board worktree (`root.Remove`, an absent file no error, inside the one `BoltWrite` it hands `Bolt.PullThenCommitWritten`, so it never writes over a stale Bolt; a skipped write keeps the marker for a later reconcile and is reported `kept` with the skip's detail), as `hubworkspace_write.go` removes its files; neither `os.Remove(` nor `RemoveAll(` appears, so `destructiveGuardAllowlist` gains no entry, and the files are fabric's own metadata, never operator content.
The census reads local refs only, makes no network call, and a branch whose ref is not fetched, a marker commit not yet pushed or an empty committed marker keeps the Bolt marker for a later reconcile.
It returns its verdict (`removed`, or `kept` with the first branch that keeps it); `ReconcileResult` gains `BoltAnchor string` (`json:"bolt_anchor,omitempty"`) and `runReconcile` puts it in the envelope as `bolt_anchor`.
A hub with no Bolt marker reports nothing.
`lyx fabric unwire` keeps leaving every marker.

`Topology.add`'s adopt path (D5): an adopted weft branch with no committed marker, or with a committed marker that is empty after trimming, which counts as none as the census counts it, gets `treeAnchor` of its tip committed with `commitAnchorMarker` right after its worktree is added, and every path Add builds for that pair, origin record included, uses the adopted pair's own Location read back with `NamedWorktree`; an adopted branch that carries a marker uses it.
The fork path keeps card 7's fork-point anchor.

`internal/fabricengine/doc.go`'s anchor section gains the migration and the census, replacing the `refuseEmptyAnchorMarker` paragraph.

Tests:
`TestReconcile_RefusesEmptyAnchorMarkerInsteadOfWiringAtTheRoot` becomes a table: an empty marker in the prime's weft, and one in a pair's weft, each reports that pair failed naming `.lyx-anchor` and `git -C`, wires nothing at the warp root, leaves the real junctions at the anchor, and still reconciles the other pairs.
`TestReconcile_MigratesMarkerlessWefts` (tier1, hubforge, serial since it swaps the package push seam with `SwapPushRecorderForTest`) makes a weft markerless with a commit that removes its `.lyx-anchor` (`git rm` and commit in its records worktree) and re-anchors the prime with `hubforge.ReanchorPair`; it covers: a markerless pair whose content sits at `.` while the prime was re-anchored at `sub` after it forked gets `.` committed, and a markerless prime gets its content's anchor, the prime's migration standing first in the push recorder's calls, before any pair's; each marker commit's push is recorded by the swapped recorder for that pair's records worktree; the Bolt marker and a `.fabric-anchor` beside it are removed in one board commit once every local weft and every fetched weft ref carry a marker; the Bolt marker is kept while a fetched remote weft ref lacks one, while a committed marker is empty, and while a marker commit is unpushed; the census makes no network call, shown by pointing every weft remote's URL at a missing path before the pass, after which the census still decides from the fetched refs and the pass reports no error; an orphan remote weft branch is not counted; the same task, removed locally and added again over its pushed markerless branch, adopts with that anchor written, and so does one whose pushed branch carries an empty marker; `lyx fabric unwire` of a migrated pair leaves its marker committed and in place.
`TestTreeAnchor` (tier1, `gitkit` repo, through `TreeAnchorForTest`) pins D5's rule in one table: a committed marker wins; a markerless tree with a wired name at the root answers `.`; one with a wired name under both `b/x` and `a/y`, at one depth, answers `a/y` by byte order; one with wired names at depths 1 and 2 answers the shallower; a tree holding no wired name answers the fallback; an empty committed marker counts as none.

**ImpactSummary:** the pass-wide empty-marker refusal becomes per pair with its one caller, Add's adopt path and Reconcile gain marker writes, one result field and one envelope key are added; hubs that already carry markers reconcile as before.
