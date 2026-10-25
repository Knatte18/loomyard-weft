# Card 15 — fabricengine moves a worktree's anchor and recovers a killed move

**Edit:**
- `internal/fabricengine/doc.go`

**Create:**
- `plan:internal/fabricengine#AnchorChange` -> `type AnchorChange struct`
- `plan:internal/fabricengine#Topology.BeginAnchorChange` -> `func (t *Topology) BeginAnchorChange(pair *lyxcwd.Location, target string) (*AnchorChange, error)`
- `plan:internal/fabricengine#AnchorChange.Apply` -> `func (c *AnchorChange) Apply(rec *Mutations) error`
- `plan:internal/fabricengine#AnchorChange.Finish` -> `func (c *AnchorChange) Finish(rec *Mutations) error`
- `plan:internal/fabricengine#AnchorChange.Close` -> `func (c *AnchorChange) Close()`
- `plan:internal/fabricengine#validateAnchorTarget` -> `func validateAnchorTarget(pair *lyxcwd.Location, target string, names []string) error`
- `plan:internal/fabricengine#anchorFailBeforeCommitHook` -> `var anchorFailBeforeCommitHook func() error`
- `plan:internal/fabricengine#SetAnchorFailBeforeCommitHookForTest` -> `func SetAnchorFailBeforeCommitHookForTest(t *testing.T, hook func() error)`
- `plan:internal/fabricengine_test#TestAnchorChange` -> `func TestAnchorChange(t *testing.T)`

**Uses:**
- `plan:internal/fabricengine#MoveRecord`
- `plan:internal/fabricengine#MoveRecordPath`
- `plan:internal/fabricengine#ReadMoveRecord`
- `plan:internal/fabricengine#MoveInProgress`
- `plan:internal/fabricengine#moveLockPath`
- `plan:internal/fabricengine#integrateWeft`
- `plan:internal/fabricengine#treeAnchor`
- `plan:internal/fabricengine#committedMarker`
- `plan:internal/fabricengine#refuseEmptyOwnMarker`
- `plan:internal/lyxcwd#NamedWorktree`
- `plan:internal/hubforge#ReanchorPair`
- `internal/fabricengine#WireJunctionsWith`
- `internal/fabricengine#writeLaunchers`
- `internal/fabricengine#removeLaunchers`
- `internal/fabricengine#RegenerateHubWorkspace`
- `internal/fabricengine#ensureWeftLockDirAt`
- `internal/hubforge#CloneSecond`

**Intent:**
D7's `fabricengine` work, in three calls the CLI of card 17 interleaves with its session, run and dobby steps (Shared Decisions); D6's create reuses it in card 18.

`BeginAnchorChange(pair, B)` validates and refuses before touching anything, then holds the move lock:
it refuses an empty own marker (`refuseEmptyOwnMarker`);
`validateAnchorTarget` requires `B` to pass `ValidateAnchorRel`, be a directory git tracks at the pair's warp `HEAD` (`git ls-tree -d`), have no segment that is a link or a wired, `.lyx`, `.scratch` or hub-reserved name, and hold no real wired name (warp-pristine); a `B` the warp `HEAD` lacks while the warp's fetched upstream tracks it is refused naming an update of the warp checkout by hand to a commit that tracks `B`;
a held move lock is refused with `MoveInProgress`'s text, naming the record's pid, or naming the run as starting when the record's pid and start time are not a live process (`parentlock.ProcessAlive`);
with a standing record and a free lock (a killed run), a `B` other than the record's is refused naming the record's `B`, and the record's `A`, `B`, kind and `U` govern;
with no record, the weft must have a committed marker at `HEAD`, else it refuses naming `lyx fabric reconcile`, and must be clean, else it refuses: an uncommitted or untracked path under `A/<names>`, bar the never-tracked `A/.lyx`, naming `lyx fabric commit` or removing the files, and an uncommitted marker, naming `git -C <weft> restore .lyx-anchor`.
The kind: integrate when the weft's fetched upstream has commits its tip lacks and the two tips carry different markers; else move when `B` differs from `A`; else no-op, which `BeginAnchorChange` returns as a change whose `Apply` and `Finish` do nothing and which takes no lock.
It then takes the move lock (held by this process until `Close` or exit) and writes the record with this process's pid and start time; a landed re-run keeps the record's other fields.
A re-run whose step 4 did not land (a move landed when `HEAD`'s committed marker is `B`; an integration when it is `B` and the record's `U` is an ancestor of `HEAD`; no `U` means not landed) renames `B/.lyx` back to `A/.lyx` when only it exists, refuses naming both when both exist (merge `B/.lyx` into `A/.lyx` by hand and remove it), resets `A/<names>`, `B/<names>` and the marker, each excluding `.lyx`, to `HEAD` in index and working tree, deletes what `HEAD` does not track under `B/<names>`, and then runs the dirtiness check; a landed re-run renames `A/.lyx` to `B/.lyx` when only `A/.lyx` exists and skips step 4.

`Apply` runs D7 steps 3 to 7: it removes the pair's junctions and scratch link at `A` through the destruction chokepoint; holds the weft write lock across step 4; to move, moves each wired name's tracked content from `A` to `B`, renames `A/.lyx` to `B/.lyx`, writes the marker `B` last and commits all of it in one weft commit (a commit spanning `A/<names>`, `B/<names>` and the root marker, not an anchor-scoped `CommitRecordsPaths`); to integrate, writes the fetched upstream tip `U` into the record, runs `integrateWeft` onto `B` and renames `A/.lyx` to `B/.lyx`; then wires the junctions and scratch link at `B`, writes the launchers under `_launchers/<B>/<slug>` and removes `_launchers/<A>/<slug>` (for the prime, the menu launcher too), and regenerates the hub and beam workspace files.
A failure before the commit, or before the integration moves the branch (`anchorFailBeforeCommitHook` stands in for one under test), restores the working tree, the index, `A/.lyx` and the marker, rewires `A`, removes the record and releases the weft write lock, and returns the error.
`Finish` removes the record and spawns the pair's detached push (`SpawnPairPush`).

`internal/fabricengine/doc.go`'s anchor section describes the move, the record, the recovery and the integration.

`TestAnchorChange` (tier1, hubforge) first commits `sub/.keep` in the prime's warp before it adds pairs, so `sub` is a directory git tracks in every pair's warp, and covers:
a pair moved from `.` to `sub`, with an untracked file the test put in its `.lyx` beforehand (no dirt refusal for the never-tracked `A/.lyx`): its tracked content and `.lyx`, that file included, sit under `sub`, the marker commit is `HEAD`, junctions and scratch link exist at `sub` and none at `.`, its launcher moved to `_launchers/sub/<slug>`, the workspace files name `<slug>/sub`, the record is gone after `Finish`, and the exclude lines `/_lyx` and `/.lyx` that the prime and a second pair still wired at `.` need stay while `/sub/_lyx` is added;
each refusal above in a table, each leaving the weft `HEAD`, working tree and junctions unchanged and no record;
a failure through the hook before the move commit, and one before an integration moves the branch, each leave the pair wired at `A`, its weft clean and no record;
a re-run after a kill before the commit (record standing, lock free, content half moved) and after a kill between the commit and the wiring at `B` each finish at `B` keeping a file the test put in `.lyx`;
a not-landed re-run with both `A/.lyx` and `B/.lyx` present refuses naming both and the merge by hand, touching nothing;
a re-run after a killed integration on a checkout already at `B` finishes, its fetched upstream made by a second clone (`hubforge.CloneSecond`) that moved its own copy and pushed;
a `--set` equal to the inherited anchor on a child forked after its parent's change (made with `hubforge.ReanchorPair` on the parent) is a no-op.

**ImpactSummary:** one doc file; everything else is new code that only card 17's verb and card 18's create call.
