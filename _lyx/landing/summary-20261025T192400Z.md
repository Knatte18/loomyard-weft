# Give each worktree its own anchor in a weft-root .lyx-anchor marker

## What changes

Until now a hub had one anchor, set once by `lyx fabric clone --subpath` and kept in a marker on Bolt.
Every worktree used it, so all beams shared one subdirectory for `_lyx`, `.lyx/`, junctions and workspace roots.

Now each worktree carries its own anchor in `<hub>/<worktree>-weft/.lyx-anchor`, a tracked file at the root of its weft branch.
A pair's weft branch forks from its parent's, so the pair inherits the parent's anchor through git.
Worktrees with different anchors coexist in one hub under one board.

- `lyx beam create --anchor <dir>` makes a beam at a new anchor.
- `lyx fabric anchor [<slug>] [--set <dir>]` prints a worktree's anchor, or moves its tracked content, `.lyx`, junctions, scratch link, launchers and workspace files to a new one.
- `lyxcwd` reads the worktree's own marker, then the legacy Bolt marker, then `"."`.
  A new constructor returns the Location of any named worktree, and every production site that built another worktree's anchored path or weft pathspec now uses it instead of the acting worktree's `AnchorRel`.
- `lyx fabric clone` writes the marker on `main-weft` instead of Bolt, and adopts it from there on an existing hub, falling back to the Bolt marker.
- `lyx fabric reconcile` writes a marker into each markerless weft, then removes the Bolt marker (and any leftover `.fabric-anchor`) once every counted branch carries one on its fetched ref.
  It makes no network call for this.
- The git-exclude census is keyed by anchored pattern, so unwiring one worktree no longer strips lines another worktree at a different anchor still needs.

## Design decisions to review

- Only `fabricengine` writes the marker, and only it reads it from git objects.
  `lyxcwd` reads the working tree and imports `internal/weftname`, a standard-library-only leaf.
- A scan test fails on a production `lyxcwd.Location` literal outside `lyxcwd` and `internal/testkit`.
  A slug-built path joined with another Location's `AnchorRel` is held by review only.
- A weft branch with no marker predates this change, so its anchor is where its content sits: the shallowest directory holding a wired name.
  Migration and adopt write that value, which keeps a parent re-anchored after its child forked from rewiring the child.
- `--set` runs under a move lock and a move record in the weft's lock directory.
  It refuses a live reed strand, held driver lock, live batten run, unfinished run, dirty weft, another machine's worktree, and a `B` under a link or holding a real wired name.
  It ends the worktree's dobby sparing pushes and board syncs, holds the branch's parent lock to the end, and recovers from a kill by re-running with the same `--set`.
  While the record stands, `dobbyengine.EnsureLive`, reconcile, adopt, `Add` as a fork point, checkout and the weft writers refuse or stop, each naming the way forward.
- A marker change committed on a shared weft branch stops other machines' weft pulls, replaying pushes and origin-reconcile weft moves, and `lyx fabric checkout` and `lyx adopt --drop-local` refuse across an anchor mismatch.
  The operator runs `lyx fabric anchor <slug> --set <anchor>` there, which integrates by replaying unpushed local commits onto the upstream tip re-rooted to the new anchor.
  `lyx beam promote` refuses a task whose anchor differs from its parent's.
  One `fabricengine` function holds the marker comparison for all of these.
- The new `PATTERN-cwd-resolution` entry says anchors come only from `lyxcwd` and the worktree's own weft-root marker.
  Its text was shortened to fit the overview line cap.

## To check by hand

- Integration of a weft across another machine's anchor change builds its result in a temporary index, and that code has no `gitexec` seam.
  Check its tests cover the failure paths, and that it is acceptable as is.
- Multi-machine behavior was not run: the pull, push, origin-reconcile stops and the integrating `--set` on a second machine.
- The kill-and-re-run recovery of `lyx fabric anchor --set` and of `lyx beam create --anchor` was exercised by tests, not on a real hub with live dobbies.
- Windows: the `--set` refusal for a live dobby it cannot end (`runproc.End` signals nothing there) was not run.

## Operator notes and open risks

- Update every machine to the new binary, then run `lyx fabric reconcile` first on each, before any other lyx command there.
  A machine on the old binary reads an absent Bolt marker as `"."`, and on a subpath hub its reconcile would wire a second junction set at the repo root.
- A local-only markerless pair, and a branch no machine has fetched, is outside the census.
  After the Bolt marker is removed it resolves to `"."` until its own machine's reconcile writes its marker.
- Portals left under an old prime anchor after the prime is re-anchored are not swept.
- A worktree checked out on several machines keeps its records local on the others until the integrating `--set` runs there.
