# An anchor per beam: `.lyx-anchor` at each weft branch root

Each worktree of a hub now carries its own anchor in a `.lyx-anchor` marker at the root of its weft branch.
A pair's weft branch forks from its parent's, so git carries the marker to every pair forked from it.
Worktrees with different anchors coexist in one hub under one board, which lets a beam work in a deep subdirectory without walking down to it.
Before this change the anchor was one value per hub, kept on Bolt.

## What changes

- `lyxcwd` reads the marker for every Location it resolves: the worktree's own marker, else the legacy Bolt marker (`_board/.lyx-anchor`), else `"."`.
  An invalid own marker is an error.
  `lyxcwd` gains a constructor for another worktree's Location, and every production site that built another worktree's path from the acting worktree's `AnchorRel` now uses it.
  A scan test fails on a `lyxcwd.Location` literal in production code outside `lyxcwd` and `internal/testkit`.
- `lyx fabric clone` writes the first marker on `main-weft` instead of Bolt, and a clone of an existing hub adopts it from there, falling back to the Bolt marker for a hub not yet migrated.
- `lyx fabric reconcile` writes and commits a marker on every markerless local weft and pushes it.
  It removes the Bolt marker, and a leftover `.fabric-anchor`, only once the census over already-fetched refs finds every counted branch marked.
  It makes no network call for that.
- `lyx beam create --anchor <dir>` makes a beam whose weft content, marker, junctions, scratch link, launchers and workspace folder sit at `<dir>`.
  Without the flag the beam inherits its parent's anchor.
- `lyx fabric anchor [<slug>] [--set <dir>]` prints a worktree's anchor, or changes it: it moves the weft content and `.lyx`, commits the marker, rewires the junctions and scratch link, moves the launchers and regenerates the workspace files.
  A move record lets a killed run be recovered by running the verb again with the same `--set`.
- Checkout, weft pull, a replaying weft push, the origin reconcile's weft moves, `lyx adopt --drop-local` and `lyx beam promote` refuse or stop, each naming `lyx fabric anchor <slug> --set <anchor>`, when they would change the checked-out marker.
  The origin reconcile reports the branch stopped and goes on with the others.
- The exclude census that strips a worktree's junction lines is keyed by the anchored pattern, so re-anchoring one worktree keeps the lines another worktree at the same anchor needs.
- The hub and beam workspace files, the dobby geometry, the pair teardown, `hubforge` fixtures and the `_portals/` and `_launchers/` paths use each worktree's own anchor.
- `PATTERN-cwd-resolution` gains the per-worktree rule and the import set of `lyxcwd` (now including `internal/weftname`).
  The module docs (`lyxcwd`, `fabricengine`, `fabriccli`, `beamcli`) and the CLI help of the two verbs describe the new behavior.

Closes no issue.

## Design decisions to know

- The marker sits at the weft root, outside the anchored subtree, so moving the anchor never moves the marker.
  Only `fabricengine` writes it and git-reads it.
- A weft branch with no marker predates this change.
  Its anchor is the shallowest directory in its tree holding a wired name, else the `lyxcwd` fallback.
  Migration, adopt, the checkout comparison and the integration all use that value.
- `fabric anchor --set` refuses before touching anything when the worktree has a live or stored reed strand, a held driver lock, a live batten run, a dirty weft, an empty own marker or another machine as owner.
  For the prime it also refuses a live session anywhere in the hub, a held gate slot or a live hibernate daemon.
  While the move record stands, no dobby of the affected geometry starts a pass, and `dobbyengine.EnsureLive`, reconcile, adopt, `Add`, checkout, pull and push all refuse or report the pair.
- The verb takes the worktree branch's parent lock after waiting out in-flight dobby passes, so a child's landing queues behind the move.
- A change commits the new marker on the worktree's weft branch, which other machines share.
  On each such machine the pull, push and origin reconcile stop until the operator runs `lyx fabric anchor <slug> --set <B>`, which rebuilds the weft on the fetched upstream tip, replaying local commits under the new anchor.
  A conflict stops it with the weft unchanged and names the conflicting commit.
- Landing needs no mechanism to keep the parent's marker: the weft is never a merge participant.

## Check by hand

- Update every machine to the new binary, and run `lyx fabric reconcile` first on each, before reconciling a subpath hub.
  A machine on the old binary reads an absent Bolt marker as `"."` and wires a second junction set at the repo root once another machine has removed the Bolt marker.
- A local-only (`--local`, never pushed) markerless pair is outside the census and resolves to `"."` once the Bolt marker is gone, until its machine's reconcile writes its marker.
  The census also cannot see a branch this machine never fetched.
- Origin reconcile can move a weft past the D10 stop (finding `origin-reconcile-moves-weft-past-d10` in `ReconcileOrigin`): check that hub-wide and run-scoped runs both leave a weft with a changed marker unmoved.
- The integration's temporary index has no `gitexec` seam (finding `card14-temp-index-no-gitexec-seam`, in the integration code in `internal/fabricengine`): check how it is built and that tests can fake it.
- The fixture test `TestDobbyRuns_PairAtItsOwnAnchor/InvalidMarkerRefusesTheRunsAndLeavesThePairOutOfTheWatch` in `internal/beamcli` failed once on the dobby run listing and was fixed in `cf28d7203`: check that a pair with an invalid marker is refused and not silently dropped from the listing.
- The tmux-driven `lyx fabric anchor` flow (a real dobby ended and restarted, a kill mid-move and the re-run, the prime's refusals) and the two-machine pull/push stop and integration were covered by tests but not run against a real hub: run them once by hand.
