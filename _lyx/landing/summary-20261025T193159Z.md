# An anchor per worktree: .lyx-anchor on each weft branch

Each worktree of a hub now carries its own anchor in a `.lyx-anchor` file at the root of its weft branch.
A pair forked from a worktree inherits the marker through git, and worktrees with different anchors coexist in one hub under one board.
Before this, the anchor was one value per hub, kept on Bolt, so every beam shared it.

## What changes

- `lyxcwd` reads the anchor of the worktree's own marker at `<hub>/<worktree>-weft/.lyx-anchor`.
  The fallback order is the own marker, then the legacy Bolt marker (`_board/.lyx-anchor`), then `"."`.
  An invalid own marker is an error, and an empty one counts as absent for this read.
  `lyxcwd` gains a constructor for the Location of any named worktree, and every production site that built another worktree's path from the acting worktree's `AnchorRel` now uses it.
  A scan test fails on a `lyxcwd.Location` literal in production code outside `lyxcwd` and `internal/testkit`.
- `lyx fabric clone` writes the marker on `main-weft` instead of Bolt, and adopts it from there on a later clone, falling back to the Bolt marker for an unmigrated hub.
- `lyx fabric reconcile` writes the marker into each markerless local weft and spawns its push.
  It removes the Bolt marker, and a leftover `.fabric-anchor`, only once the prime's weft, every local pair's weft and every fetched remote-tracking weft branch with a warp sibling on origin carry a committed marker.
  The census makes no network call.
  A pair with an empty marker or a standing move record is reported as failed, and reconcile goes on with the others.
- `lyx beam create --anchor <dir>` makes a beam at a new anchor.
  Without it the beam inherits its parent's.
- `lyx fabric anchor [<slug>] [--set <dir>]` prints or changes a worktree's anchor.
  A change ends the worktree's dobby, moves the tracked weft content and `.lyx` to the new anchor, commits the marker, rewires the junctions and scratch link, moves the launchers and regenerates the workspace files.
  A kill mid-change is recovered by re-running the same `--set`, driven by a move record beside the weft's lock directory.
- `lyx fabric checkout`, weft pull, a replaying weft push, the origin reconcile's weft moves, `lyx adopt --drop-local` and `lyx beam promote` refuse or stop, before changing anything, when the move would change the checked-out marker.
  Each names `lyx fabric anchor <slug> --set <anchor>` as the way forward.
  On another machine the same `--set` integrates: it rebuilds the weft on its fetched upstream tip and re-roots the local commits at the new anchor.
- The git-exclude census is keyed by anchored pattern, so removing one worktree's junction lines keeps the lines another worktree at the same anchor needs.
- Closes no issue; the board entry lists none.

## Design decisions to check

- The marker sits at the weft root, outside the anchored subtree, so a move never moves it.
  Only `fabricengine` writes it or reads it from git objects.
  `lyxcwd` now imports `internal/weftname`, a standard-library-only leaf, so the `-weft` suffix keeps one declarer.
- A weft branch with no marker is anchored where its content sits: the shallowest directory in its tree holding a wired name, else the D2 fallback.
  Migration, adopt, checkout, pull, push and the integration all compare by that value.
- The weft stays out of the landing merge, so the parent keeps its own marker with no extra mechanism.
- While a move record stands, `dobbyengine.EnsureLive` starts none of the affected dobbies, and no live one starts a pass.
  Reconcile, adopt, `Add` as a fork point, checkout, and weft pull and push refuse the pair.
- `fabricengine` holds one marker comparison function, called by pull, push, the origin reconcile and the drop-local reset.
- `pattern/PATTERN-cwd-resolution.md` and the `PATTERN.md` overview line are updated; the package docs and the CLI help of the two verbs describe the behaviour.

## Check by hand

- Plan-review finding: the temporary-index commit in `internal/fabricengine` (the move commit built with `GIT_INDEX_FILE`) goes through a new `gitexec` env-carrying helper rather than an existing seam.
  Check that its failure reporting matches `gitexec.Run` and that the move commit staged exactly `A/<names>`, `B/<names>` and the marker.
- Discussion-time risk, open by design: a machine on the old binary reads an absent Bolt marker as `"."`.
  On a subpath hub, update every machine, and run `lyx fabric reconcile` first on each, before any other lyx command there.
- A local-only (`--local`, never pushed) markerless pair resolves to `"."` once Bolt's marker is gone, until its machine's reconcile writes its marker.
- A worktree checked out on several machines: until another machine runs the integrating `lyx fabric anchor <slug> --set <B>`, its weft pushes and origin-reconcile moves of that weft stop.
- Not run: a real two-machine anchor change of the prime, and `lyx fabric anchor` on Windows, where `runproc.End` signals nothing.
- A verify run failed once on `internal/beamcli` `TestDobbyRuns_PairAtItsOwnAnchor/InvalidMarkerRefusesTheRunsAndLeavesThePairOutOfTheWatch`; a follow-up commit makes the dobby run listing refuse a pair with an invalid marker.
  Check that the refusal, not an omission, is the intended behaviour.
