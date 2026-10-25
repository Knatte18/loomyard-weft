# Card 14 — The integration that carries a weft across an anchor change

**Create:**
- `plan:internal/fabricengine#ErrIntegrateConflict` -> `var ErrIntegrateConflict error`
- `plan:internal/fabricengine#IntegrateOutcome` -> `type IntegrateOutcome struct`
- `plan:internal/fabricengine#integrateWeft` -> `func integrateWeft(rec *Mutations, weftPath, upstream, target string, names []string, fallback string) (IntegrateOutcome, error)`
- `plan:internal/fabricengine#IntegrateWeftForTest` -> `func IntegrateWeftForTest(weftPath, upstream, target string, names []string, fallback string) (IntegrateOutcome, error)`
- `plan:internal/fabricengine_test#TestIntegrateWeft` -> `func TestIntegrateWeft(t *testing.T)`

**Uses:**
- `plan:internal/fabricengine#treeAnchor`
- `plan:internal/fabricengine#committedMarker`
- `internal/fabricengine#replayWeft`
- `internal/fabricengine#cherryPickOnto`

**Intent:**
D10's integration onto a target anchor `T`, which card 15's integrating `--set` runs as its step 4; it is not reachable from any verb before card 15.
`integrateWeft` rebuilds the weft checked out at `weftPath` on its fetched upstream tip `U` (`upstream`, a SHA):
it replays, oldest first, each local commit `U` lacks, skipping each commit that changed `.lyx-anchor`, with each changed path moved from under the anchor its parent commit names (`treeAnchor` of the parent, `fallback` when markerless) to under `T`; paths outside every wired name, and the marker, are replayed unmoved;
then, when `U`'s anchor (`treeAnchor` of `U`) is not `T`, it applies the move from that anchor to `T` on the result: each wired name's tracked content re-rooted and `.lyx-anchor` set to `T`, in one commit.
It builds every commit off the checked-out branch with git plumbing over a temporary index file made through `internal/lyxtemp` (`read-tree`, `update-index`, `write-tree`, `commit-tree`), never touching the weft's working tree, index or branch, and only once the result is whole moves the branch with `reset --keep` to it.
A replay whose paths conflict stops with the weft unchanged and returns `ErrIntegrateConflict` naming the conflicting local commit and its paths, with the way forward: tag the local tip by hand (`git -C <weft> tag archive/<slug>/<tip> <tip>`), save the changes of the conflicting commit and of every later local commit outside the weft, reset the weft branch by hand to the conflicting commit's parent (`git -C <weft> reset --keep <commit>^`), re-run `lyx fabric anchor <slug> --set <T>`, then write the saved changes back at `T` and commit them; the tag keeps every dropped commit reachable until they are back.
`IntegrateOutcome` reports the new tip, the replayed and skipped commits.
`IntegrateWeftForTest` is the `export_test.go` shim the test drives.

`TestIntegrateWeft` (tier1, a records repo with a bare origin built with `gitkit`) covers: `U` re-anchored from `a` to `b` while the local branch holds two records commits at `a`, integrated onto `b`, puts both commits on `U` with their paths under `b` and the marker `b`; integrating onto `c`, which neither side names, also moves `U`'s content to `c`; a local commit that changed the marker is skipped; a markerless local parent tree re-roots from D5's value; a conflicting local commit stops with `ErrIntegrateConflict` naming it and its paths, the branch, index and working tree unchanged.
