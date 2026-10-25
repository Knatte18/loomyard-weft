# Decision record — umbrella-anchor: an anchor per beam

## Goal

Each worktree of a hub carries its own anchor in a `.lyx-anchor` marker at the root of its weft branch, inherited through git by every pair forked from it.
`lyx beam create --anchor <dir>` makes a beam at a new anchor, and `lyx fabric anchor` changes an existing worktree's anchor.
Every path lyx builds for a worktree uses that worktree's own anchor, so worktrees with different anchors coexist in one hub, under one board.

## Scope

In:

- The marker at `<hub>/<worktree>-weft/.lyx-anchor`, read by `lyxcwd` for every Location it resolves.
- The legacy marker on Bolt (`_board/.lyx-anchor`): read during migration, then removed by `lyx fabric reconcile`.
- `lyx fabric clone` writing and adopting the marker on the prime's weft branch, `main-weft`, instead of Bolt.
- `lyx fabric reconcile` migrating markerless weft branches.
- Every builder of another worktree's anchored path or weft pathspec.
- `lyx beam create --anchor <dir>`.
- `lyx fabric anchor [<slug>] [--set <dir>]`, which prints or changes a worktree's anchor.
- `lyx fabric checkout`, weft pulls and `lyx beam promote` refusing an anchor mismatch.

Out:

- A separate hub per subdirectory (`fabric clone --subpath` into a new weft repo).
- An anchor index on Bolt.
- `--anchor` on `lyx fabric add` or `lyx beam promote`; `lyx fabric anchor` changes either kind of pair after the fact.
- Moving or rewriting a worktree's `.vscode/` config; the next `lyx ide` open writes it at the new anchor.

## Decisions

### D1 — The marker lives at each weft branch's root

The anchor of worktree `<name>` is the content of `<hub>/<name>-weft/.lyx-anchor`, a tracked file at the root of that worktree's checked-out weft branch, holding one worktree-relative path validated by `lyxcwd.ValidateAnchorRel`.
The marker sits at the weft root, outside the anchored subtree, so moving the anchor never moves the marker.
`lyxcwd` reads it from the working tree, through the weft sibling name `weftname.SiblingPath` declares; `lyxcwd`'s import set grows by `internal/weftname`, a standard-library-only leaf, so the `-weft` suffix keeps its one declarer.
Only `fabricengine` writes the marker, and only `fabricengine` reads it from git objects (a fetched ref, a target branch, history).

Rationale: the operator's design; lyx finds the marker from the worktree's name alone, and git carries inheritance.

### D2 — Read order and fallback

`lyxcwd`'s anchor read for worktree `<name>`, in order:

1. the worktree's own marker;
2. else the legacy Bolt marker `<hub>/_board/.lyx-anchor`, with today's stale-spelling guard (`ErrStaleAnchorMarker`) applying to this read only;
3. else `"."`.

An own marker that fails validation is an error, as today's Bolt read is.
An own marker that is empty after trimming counts as absent for this read, as today.
Every `fabricengine` path that acts on a marker, D5 to D10, refuses an own marker that is present but empty, naming the file and the way forward, before touching anything.

Rationale: a hub migrates without a flag day, fixtures, mid-clone states and plain git repos keep resolving to `"."`, and a truncated marker never turns into a rewire at the repo root.

### D3 — One way to get another worktree's Location

`lyxcwd` gains a constructor for the Location of a named worktree of a hub that reads that worktree's anchor per D2 and applies no cwd gate; `ResolveWorktree` reads the resolved worktree's own marker the same way.
Every production site that builds a path or a weft pathspec for a worktree other than the resolved one gets that worktree's Location from `lyxcwd`; the `(l, slug)` path builders take the target worktree's Location instead of the acting one's `AnchorRel`.
A scan test fails on a `lyxcwd.Location` composite literal in production code outside `lyxcwd`, with `internal/testkit` exempt; a slug-built path joined with another Location's `AnchorRel` is held by review.
A consumer acting on a pair whose weft is gone uses no anchored path of that pair, since the fallback cannot know its anchor.
The retired-portal sweep keys on the prime's anchor, which equals the hub anchor every retired portal was made under until the prime is re-anchored; portals left under an old prime anchor are not swept.

Rationale: copying the acting Location's `AnchorRel` is the one-anchor assumption the entry names; the constructor removes the need, and the scan stops the literal form of it from coming back.

### D4 — New hubs record the anchor on `main-weft`

`fabric clone` on a fresh bind writes the marker at the root of the prime's weft worktree and commits it on `main-weft`, and writes no Bolt marker.
`fabric clone` of an existing hub adopts the marker from `main-weft`, falling back to the Bolt legacy marker for a hub not yet migrated.
A `--subpath` that disagrees with the adopted value is refused, as today.
The fresh-bind test ("a weft carrying neither `.lyx-warp` nor `.lyx-anchor`") counts a marker on `main-weft` or on Bolt.

Rationale: every later weft branch forks from `main-weft` or from a branch that did, so every worktree inherits a marker.

### D5 — Forks inherit; a markerless branch's anchor is where its content is

A pair's weft branch forks from its parent's, so it carries the parent's marker and its content at the parent's anchor.
`Add` takes the new pair's anchor from the fork point, the parent's Location, or from `--anchor` (D6), never from a read of the not-yet-written weft; every path it builds for the new pair, its origin record included, uses that anchor.
`createWeftWorktreeDroppingRuns` scopes the run-records root under the start point's anchor, the parent's.

A weft branch with no marker predates this task, so its content sits at one anchor already.
Its anchor, wherever one must be written or compared (D8 migration, an adopt, D10's checkout), is the shallowest directory in its tree holding a wired name, else D2's resolution.
An adopt of such a branch writes and commits that value.

Rationale: with `--parent` the acting worktree is not the fork point, and a parent re-anchored after a child forked no longer says where the child's content is.

### D6 — `lyx beam create --anchor <dir>`

`--anchor` takes a path relative to the warp repo root, validated as D7 validates `B`, in the commit the beam forks from.
Without it the beam inherits its parent's anchor through the fork (D5).
With it, `Add` forks the pair, then, before any write at the new anchor, moves the weft content to `<dir>` and commits the new marker with D7's step-3 move, and wires the junctions at `<dir>`.
On a re-run whose pair is present, an `--anchor` equal to the pair's anchor is a skip, and a different one is refused, naming `lyx fabric anchor <slug> --set <dir>`.
The flag reaches only the pair being created.

Rationale: the operator's design; one move step serves create and change.

### D7 — `lyx fabric anchor [<slug>] [--set <dir>]`

Without `--set` the verb prints the anchor of the worktree `<slug>`, the acting worktree when `<slug>` is omitted.
With `--set <dir>` it changes that worktree's anchor from `A` to `B` on this machine.
It runs from the prime or any pair of the hub; `<slug>` names the prime by its worktree name or any pair present on this machine.

Validation of `B`, before anything is touched: it passes `ValidateAnchorRel`, is a directory git tracks in the worktree's warp checkout, has no segment that is a link or a wired, `.lyx`, `.scratch` or hub-reserved name, and holds no real wired name (the warp-pristine invariant).

Refusals, each naming the way forward:

- The board names another machine as the worktree's owner.
- The worktree's own marker is empty (D2).
- The worktree's reed state holds a strand, live or stored; way forward: `lyx reed down` in that worktree, which ends the session and clears the strands.
- A run driver lock in the worktree is held, a registered batten run of the worktree is live, or its run status names an unfinished run.
- On Windows, where `runproc.End` signals nothing, a live dobby the verb would end; the refusal names its pid.
- For the prime only, since the hub's batten records, gate slots, orch and daemon state sit under the prime's anchor: any live reed session or stored strand in the hub, any live batten run, a held gate slot, or a live hibernate or creel daemon of the hub.

Steps, in order:

1. End the worktree's dobby process alone through `runproc.End`, leaving any pass it spawned to finish on its own; for the prime, every dobby this machine runs, since each dobby's geometry holds the prime's anchor.
   Then run the refusals above; a refusal restarts the dobbies this step ended.
2. Remove the worktree's junctions and scratch link at `A` through the destruction chokepoint.
3. Close the verb's own log sink under `A/.lyx`.
   In the weft, move each wired name's tracked content from `A` to `B` and rename the never-tracked `A/.lyx` to `B/.lyx`; write the marker `B` last, and commit all of it in one weft commit carrying the trailer `Lyx-Anchor-From: A`.
   The warp's `<anchor>/.lyx` is the junction into that directory, so this covers it.
   A failure before the commit restores the working tree, the index, `A/.lyx` and the marker, and rewires `A`.
4. Wire the junctions and the scratch link at `B`, and reopen the log sink there.
5. Write the worktree's launchers under `_launchers/<B>/<slug>` and remove `_launchers/<A>/<slug>`; for the prime, the menu launcher moves the same way.
6. Regenerate the hub workspace file and every beam workspace file.
7. Spawn the pair's push, and start again exactly the dobbies step 1 ended.

Recovery is a re-run of the verb, from any worktree of the hub; `lyx fabric reconcile` does not repair it, since it treats a pair with no `_lyx` at its anchor as unmanaged.
A re-run first undoes an uncommitted move a killed run left in the weft.
It then runs steps 2 to 7 idempotently, with `A` taken from the `Lyx-Anchor-From` trailer of the commit that last changed the marker; with no such trailer, as for an anchor inherited through a fork, nothing at another anchor is cleaned.
A `--set` equal to a settled worktree's anchor is a no-op.

The verb holds no lock against a session or run started by hand while it runs; the operator does not start one in the worktree meanwhile.
It skips no dirtiness check on the weft, and it never acts on another machine's worktree.
Child worktrees keep their own markers and anchors; the verb neither refuses on them nor touches them.

Rationale: the operator's design; every agent and daemon in the worktree has its cwd or state at the anchor, so the move needs them gone.

### D8 — Migration by `lyx fabric reconcile`

For each managed pair whose weft worktree is present and has no marker, reconcile writes and commits D5's markerless value; the prime's weft is migrated first.
Reconcile removes the Bolt marker in a board commit once the prime's weft and every local pair's weft carry a committed marker, and every weft branch whose warp sibling is on origin carries one on its already-fetched remote-tracking ref.
The census makes no network call: a branch whose ref is not fetched keeps the Bolt marker, and so does a marker commit not yet pushed, so removal lands on a later pass.
A weft branch on origin with no warp sibling there is an orphan and is not counted; an adopt of it gets D5's value.
`refuseEmptyAnchorMarker` becomes per pair: a pair whose own marker is present but empty is reported as failed with the way forward, and reconcile goes on with the other pairs.
`lyx fabric unwire` leaves the marker, so a later reconcile rewires at the same anchor.

Rationale: no marker stays on Bolt, as the entry requires; removal waits until no live branch needs the fallback, and no lyx command waits on the network (note `fabric-remote-async`).

### D9 — The exclude census is keyed by anchored pattern

The git-exclude file is shared by every worktree of the hub.
Removing a worktree's junction lines strips the line `/<anchor>/<name>` only when no other worktree has a junction at that same anchored path, each worktree's link checked at its own anchor.

Rationale: today's census checks every sibling at the acting worktree's anchor, which with two anchors strips lines another worktree still needs.

### D10 — Branch switches, pulls and promote across anchors

`lyx fabric checkout` refuses, before switching either side, a target weft branch whose anchor, its marker or D5's markerless value, differs from the worktree's; way forward: `lyx fabric anchor --set <target anchor>` first.
A forked new weft branch inherits the current marker and is never refused.
A weft pull whose incoming tip carries a marker different from the pair's refuses that pair before updating either side; way forward: `lyx fabric remove <slug>` and adopt the pair again, which wires it at the incoming anchor.
`lyx beam promote` refuses a task whose anchor differs from its parent's, since resetting the records to the parent's tree carries the parent's marker; way forward: `lyx fabric anchor <slug> --set <parent's anchor>`.

Rationale: these are the paths that change a pair's checked-out marker; each refuses before changing anything, so no half-rewired pair needs a recovery of its own.

## Constraints

- The weft is never a merge participant (`fabricengine/doc.go`, "The merge surface"), so landing never touches the parent's weft or its marker; the entry's "kept out of the merge as raddle is" needs no mechanism.
- `lyxcwd` stays the sole cwd and worktree resolver (`PATTERN-cwd-resolution`), with the import set of D1.
- Git on warp or weft runs only in `fabricengine`, and removals go through its destruction chokepoint (`PATTERN-fabric-git`); the D7 move, the D8 marker commits and the D10 refusals are `fabricengine` work.
  Session, run and dobby checks run in the CLI layer above it, as `pairteardown` sits above fabric.
- Dobbies end only through `runproc.End` (`PATTERN-run-process-owner`).
- Every refusal names its way forward (`PATTERN-refusal-way-forward`).
- `_lyx` and `.lyx` keep the same subpath under one anchor (`PATTERN-durable-vs-ephemeral-state`); the move carries both.
- The repo-wide wired name set in `fabric.yaml` stays one set of names; only where they are anchored varies.
- Remote work stays async (note `fabric-remote-async`): D8's census reads fetched refs only.
- Test hubs come from `hubforge`, which builds them through `CloneHub`, so fixtures carry the marker of D4.
- Docs land with the code: `lyxcwd/anchor.go` and package docs, `fabricengine/doc.go`'s anchor section, `fabriccli` and `beamcli` docs, and the CLI help of the two verbs.

## Auto-mode assumptions

- Launchers keep their anchor-mirrored layout, keyed by each worktree's own anchor.

## Open risks

- A machine still on the old binary reads an absent Bolt marker as `"."`; on a subpath-anchored hub, once another machine has removed the Bolt marker, its `lyx fabric reconcile` wires a second junction set at the repo root.
  The operator updates every machine before reconciling a subpath hub on the new binary.
- A local-only (`--local`, never pushed) markerless pair is outside the census, and after the Bolt marker is gone resolves to `"."` until its machine's reconcile writes its marker.

## Acceptance criteria

- Two pairs in one hub with different markers each resolve their own `AnchorRel`, and the cwd gate refuses each at the other's anchor.
- A markerless weft resolves to the Bolt legacy value while Bolt has one, and to `"."` when neither exists; an invalid own marker is an error.
- An empty own marker is refused by checkout, pull, `Add` from it, the anchor verb and reconcile, each touching nothing.
- A fresh `fabric clone --subpath backend` writes `backend` on `main-weft` and no Bolt marker; cloning that hub again adopts `backend` and refuses a different `--subpath`.
- A task forked from a beam at anchor `X`, with and without `--parent`, has marker `X`, junctions and origin record at `X`, and no run records of its parent on disk.
- A markerless task whose parent was re-anchored after it forked migrates, and adopts, to the anchor its content sits at.
- `lyx beam create --anchor Y` makes a beam whose marker, weft content, junctions, scratch link, launchers and workspace folder sit at `Y`; it refuses an invalid, absent or link-crossing `Y`, and on a present pair refuses a different anchor and skips an equal one.
- `lyx fabric anchor <slug> --set B` moves tracked content and `.lyx`, commits the marker with its trailer, wires junctions and the scratch link at `B`, leaves none at `A`, moves the launchers, regenerates the workspace files and restarts the dobby it ended, while a pass that dobby had in flight finishes.
- `lyx fabric anchor` refuses a `B` under a junction, a stored or live reed strand, a held driver lock, a live batten run, an unfinished run, another machine's worktree and a `B` holding a real wired name, each with nothing touched and any ended dobby restarted; for the prime it also refuses a live session anywhere in the hub, a held gate slot and a live hibernate or creel daemon.
- A failure before the marker commit leaves the worktree wired at `A` with a clean weft; a re-run after a kill at any step finishes at `B`; a `--set` equal to an inherited anchor is a no-op.
- A child pair forked before its parent's anchor change keeps its own anchor, still runs, and lands into the parent, whose marker stays unchanged.
- Unwiring or re-anchoring one worktree keeps the exclude lines another worktree at the same anchor needs.
- Reconcile writes the right marker into each markerless local weft, removes the Bolt marker only when the census over fetched refs finds every counted branch marked, makes no network call for it, and fails only the pair with an empty marker.
- `fabric checkout` to a branch with a different anchor and a weft pull bringing a different marker are refused with both sides unchanged.
- `beam promote` refuses a task whose anchor differs from its parent's.
- The hub and beam workspace files, the dobby geometry and the pair teardown use each worktree's own anchor in a hub with two anchors.
- The scan fails on a production `lyxcwd.Location` literal outside `lyxcwd` and `internal/testkit`.

## PATTERN changes

- `PATTERN-cwd-resolution` — changes: Resolving a cwd, worktree root or anchor: only `internal/lyxcwd`, a worktree's anchor from its own weft-root `.lyx-anchor`, never another Location's; only `fabricengine` writes it or reads it from git; an engine is handed absolute paths and never imports it; hub-level containers and workspace files stay hub-reachable only. (test)

## Notes for the plan writer

- The background file `pattern/PATTERN-cwd-resolution.md` needs the import set of D1 and a line on D3's constructor and scan; `leaf_enforcement_test.go` pins the import cap.
- The D7 commit spans `A/<names>`, `B/<names>` and the root marker, so it is not one of today's anchor-scoped `CommitRecordsPaths` calls.
- `isRawWarpWorktree` (`reconcile.go`) is why reconcile does not recover an interrupted D7.
