# Card 15 — fabricengine moves a worktree's anchor and recovers a killed move

**Edit:**
- `internal/fabricengine/doc.go`

**Create:**
- `internal/fabricengine#AnchorChange`
- `internal/fabricengine#Topology.BeginAnchorChange`
- `internal/fabricengine#AnchorChange.Apply`
- `internal/fabricengine#AnchorChange.Finish`
- `internal/fabricengine#AnchorChange.Close`
- `internal/fabricengine#validateAnchorTarget`
- `internal/fabricengine#moveWeftContent`
- `internal/fabricengine#restoreMovePathsToHead`
- `internal/fabricengine#ownedMovePathspecs`
- `internal/fabricengine#UpstreamRunUnfinished`
- `internal/fabricengine#anchorFailHook`
- `internal/fabricengine#SetAnchorFailHookForTest`
- `internal/fabricengine_test#TestAnchorChange`

**Uses:**
- `internal/fabricengine#MoveRecord`
- `internal/fabricengine#MoveRecordPath`
- `internal/fabricengine#ReadMoveRecord`
- `internal/fabricengine#RemoveMoveRecord`
- `internal/fabricengine#MoveInProgress`
- `internal/fabricengine#moveLockPath`
- `internal/fabricengine#integrateWeft`
- `internal/fabricengine#treeAnchor`
- `internal/fabricengine#committedMarker`
- `internal/fabricengine#refuseEmptyOwnMarker`
- `internal/lyxcwd#NamedWorktree`
- `internal/hubforge#ReanchorPair`
- `internal/fabricengine#WireJunctionsWith`
- `internal/fabricengine#writeLaunchers`
- `internal/fabricengine#removeLaunchers`
- `internal/fabricengine#RegenerateHubWorkspace`
- `internal/fabricengine#ensureWeftLockDirAt`
- `internal/fabricengine#dirtinessNA`
- `internal/hubforge#CloneSecond`
- `internal/shedrun#TreeRunFileRels`

**Intent:**
D7's `fabricengine` work, in three calls the CLI of card 17 interleaves with its session, run and dobby steps (Shared Decisions); D6's create reuses it in card 18.
It also gives card 17 `UpstreamRunUnfinished`, the run-status reader for a worktree another machine owns, whose checked-out status goes stale once the marker change stops its weft pull.
`UpstreamRunUnfinished(pair, names)` reads the pair's own run status at the tree of its weft's fetched upstream tip, never the checked-out copy: the path is `TreeRunFileRels` of the pair's slug, falling back to the legacy `self` segment when the tree lacks the slug's directory, as the on-disk resolution does, joined under `treeAnchor` of that tip (fallback: `pair.AnchorRel`); it judges the status unfinished when its `state` is other than `done`, the rule card 17 applies to the checked-out copy, reports a status the tree lacks as finished, and reads `HEAD` when the weft has no upstream.

`BeginAnchorChange(pair, B)` validates and refuses before touching anything, then holds the move lock:
it refuses an empty own marker (`refuseEmptyOwnMarker`);
`validateAnchorTarget` requires `B` to pass `ValidateAnchorRel`, be a directory git tracks at the pair's warp `HEAD` (`git ls-tree -d`), have no segment that is a link or a wired, `.lyx`, `.scratch` or hub-reserved name, and hold no real wired name (warp-pristine), each refusal naming its next step:
a value `ValidateAnchorRel` rejects, or one with a link or reserved segment, names choosing another directory of the warp;
a `B` the warp `HEAD` lacks while the warp's fetched upstream tracks it names an update of the warp checkout by hand to a commit that tracks `B`, and one neither tracks names committing the directory in the warp checkout first;
a real wired name under `B` names moving it out of `B` in the warp checkout and committing that;
each warp update or commit names its cost, a code commit on the checked-out warp branch, shared by every machine that checks it out;
each then the same verb again;
a held move lock is refused with `MoveInProgress`'s text, naming the record's pid, or naming the run as starting when the record's pid and start time are not a live process (`parentlock.ProcessAlive`);
with a standing record and a free lock (a killed run), a `B` other than the record's is refused naming the record's `B`, and the record's `A`, `B`, kind and `U` govern;
with no record, the weft must have a committed marker at `HEAD`, else it refuses naming `lyx fabric reconcile`, and must be clean, else it refuses: an uncommitted or untracked path under `A/<names>`, bar the never-tracked `A/.lyx`, naming `lyx fabric commit` or removing the files, and an uncommitted marker, naming `git -C <weft> restore .lyx-anchor`.
The kind: integrate when the weft's fetched upstream has commits its tip lacks and the two tips carry different markers; else move when `B` differs from `A`; else no-op, which `BeginAnchorChange` returns as a change whose `Apply` and `Finish` do nothing and which takes no lock.
It then takes the move lock (held by this process until `Close` or exit) and writes the record with this process's pid and start time; a landed re-run keeps the record's other fields.
`Close` releases the move lock; when `Apply` never ran and this `BeginAnchorChange` wrote the record with none standing before, it first removes the record through `RemoveMoveRecord`, so an error of the verb's steps before `Apply`, with nothing moved, leaves no record, as D7 step 4's failure before the commit does; a record a re-run found standing is kept.
A re-run whose step 4 did not land (a move landed when `HEAD`'s committed marker is `B`; an integration when it is `B` and the record's `U` is an ancestor of `HEAD`; no `U` means not landed) renames `B/.lyx` back to `A/.lyx` when only it exists, refuses naming both when both exist (merge `B/.lyx` into `A/.lyx` by hand and remove it), resets `A/<names>`, `B/<names>` and the marker, each excluding `.lyx`, to `HEAD` in index and working tree and deletes what `HEAD` does not track under `B/<names>`, both through `restoreMovePathsToHead`, and then runs the dirtiness check; a landed re-run renames `A/.lyx` to `B/.lyx` when only `A/.lyx` exists and skips step 4.

`Apply` runs D7 steps 3 to 7: it removes the pair's junctions and scratch link at `A` through the destruction chokepoint; holds the weft write lock across step 4; to move, runs `moveWeftContent(rec, weft, A, B, names)`; to integrate, writes the fetched upstream tip `U` into the record, runs `integrateWeft` onto `B` and renames `A/.lyx` to `B/.lyx`; then wires the junctions and scratch link at `B`, writes the launchers under `_launchers/<B>/<slug>` and removes `_launchers/<A>/<slug>` (for the prime, the menu launcher too), and regenerates the hub and beam workspace files.
`moveWeftContent` is step 4's move, the one step D6's create shares (card 18): it moves each wired name's tracked content from `from` to `to`, renames `from/.lyx` to `to/.lyx` when it exists, writes the marker `to` last through an `os.Root` opened at `weftPath` (`root.WriteFile`), so it needs no `uncontainedWriteAllowlist` entry, and commits all of it in one weft commit, a commit spanning `from/<names>`, `to/<names>` and the root marker rather than an anchor-scoped `CommitRecordsPaths`, returning that commit.
A failure before the commit, or before the integration moves the branch, restores the working tree and the index through `restoreMovePathsToHead`, then `A/.lyx`, rewires `A`, removes the record through `RemoveMoveRecord` and releases the weft write lock, and returns the error with `landed` false; `Apply` returns `landed` false only once the record is removed and the pair is wired at `A` again.
When `restoreMovePathsToHead`, the `A/.lyx` rename or the rewire at `A` itself fails, `Apply` keeps the record, releases the weft write lock and returns the error, with `landed` false, naming the re-run `lyx fabric anchor <slug> --set <B>`, which recovers a not-landed record as the not-landed re-run above does, its refusal over both `A/.lyx` and `B/.lyx` included.
A failure after the commit, or after the integration moved the branch (in the wiring at `B`, the launchers or the workspace files), leaves the record standing, releases the weft write lock and returns the error with `landed` true, naming the landed re-run `lyx fabric anchor <slug> --set <B>`, which skips step 4 and does steps 5 to 7.
`anchorFailHook` stands in for either failure under test: `Apply` calls it with `afterCommit` false just before the commit or the branch move, and with `afterCommit` true just after.
`Finish` removes the record through `RemoveMoveRecord` and spawns the pair's detached push (`SpawnPairPush`).

`restoreMovePathsToHead` is a `destroy.go` executor, since `PATTERN-fabric-git` keeps every destroying primitive of `fabricengine` there: it resets the pathspecs to `HEAD` in index and working tree and deletes what `HEAD` does not track under them.
Its gates are containment of every pathspec under the weft worktree, ownership through the new `ownedMovePathspecs`, which owns a target only when it lies under one of the move's pathspecs (`A/<names>`, `B/<names>` and the marker, `.lyx` excluded), and dirtiness not applicable (`dirtinessNA`), since the record, or the failing `Apply`, proves the changes under them are the move's own.

`internal/fabricengine/doc.go`'s anchor section describes the move, the record, the recovery and the integration.

`TestAnchorChange` (tier1, hubforge) first commits `sub/.keep` in the prime's warp before it adds pairs, so `sub` is a directory git tracks in every pair's warp, and covers:
a pair moved from `.` to `sub`, with an untracked file the test put in its `.lyx` beforehand (no dirt refusal for the never-tracked `A/.lyx`): its tracked content and `.lyx`, that file included, sit under `sub`, the marker commit is `HEAD`, junctions and scratch link exist at `sub` and none at `.`, its launcher moved to `_launchers/sub/<slug>`, the workspace files name `<slug>/sub`, the record is gone after `Finish`, and the exclude lines `/_lyx` and `/.lyx` that the prime and a second pair still wired at `.` need stay while `/sub/_lyx` is added;
each refusal above in a table, each leaving the weft `HEAD`, working tree and junctions unchanged and no record, and each `validateAnchorTarget` row asserting its next step;
a failure through the hook before the move commit, and one before an integration moves the branch, each leave the pair wired at `A`, its weft clean and no record, with `landed` false;
a failure before the move commit whose restore also fails, the hook first making a non-empty `A/.lyx` so the rename back of `B/.lyx` fails, returns `landed` false with the record standing and an error naming `lyx fabric anchor <slug> --set sub`, and that re-run refuses naming both `.lyx` directories and the merge by hand;
`Close` after a `BeginAnchorChange` with no record before and no `Apply` leaves no record, and after one over a standing record keeps it;
`UpstreamRunUnfinished` reports a pair whose checked-out status reads unfinished and whose fetched upstream tip's status reads finished as finished, and one whose upstream tip's status reads unfinished as unfinished, its upstream made by a second clone that finished or started the run and pushed;
a failure through the hook after the move commit returns `landed` true and an error naming `lyx fabric anchor <slug> --set sub`, leaves the record standing, and a re-run of that `--set` finishes at `sub` with no record left;
a re-run after a kill before the commit (record standing, lock free, content half moved) and after a kill between the commit and the wiring at `B` each finish at `B` keeping a file the test put in `.lyx`;
a not-landed re-run with both `A/.lyx` and `B/.lyx` present refuses naming both and the merge by hand, touching nothing;
a re-run after a killed integration on a checkout already at `B` finishes, its fetched upstream made by a second clone (`hubforge.CloneSecond`) that moved its own copy and pushed;
a `--set` equal to the inherited anchor on a child forked after its parent's change (made with `hubforge.ReanchorPair` on the parent) is a no-op.

**ImpactSummary:** one doc file; everything else is new code, one new executor in `destroy.go` among it, that only card 17's verb and card 18's create call.
