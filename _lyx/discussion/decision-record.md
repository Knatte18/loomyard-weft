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
- `lyx fabric checkout`, weft pulls and pushes, the origin reconcile's weft moves, `lyx adopt --drop-local` and `lyx beam promote` refusing or stopping on an anchor mismatch, and the integration that carries a weft across another machine's anchor change.

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
Clone's stale-spelling guard on a leftover Bolt `.fabric-anchor` runs only on the Bolt fallback path, when `main-weft` carries no marker.
The fresh-bind test ("a weft carrying neither `.lyx-warp` nor `.lyx-anchor`") counts a marker on `main-weft` or on Bolt:
`probeWeftBinding` also reads the root of `main-weft`, by name, beside the weft's `HEAD`, and the bootstrap guard's evidence and message count a marker on either branch.

Rationale: every later weft branch forks from `main-weft` or from a branch that did, so every worktree inherits a marker.

### D5 — Forks inherit; a markerless branch's anchor is where its content is

A pair's weft branch forks from its parent's, so it carries the parent's marker and its content at the parent's anchor.
`Add` takes the new pair's anchor from the fork point, the parent's Location, or from `--anchor` (D6), never from a read of the not-yet-written weft; every path it builds for the new pair, its origin record included, uses that anchor.
`createWeftWorktreeDroppingRuns` scopes the run-records root under the start point's anchor, the parent's.
`Add`, whether from `lyx fabric add`, `lyx beam create` or adopt's `Add`, refuses a fork point whose weft has a standing D7 move record, before touching anything, naming a change in progress while the record's move lock is held and `lyx fabric anchor <slug> --set <B>`, naming the record's `B`, once it is free;
a kill mid-move can leave the fork point's working-tree marker at `B` over a weft branch whose content and marker sit at `A`.

A weft branch with no marker predates this task, so its content sits at one anchor already.
Its anchor, wherever one must be written or compared (D8 migration, an adopt, D10's checkout, D10's pull, push, origin reconcile and drop-local comparison, the integration's source anchor, `U`'s marker, and a replayed local commit's parent tree), is the shallowest directory in its tree holding a wired name, else D2's resolution.
An adopt of such a branch writes and commits that value.

Rationale: with `--parent` the acting worktree is not the fork point, and a parent re-anchored after a child forked no longer says where the child's content is.

### D6 — `lyx beam create --anchor <dir>`

`--anchor` takes a path relative to the warp repo root, validated as D7 validates `B`, in the commit the beam forks from.
Without it the beam inherits its parent's anchor through the fork (D5).
With it, `Add` forks the pair, then, before any write at the new anchor, takes D7's move lock and writes its move record, moves the weft content to `<dir>` and commits the new marker with D7's step-4 move, wires the junctions at `<dir>`, and removes the record once the remaining create steps are done.
On a re-run whose pair is present and has a standing move record, `Add` first recovers as D7's re-run does: it undoes an uncommitted move and moves again, or, after the commit, goes on.
Then an `--anchor` equal to the pair's anchor skips the move but runs the remaining create steps, the wiring, scratch link, launchers and workspace files, idempotently, and a different one is refused, naming `lyx fabric anchor <slug> --set <dir>`.
The flag reaches only the pair being created.

Rationale: the operator's design; one move step serves create and change.

### D7 — `lyx fabric anchor [<slug>] [--set <dir>]`

Without `--set` the verb prints the anchor of the worktree `<slug>`, the acting worktree when `<slug>` is omitted.
With `--set <dir>` it gives that worktree's checkout on this machine the anchor `B`, from its checked-out anchor `A`.
It runs from the prime or any pair of the hub; `<slug>` names the prime by its worktree name or any pair present on this machine.
A `--set` integrates when the weft's fetched upstream has commits the weft's tip lacks and the two tips carry different markers: its step 4 runs D10's integration onto `B` in place of the move.
A `--set` that does not integrate and differs from `A` moves.
With no move record standing, a `--set` that neither integrates nor differs from `A` is a no-op, as on a child forked after its parent's change, whose marker is already its parent's `B`.

Validation of `B`, before anything is touched: it passes `ValidateAnchorRel`, is a directory git tracks in the worktree's warp checkout, has no segment that is a link or a wired, `.lyx`, `.scratch` or hub-reserved name, and holds no real wired name (the warp-pristine invariant).
A `B` the warp checkout lacks while the warp's fetched upstream tracks it, as on a machine whose warp predates the commit that added `B`, is refused naming the way forward of updating the warp checkout by hand to a commit that tracks `B`, since the weft pull that would update it stops (D10).

Refusals, all run before anything is touched, each naming the way forward:

- The board names another machine as the worktree's owner, while `B` differs from the marker at the fetched upstream tip or the weft has no upstream; taking up the upstream's anchor is open to every machine that checks the worktree out.
- The worktree's own marker is empty (D2).
- With no move record standing, the worktree's weft has no committed marker; way forward: `lyx fabric reconcile`, which writes D5's markerless value, so `A` is always a committed marker.
- With no move record standing, the weft is dirty: an uncommitted or untracked path under `A/<names>`, bar the never-tracked `A/.lyx`, or an uncommitted marker; way forward: `lyx fabric commit` or removing the files for a path under `A/<names>`, and restoring the committed marker (`git -C <weft> restore .lyx-anchor`) for the marker, which `lyx fabric commit` does not stage.
- The worktree's reed state holds a strand, live or stored; way forward: `lyx reed down` in that worktree, which ends the session and clears the strands.
- A run driver lock in the worktree is held, a registered batten run of the worktree is live, or its run status names an unfinished run.
- The move lock in the weft's lock directory is held: another run of the verb on this worktree is live; the refusal names its pid, read from the move record, or names the run as starting when the record's pid and start time are not the live holder's, and the way forward is to wait for it.
- On Windows, where `runproc.End` signals nothing, a live dobby the verb would end; the refusal names its pid.
- For the prime only, since the hub's batten records, gate slots, orch and hibernate daemon state sit under the prime's anchor: any live reed session or stored strand in the hub, any live batten run, a held gate slot, or a live hibernate daemon of the hub, naming `lyx hibernate cancel`.
  The creel daemon is not refused: its state sits under `<hub>/_board/.lyx/creel`, outside every anchor, and it runs from the hub directory.

Steps, in order:

1. Take the move lock, a file lock in the weft's lock directory `.weft/`, held by this process until it exits.
   Write the move record beside it, a file git excludes and no move touches, holding `A`, `B` and whether the run moves or integrates; an integration adds its upstream tip `U` at step 4.
   Right after taking the move lock, every holder, a run or re-run of the verb or D6's `Add`, writes its own pid and start time into the record, which the move-lock refusal reads.
   While the record stands, every dobby whose geometry the worktree's anchor reaches, the worktree's own and for the prime every dobby of this machine, starts no pass and no task;
   `dobbyengine.EnsureLive` starts none of those dobbies and reports each, naming a change in progress while the move lock is held and `lyx fabric anchor <slug> --set <B>` once it is free;
   `lyx fabric reconcile` refuses the pair (D8), `lyx adopt` refuses as below, `Add` refuses the worktree as a fork point (D5), a weft pull of the worktree, or a weft push of it that must replay onto its upstream, refuses with the same naming, and the origin reconcile leaves the worktree's weft unmoved and reports it stopped with the same naming (D10).
   Then the verb waits until no dobby step 2 will end has a pass in flight: the worktree's own dobby, and for the prime every dobby of this machine, whatever run it drives.
   A pass is in flight while its driver lock is held or while its dobby has a live child process, read from the process table, other than a spared push or board sync, so a pass spawned just before the dobby saw the record and not yet holding its driver lock counts.
   Once none is in flight, the verb takes the parent lock of the worktree's branch through `internal/parentlock` as its own owner, this process's pid and start time, queuing behind a holder so a child's landing into the worktree finishes first, and holds it until step 8.
   A child's Finalize that reaches the lock meanwhile, such as a loom run's detached loop that outlives the passes, queues behind the verb and commits no parent records and merges nothing into the worktree while the verb runs.
   The lock is taken only after the pass wait, so no pass the verb waits on is itself queued behind the verb.
   It prints what it waits on.
2. End the worktree's dobby, when it is a beam, through `runproc.EndSparing` sparing the push and sync roles; it ends the dobby's descendants with it, which step 1's wait has left with no pass among them.
   For the prime, end every dobby this machine runs the same way, since each dobby's geometry holds the prime's anchor.
   A push or board sync outside the ended dobbies' process trees, such as one registered under `runproc.HubOwner` by another process, is spared; one a dobby spawned itself is ended with it, and the next push of its tree carries its commits.
3. Remove the worktree's junctions and scratch link at `A` through the destruction chokepoint.
4. Close the verb's own log sink under `A/.lyx`, and take the weft write lock (`.weft/weft.write.lock`), held across the whole step, its commit included, so no other weft commit lands between the moves.
   To move, in the weft, move each wired name's tracked content from `A` to `B` and rename the never-tracked `A/.lyx` to `B/.lyx`; write the marker `B` last, and commit all of it in one weft commit.
   To integrate, write into the record the fetched upstream tip `U` the integration rebuilds on, then run D10's integration onto `B` and rename `A/.lyx` to `B/.lyx`.
   The warp's `<anchor>/.lyx` is the junction into that directory, so this covers it.
   A failure before the commit, or before the integration moves the branch, restores the working tree, the index, `A/.lyx` and the marker, rewires `A`, removes the move record, restarts the dobbies step 2 ended and releases the weft write lock and the parent lock.
5. Wire the junctions and the scratch link at `B`, and reopen the log sink there.
6. Write the worktree's launchers under `_launchers/<B>/<slug>` and remove `_launchers/<A>/<slug>`; for the prime, the menu launcher moves the same way.
7. Regenerate the hub workspace file and every beam workspace file.
8. Remove the move record, then spawn the pair's push, which a standing record would stop when it must replay, and start the worktree's dobby when it is a beam this machine owns and its lock reads free, as `dobbyengine.EnsureLive` starts one; for the prime, run `dobbyengine.EnsureLive`, which starts every dobby of this machine whose lock reads free.
   Release the parent lock; the move lock goes with the process, and a killed run's parent lock is freed with it, since its owner process is gone.

Recovery from a kill is a re-run of the verb with the same `--set`, from any worktree of the hub.
A standing move record with its move lock free is the evidence that a killed run left a change in flight, and the record's `A`, `B` and kind govern the re-run; a re-run naming another `--set` while it stands is refused, naming the record's `B`.
`lyx fabric reconcile` refuses such a pair rather than repair it (D8): its wiring repair wires at whatever anchor the checked-out marker names and moves neither content nor launchers.
While the record stands, the verb's log sink opens in the weft's lock directory until step 5 reopens it at `B`.
The re-run tells whether step 4 landed: a move landed when the marker committed at the weft's `HEAD` is `B`, and an integration landed when the marker committed at `HEAD` is `B` and the record's `U` is an ancestor of `HEAD`; a record holding no `U` is an integration that never reached step 4.

- Not landed: when `A` differs from `B`, the re-run first renames `B/.lyx` back to `A/.lyx` when only `B/.lyx` exists; when both exist it refuses, naming both, with the way forward of merging `B/.lyx` into `A/.lyx` by hand and removing it.
  It then resets the move's pathspecs, `A/<names>`, `B/<names>` and the marker, each excluding `.lyx`, to `HEAD` in the index and the working tree, and deletes what `HEAD` does not track under those `B/<names>`; `.lyx` is never reset or deleted by the recovery.
  An integration, which moves the branch only once its result is whole, has left none of them changed, and this resets nothing when `A` is `B`.
  It then runs the dirtiness check on the rest of the weft, and steps 1 to 8, its step 1 rewriting the record.
- Landed: the re-run renames `A/.lyx` to `B/.lyx` when `A` differs from `B` and only `A/.lyx` exists, and runs steps 1 to 3 and 5 to 8 idempotently, its step 1 keeping the record's `A`, `B`, kind and `U` and rewriting only its pid and start time.

A kill after step 8 removed the record leaves owed only the push, which the next weft push carries, and the dobby start, which `lyx beam start` makes.

The move record keeps `dobbyengine.EnsureLive`, and so `lyx orch start` and `lyx beam start` run anywhere in the hub, from starting a dobby the verb ended, and keeps a live dobby from starting a pass.
`lyx adopt` refuses before its first step while a move record stands in the prime or in any worktree whose runs it would resume or under which it would add a pair, so neither its `Topology.Add`, its detached first passes, its `lyx loom resume` calls nor its finalize gate suite runs under a moving anchor; the refusal names a change in progress while the move lock is held, and `lyx fabric anchor <slug> --set <B>`, naming the record's `B`, once it is free.
The verb holds no lock against a session, a hand pass or the hidden `lyx beam dobby` started by hand while it runs; the operator starts none in the worktree meanwhile, and for the prime none in the hub.
The dirty-weft refusal on a first run, and the recovery's dirtiness check on a re-run, keep any uncommitted change out of the move commit, and the verb never acts on another machine's checkout.
Child worktrees keep their own markers and anchors; the verb neither refuses on them nor touches them.

A change commits `B` on the worktree's weft branch, which every other machine that checks the worktree out shares; for the prime that is `main-weft`, which every machine's prime checks out.
On each such machine the weft pull and push, and the origin reconcile's move of that weft, stop on the marker change (D10), and the operator runs `lyx fabric anchor <slug> --set B` there, which integrates.

Rationale: the operator's design; every agent and daemon in the worktree has its cwd or state at the anchor, so the move needs them gone.

### D8 — Migration by `lyx fabric reconcile`

For each managed pair whose weft worktree is present and has no marker, reconcile writes and commits D5's markerless value, and spawns the detached weft push for that pair, as every other weft writer does; the prime's weft is migrated first.
Reconcile removes the Bolt marker in a board commit once the prime's weft and every local pair's weft carry a committed marker, and every weft branch whose warp sibling is on origin carries one on its already-fetched remote-tracking ref.
The same board commit removes a leftover `.fabric-anchor` beside it.
A committed marker that is empty after trimming counts as no marker in the census.
The census makes no network call: a branch whose ref is not fetched keeps the Bolt marker, and so does a marker commit not yet pushed, so removal lands on a later reconcile, once those pushes and a fetch have landed.
It cannot count a branch this machine never fetched, nor a marker another machine has not yet committed (Open risks).
A weft branch on origin with no warp sibling there is an orphan and is not counted; an adopt of it gets D5's value.
`refuseEmptyAnchorMarker` becomes per pair: a pair whose own marker is present but empty is reported as failed with the way forward, and reconcile goes on with the other pairs.
A pair with a standing D7 move record is reported as failed, its wiring untouched, and reconcile goes on with the other pairs; the report names a change in progress while the record's move lock is held, and the way forward `lyx fabric anchor <slug> --set <B>`, naming the record's `B`, once it is free.
`lyx fabric unwire` leaves the marker, so a later reconcile rewires at the same anchor.

Rationale: no marker stays on Bolt, as the entry requires; removal waits until no live branch needs the fallback, and no lyx command waits on the network (note `fabric-remote-async`).

### D9 — The exclude census is keyed by anchored pattern

The git-exclude file is shared by every worktree of the hub.
Removing a worktree's junction lines strips the line `/<anchor>/<name>` only when no other worktree has a junction at that same anchored path, each worktree's link checked at its own anchor.

Rationale: today's census checks every sibling at the acting worktree's anchor, which with two anchors strips lines another worktree still needs.

### D10 — Branch switches, pulls and promote across anchors

`lyx fabric checkout` refuses, before switching either side, a target weft branch whose anchor, its marker or D5's markerless value, differs from the worktree's.
Way forward: `lyx fabric anchor --set <target anchor>` and then the checkout, which the refusal names together with its cost: the `--set` commits the new marker on the branch being left, shared by every machine that checks it out, whose pulls and pushes then stop until each integrates.
When the target's anchor is not a directory git tracks in the worktree's current warp checkout, which D7's validation of `B` requires, the refusal names first updating the warp checkout by hand to a commit that tracks it, such as by merging the target warp branch into the current one or committing the anchor directory on it, then the `--set` and then the checkout.
It names that update's cost beside the `--set`'s: a code commit on the branch being left, shared by every machine that checks it out.
A forked new weft branch inherits the current marker and is never refused.
A weft pull, or a weft push that must replay onto its upstream, stops that pair before updating either side when it would change the checked-out marker or replay commits across a marker change:
the tip it would leave checked out, the fast-forward target or the replay result in `settleUnmovedWeft`, carries a marker other than the current one, or the upstream has commits the local tip lacks while the two tips carry different markers.
A markerless tip compares by D5's markerless value, so an upstream that gained markerless commits from a machine not yet migrated compares equal to a local marker naming where its content sits.
Either also refuses while the pair has a standing D7 move record (D7 step 1).
Way forward: `lyx fabric anchor <slug> --set <anchor>`, which integrates, naming the upstream tip's marker when the local tip's marker is the one the merge base carries, and the local tip's marker otherwise.
A weft ahead of an upstream that carries the old marker and no commit the weft lacks is a fast-forward push, and nothing stops it.

The origin reconcile, `fabricengine.ReconcileOrigin`, hub-wide or scoped to a run, applies the same test before it fast-forwards or replays the prime's or a pair's weft.
When the test fires, the pair has a standing D7 move record, or its own marker is empty (D2), it leaves that weft unmoved and reports the branch stopped with the pull's way forward, the move record's naming of D7 step 1, or the empty marker's file and way forward; it goes on with the other branches and never fails its caller for it.
A stop of the prime's weft names, ahead of the `--set`, D7's prime refusals it must clear first: ending every reed session of the hub, the orch's included, with `lyx reed down` in each, and waiting until no batten run of the hub is live, then `lyx orch start` again.
Unlike the pull's stop, its move of the pair's code branch goes ahead, since that move changes no marker and brings the warp a commit that may track the new anchor, which the integrating `--set` validates.
`lyx adopt --drop-local`'s reset of a pair's weft to origin's tip applies the same test, and refuses before tagging or moving any branch when origin's tip carries a marker other than the checked-out one, naming `lyx fabric anchor <slug> --set <origin's marker>` and then the adopt again.
One `fabricengine` function holds the marker comparison, and the pull, the push, the origin reconcile and the drop-local reset each call it.

The integration onto a target anchor `T` rebuilds the weft on its fetched upstream tip `U`.
It replays each local commit `U` lacks, skipping each that changed the marker, with its paths under the anchor its parent commit's marker names, or D5's value of the parent's tree when that tree carries none, re-rooted to `T`;
then, when `U`'s marker is not `T`, it applies D7's move from `U`'s marker to `T` on the result, marker included.
It builds the result off the checked-out branch and moves the branch only once the result is whole, so a conflict stops it with the weft unchanged, naming the conflicting local commit and its paths.
Way forward: tag the local tip by hand (`git -C <weft> tag archive/<slug>/<tip> <tip>`) and save the changes of the conflicting commit and of every later local commit outside the weft, reset the weft branch by hand to the conflicting commit's parent (`git -C <weft> reset --keep <commit>^`), after which a weft that was clean stays clean, so the dirty-weft refusal (D7) does not fire, re-run, and then write all the saved changes back at `T` and commit them; the tag keeps every dropped commit reachable until they are back.
`lyx beam promote` refuses a task whose anchor differs from its parent's, since resetting the records to the parent's tree carries the parent's marker; way forward: `lyx fabric anchor <slug> --set <parent's anchor>`.

Rationale: every path that can change a checked-out marker outside D7, checkout, a weft pull, a replaying weft push, the origin reconcile's weft moves, adopt's drop-local reset and promote, refuses or stops before changing anything, so none leaves a half-rewired pair; the integration changes the checked-out marker only inside D7, whose move record recovers it, and D8 and an adopt of a markerless branch write only D5's value, which names where the content already sits.

### Added after Discussion (parent, 2026-10-25): Shorter PATTERN-cwd-resolution text

Decision: The PATTERN-cwd-resolution entry text is shortened to fit pattern.MaxEntryLineChars with its background link, keeping every clause of the earlier text: one worktree's anchor only, fabricengine alone writes or git-reads it, engines never import lyxcwd, hub containers and workspace files hub-reachable only.

Rationale: The recorded text made the overview line 410 characters against the 360 cap, and dropping the background link orphans pattern/PATTERN-cwd-resolution.md.

### Added after Discussion (parent, 2026-10-25): A standing move record exempts its own worktree from D7's owner refusal

Decision: D7's owner refusal does not apply to a worktree whose own move record stands, bounded to that record's worktree B; card 17's exemption stands as written. Answers plan review finding F10.

Rationale: Adopt already refuses while the move record stands, so the owner refusal there would only block the recovery the record exists for.

### Added after Discussion (parent, 2026-10-25): Checkout joins the writers a standing move record stops

Decision: D7 step 1's list of writers a standing move record stops gains checkout; card 13's checkout refusal and its TestCheckout_RefusesAnotherAnchor row stand as written. Answers plan review finding F13.

Rationale: Recovery acts on the branch the record was written on, so a checkout while the record stands would point recovery at the wrong branch.

## Constraints

- The weft is never a merge participant (`fabricengine/doc.go`, "The merge surface"), so landing never touches the parent's weft or its marker; the entry's "kept out of the merge as raddle is" needs no mechanism.
- `lyxcwd` stays the sole cwd and worktree resolver (`PATTERN-cwd-resolution`), with the import set of D1.
- Git on warp or weft runs only in `fabricengine`, and removals go through its destruction chokepoint (`PATTERN-fabric-git`); the D7 move and its record, the D8 marker commits, and the D10 refusals and integration are `fabricengine` work.
  Session, run and dobby checks run in the CLI layer above it, as `pairteardown` sits above fabric.
- Dobbies end only through `runproc.End` or its sparing form `runproc.EndSparing` (`PATTERN-run-process-owner`).
  `dobbyengine` is told every path it reads, so `EnsureLive` learns of a move record through a `StartDeps` seam and the dobby loop through its `Geometry`, both filled by `hubgeom`.
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
- The census cannot see a branch this machine never fetched, and another machine's checkout of a counted branch may still be markerless, its pull not yet run; once a reconcile elsewhere removes the Bolt marker, such a checkout on a subpath hub resolves to `"."` until its own machine's reconcile writes its marker.
  The operator runs `lyx fabric reconcile` first on each machine after updating it to the new binary, before any other lyx command there.
- While a change of a worktree checked out on several machines has not been integrated on another machine, that machine's weft pushes, and its origin reconcile's moves of that weft, stop (D10), and its records stay local until the operator runs the integrating `--set` there.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 3: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/discussion/round-3-review.md; fixer report: _lyx/reviews/discussion/round-3-fixer-report.md.
  - origin-reconcile-moves-weft-past-d10 (class design, severity BLOCKING, open in rounds 3)
<!-- /lyx:carry-over Discussion-Review -->

## Acceptance criteria

- Two pairs in one hub with different markers each resolve their own `AnchorRel`, and the cwd gate refuses each at the other's anchor.
- A markerless weft resolves to the Bolt legacy value while Bolt has one, and to `"."` when neither exists; an invalid own marker is an error.
- An empty own marker is refused by checkout, pull, push, `Add` from it, the anchor verb, reconcile and adopt's drop-local reset, each touching nothing, and stops the origin reconcile's move of that weft.
- A fresh `fabric clone --subpath backend` writes `backend` on `main-weft` and no Bolt marker; cloning that hub again adopts `backend` and refuses a different `--subpath`.
- A task forked from a beam at anchor `X`, with and without `--parent`, has marker `X`, junctions and origin record at `X`, and no run records of its parent on disk.
- A markerless task whose parent was re-anchored after it forked migrates, and adopts, to the anchor its content sits at.
- `lyx beam create --anchor Y` makes a beam whose marker, weft content, junctions, scratch link, launchers and workspace folder sit at `Y`; it refuses an invalid, absent or link-crossing `Y`, and on a present pair refuses a different anchor and, for an equal one, skips the move and finishes the create steps; a re-run after a kill before the move commit finishes at `Y`.
- `lyx fabric anchor <slug> --set B` moves tracked content and `.lyx`, commits the marker, wires junctions and the scratch link at `B`, leaves none at `A`, moves the launchers, regenerates the workspace files and restarts the dobby it ended, sparing the pushes and board syncs outside the ended dobbies' process trees; while its move record stands, `lyx beam start` starts no dobby of the worktree, no live dobby whose geometry the worktree's anchor reaches, the worktree's own and for the prime every dobby of this machine, starts a pass or a task, `dobbyengine.EnsureLive` reports each such dobby it did not start, reconcile refuses the pair, the origin reconcile leaves its weft unmoved and reports it stopped, `lyx adopt` refuses before its first step, `Add` refuses the worktree as a fork point, and a weft pull or replaying push of it refuses.
- `lyx fabric anchor` refuses a `B` under a junction, a stored or live reed strand, a held driver lock, a live batten run, an unfinished run, a live run of the verb on the same worktree, another machine's worktree, a `B` holding a real wired name, a weft with no committed marker and a dirty weft, each with nothing touched and no dobby ended; with no pass started meanwhile, it waits until no dobby it will end has a pass in flight, a spawned pass not yet holding its driver lock included, then holds the parent lock of the worktree's branch until its last step, so a child's Finalize queues behind it; for the prime it also refuses a live session anywhere in the hub, a held gate slot and a live hibernate daemon.
- A failure before the marker commit leaves the worktree wired at `A` with a clean weft; a re-run after a kill before the commit, and after a kill between the commit and the wiring at `B`, each finishes at `B` with every dobby the run ended started again, and so does a re-run after a kill of an integration on a checkout already at `B`, each keeping the worktree's `.lyx`; a `--set` on a child forked after its parent's change, equal to its inherited anchor, is a no-op.
- After the prime's change on one machine, a second machine's prime pull and push stop with both sides unchanged, a hub-scope origin reconcile there, such as `lyx orch start`'s, leaves the prime's weft and its `.lyx` unchanged and reports the way forward, and its `lyx fabric anchor <prime> --set B` integrates, carrying its unpushed records commits to `B`.
- A child pair forked before its parent's anchor change keeps its own anchor, still runs, and lands into the parent, whose marker stays unchanged.
- Unwiring or re-anchoring one worktree keeps the exclude lines another worktree at the same anchor needs.
- Reconcile writes the right marker into each markerless local weft and spawns its push, removes the Bolt marker and any `.fabric-anchor` beside it only when the census over fetched refs finds every counted branch marked, an empty committed marker counting as none, makes no network call for it, and fails only the pair with an empty marker or a standing move record.
- `fabric checkout` to a branch with a different anchor and a weft pull bringing a different marker are refused with both sides unchanged, the checkout naming the `--set` and that it re-anchors the branch being left, and first a warp update when the current warp checkout does not track the target's anchor; a markerless upstream tip whose content sits at the local marker stops neither; a weft ahead of its upstream with a re-anchor not yet pushed is neither refused on pull nor stopped on push.
- `beam promote` refuses a task whose anchor differs from its parent's.
- The hub and beam workspace files, the dobby geometry and the pair teardown use each worktree's own anchor in a hub with two anchors.
- The scan fails on a production `lyxcwd.Location` literal outside `lyxcwd` and `internal/testkit`.

## PATTERN changes

- `PATTERN-cwd-resolution` — changes: Resolving a cwd, worktree root or anchor: only `internal/lyxcwd`, from the worktree's own weft-root `.lyx-anchor`; only `fabricengine` writes or git-reads it; engines get absolute paths, never import lyxcwd; hub containers and workspace files stay hub-reachable only. (test) — added after Discussion (parent, 2026-10-25)

## Notes for the plan writer

- The background file `pattern/PATTERN-cwd-resolution.md` needs the import set of D1 and a line on D3's constructor and scan; `leaf_enforcement_test.go` pins the import cap.
- The D7 commit spans `A/<names>`, `B/<names>` and the root marker, so it is not one of today's anchor-scoped `CommitRecordsPaths` calls.
