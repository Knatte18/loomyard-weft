# Support log — umbrella-anchor

## Interview

### Turn 1 — board and code read (autonomous)

Read the board entry, the board list and the peers list; no `Decision for umbrella-anchor:` note exists, and the one peer run (`dobby-skip-done-runs`) touches only `internal/dobbyengine`.
`adopt-runs-tests` has landed (`42dee71ec`), as the entry's run section requires.
Read `lyxcwd`'s resolver and anchor read, `fabricengine`'s anchor, wiring, merge and scratch sections in `doc.go`, and the files the entry names.

Facts settled from the code:

- Today's marker is `_board/.lyx-anchor` on Bolt; this hub's reads `.`.
- The weft worktree mirrors the anchor (`<slug>-weft/<anchor>/_lyx`), and both warp `_lyx` and `.lyx` are junctions into it, so the warp's `.lyx` is the weft's.
- The weft is never a merge participant, so the entry's "kept out of the merge as raddle is" needs no mechanism.
- Every `(l, slug)` path builder in `fabricengine`, and `pairteardown`, `hubgeom`, `beamcli`, `ideengine` and `fabriccli`, copies the acting Location's `AnchorRel` onto another worktree.
- `createWeftWorktreeDroppingRuns` scopes the run-records root under the acting Location's anchor even when `--parent` names another fork point.
- The exclude census in `namesWiredInSiblingWorktrees` checks siblings at the acting anchor.
- `lyx fabric checkout`, weft pulls and `beam promote`'s records reset can each change the checked-out marker.
- `lyxcwd` imports the standard library and `dotgit` only; `weftname` is a standard-library leaf.

### Turn 2 — design batch (autonomous; every pick is an auto-pick)

1. Where the marker is read: from the working tree at `<hub>/<name>-weft/.lyx-anchor`, through `weftname` admitted into `lyxcwd` (recommended), or a private `-weft` copy in `lyxcwd`. Picked the admission (D1).
2. Fallback for a markerless weft: own marker, then Bolt legacy, then `.` (recommended); or refuse a markerless fabric weft. Picked the fallback chain (D2).
3. Bolt marker removal: after an origin-wide census (recommended), on the first reconcile, or never. Picked the census (D8).
4. Another worktree's Location: one `lyxcwd` constructor plus a scan on `AnchorRel` literals (recommended), or fixing each site by review alone. Picked the constructor and scan (D3).
5. The change verb's home and shape: `lyx fabric anchor [<slug>] [--set <dir>]` (recommended), a `lyx beam` verb, or acting-worktree only. Picked the fabric verb with an optional slug (D7).
6. Live sessions and runs: refuse (recommended) or end them as teardown does. Picked refuse (D7).
7. Launchers: keep the anchor mirror keyed by each worktree's own anchor (recommended) or flatten to `_launchers/<slug>`. Picked the mirror (D7 step 6).
8. Checkout across anchors: refuse (recommended) or rewire. Picked refuse; a pull first rewired, revised to a refusal in the lens review (D10).
9. Promote of a re-anchored task: refuse (recommended) or re-anchor. Picked refuse (D10).
10. Children of a re-anchored worktree: keep their own anchors, no refusal (recommended). Picked that (D7).

### Turn 3 — self-check

`lyx loom validate-discussion` run after writing the three files.

### Turn 4 — lens review

All seven forks returned a report; duplicates across lenses are merged on one line.

- interactions 1, walkthrough 2, bound-breaker 1 (markerless value from the parent's anchor is wrong after a re-anchor): fixed; D5 takes the shallowest directory holding a wired name, else D2's resolution, for migration, adopt and checkout.
- interactions 2 (adopt value): fixed by the same D5 rule.
- interactions 3, bound-breaker 3, walkthrough 3 (dobby race and ending its passes): fixed; D7 step 1 ends the dobby process alone first, then checks, and restarts on a refusal; Windows refuses a live dobby (superseded in round 1).
- interactions 4, existing-paths 2, bound-breaker 4 (failure restore misses the working-tree marker): fixed; the marker is written last and the restore covers working tree, index, `.lyx` and marker, and a re-run undoes an uncommitted move.
- interactions 5, walkthrough 7, bound-breaker 11 (re-run cannot find `A`; inherited anchor looks like a resume): fixed; the change commit carries `Lyx-Anchor-From`, and no trailer means nothing to clean; a settled `--set` is a no-op (superseded in round 1 by the move record).
- interactions 6, existing-paths 3, walkthrough 8 (pull rewire unbounded and not resumable): fixed by replacing the rewire with a refusal before either side moves (D10).
- interactions 7, walkthrough 1 (new pair's anchor read before the weft exists; origin record before the move): fixed; `Add` takes the anchor from the fork point or `--anchor`, and D6 moves before any write at the new anchor.
- interactions 8, existing-paths 1 (reconcile does not recover an interrupted change, `isRawWarpWorktree`): fixed; recovery is the verb's re-run, the reconcile claim is dropped.
- interactions 9, walkthrough 10, bound-breaker 13 (gone pair's anchor): fixed; D3 states consumers of a gone pair use no anchored path.
- interactions 10 (portals under an old prime anchor): fixed; stated in D3.
- existing-paths 4, bound-breaker 12 (prime's hub-wide state and daemons): fixed; the prime's refusals cover any session in the hub, gate slots and the hibernate and creel daemons.
- existing-paths 5, bound-breaker 10, walkthrough 5, goal-scope 1 (census never completes on orphan branches): fixed; branches with no warp sibling on origin are not counted, and removal lands on a later pass after pushes.
- board-fit 1 (census reads the network, against `fabric-remote-async`): fixed; the census reads fetched refs only.
- bound-breaker 2 (empty marker drives a rewire at the root): fixed; every `fabricengine` path acting on a marker refuses an empty one (D2).
- walkthrough 4, bound-breaker 5 (`B` under a junction): fixed; D7 validation requires a tracked directory with no link or managed segment, and D6 uses it.
- bound-breaker 6 (own log sink under `A/.lyx`): fixed; step 3 closes the sink and step 4 reopens it at `B` (steps 4 and 5 since round 1).
- bound-breaker 7, walkthrough 9, existing-paths LOW scan (scan misses joins and omitted fields; testkit literal): fixed; the scan bans every production Location literal outside `lyxcwd` with `internal/testkit` exempt, and slug-built joins are held by review.
- bound-breaker 8 (PATTERN line contradicts fabricengine's git-object reads): fixed; the line and D1 name `fabricengine` as the marker writer and git-object reader.
- bound-breaker 9 (old binary after Bolt removal): accepted as an open risk with the operator step; no lyx code can see another machine's binary.
- bound-breaker 14, walkthrough 6 (dobby restart scope and failure path): fixed; step 7 restarts exactly the dobbies step 1 ended, and a refusal restarts them (superseded in round 1: step 8 starts every covered dobby whose lock reads free).
- bound-breaker 15 (checkout against a markerless target): fixed; D10 compares D5's markerless value.
- existing-paths LOW reed strand state: fixed; the refusal covers stored strands, cleared by `lyx reed down`.
- goal-scope 2 (weft pathspecs implicit): fixed; D3 names weft pathspecs.
- board-fit 2 (dobby overview task): rejected; D3's rule already covers it and nothing changes in this record.
- leanness D3 call-site list: fixed; the list is cut, `relevant.md` keeps the files.
- leanness auto-mode assumptions: fixed; restated bullets cut, the portal rule moved into D3.
- leanness D2 "only reader" contradiction: fixed; the sentence is replaced by D1's writer and reader rule.
- leanness D7 rationale: fixed; it keeps the positive reason only.
- leanness Open risks prime and Windows bullets: fixed; both cut, the Windows dobby case is a D7 refusal.
- leanness Scope doc list: fixed; cut, Constraints keeps it.
- leanness Notes duplicating `relevant.md`: fixed; cut.

## Rejected alternatives

- An anchor index on Bolt: the operator ruled it out; the marker on each weft branch makes it unneeded.
- A separate hub per subdirectory: the operator ruled it out, since beams share one hub and one board.
- A private `-weft` literal in `lyxcwd`: a second declarer of a token `weftname` already owns alone.
- Refusing a markerless fabric weft in the reader: it breaks fixtures, mid-clone states and every unmigrated hub at once.
- Removing the Bolt marker on the first reconcile: another machine's unmigrated pairs would resolve to `.` on a subpath hub.
- Keeping the Bolt marker forever: the entry says no marker stays on Bolt.
- Fixing each `AnchorRel` copy by review alone: the one-anchor assumption would return with the next sibling builder.
- Ending sessions and runs in the change verb: every agent's cwd stops being the anchor mid-turn; a refusal with `lyx reed down` is simpler.
- Flattening `_launchers` to `_launchers/<slug>`: a layout migration the entry does not ask for.
- A markerless branch taking its parent's anchor: wrong once the parent is re-anchored after the fork (lens review).
- Rewiring a pair on a weft pull that changes its marker: an interrupted rewire cannot be resumed and runs without the change verb's quiet checks (lens review).
- Recovering an interrupted anchor change through `lyx fabric reconcile`: its wiring repair wires at whatever anchor the checked-out marker names and moves neither content nor launchers, so it refuses a pair with a standing move record instead.
- Reading `A` for a recovery from a `Lyx-Anchor-From` trailer: a kill before the commit leaves an older trailer last, and a child forked after its parent's change inherits the parent's (round 1).
- Ending a dobby alone through `runproc.End`: `End` also ends every descendant, so passes in flight are waited out first and pushes are spared through `EndSparing` (round 1).
- Refusing a prime change while other machines hold a prime: no record names the machines that check the prime out (round 1).
- `lyx fabric remove` and re-adopt as a pull's way forward: `refusePrimeSlug` refuses it for the prime, and the integrating `--set` serves the prime and pairs alike (round 1).
- Refusing while a child pass is in flight: a live dobby keeps starting continue passes, a pause included, so the refusal could fire on every attempt; the dobby starts no pass while the move record stands and the verb waits (round 1 self-check).
- Restarting exactly the dobbies the run ended: a killed run's set is lost, so the restart starts every dobby the verb covers whose lock reads free (round 1).
- A lyxcwd fallback to the prime's marker for a markerless weft: `lyxcwd` does not know the prime, and an operator step bounds the window (round 1).
- Rewiring on `fabric checkout` across anchors: checkout is all-or-nothing with rollback, and an anchor rewire inside it widens that rollback for a case only the change verb creates.
- Re-anchoring the target branch on a `fabric checkout` across anchors, or routing the switch through `Add`: a worktree holds one anchor at a time, so a switch across anchors re-anchors one of the two branches, and the branch being left is the one this machine has checked out and can move with D7's recovery; `Add` wires a new pair at its fork point's anchor (D5), so it offers no route around the `--set` (round 3).
- Re-anchoring on `beam promote`: the reset to the parent's tree would need a move inside a reset; a refusal costs one verb call.
- `--anchor` on `lyx fabric add`: no one asked for it, and `lyx fabric anchor` covers a task pair.

## Review rounds

### Round 1

The review returned three BLOCKING findings, eight MEDIUM and five LOW; every premise held against the code and every finding was fixed.

- F1, F15, F8: every D7 refusal runs before anything is touched; with the move record standing, dobbies start no pass and the verb waits out a child pass in flight and a held parent lock, then ends the dobbies through `runproc.EndSparing`, sparing pushes and syncs outside their process trees.
- F2, F9: D10 stops a pull or push that would change the checked-out marker or replay across a marker change, and the integrating `--set` carries another machine's checkout across the change.
- F3, F5, F6, F7, F4, F16: a move record and move lock in the weft's lock directory mark a change in flight and tell a live run from a killed one, gate `EnsureLive`, the dobby loop and reconcile, and drive the re-run; the trailer is dropped.
- Self-check: the four fix-check forks' findings were all fixed; the recovery tells an integration's landing from the tip the record holds, the integration conflict names a reset by hand, and an integrating run on a machine whose warp lacks `B` names a warp update.
- F10, F11, F12, F13, F14: D8 pushes its marker commits, counts an empty marker as none and removes `.fabric-anchor`; the census limit is an open risk with an operator step; D4 probes `main-weft` and scopes the stale guard.

### Round 2

The review returned three BLOCKING findings, seven MEDIUM and three LOW; every premise held against the code and every finding was fixed.

- F1, F3: D7 step 1 waits until no dobby step 2 will end has a pass in flight, a live non-spared child of the dobby counting, and only then takes and holds the parent lock until step 8, so a child's Finalize queues behind it and no waited pass queues behind the verb; step 4 runs under the weft write lock.
- F2: the not-landed recovery renames `B/.lyx` back first and never resets or deletes `.lyx`.
- F4, F5, F6: while a move record stands, `lyx adopt`, `Add` from the worktree, and a weft pull or replaying push refuse; an integration records its upstream tip `U` at step 4 and has landed when `HEAD` carries `B` and contains `U`; step 8 removes the record before spawning the push.
- F7: the creel daemon refusal is dropped, its state sitting outside every anchor; the hibernate refusal names `lyx hibernate cancel`.
- F8: a markerless tip compares by D5's value in D10's pull and push comparison and the integration's source anchor.
- F9, F10: checkout's way forward names `--set` together with its cost to the branch being left, and why that cost is accepted; the conflict way forward tags the local tip, saves every dropped commit's changes outside the weft and writes them back.
- Self-check: the four fix-check forks' findings were all fixed; a pair made by `lyx fabric add --adopt-local` was dropped from checkout's way forward, since `Add` wires at its fork point's anchor, the dirty-marker way forward restores the marker, and the creel lead-in names hibernate only.
- F11, F12, F13: D7 refuses a dirty weft and a weft with no committed marker on a first run; the criterion spares only pushes and syncs outside the ended dobbies' trees.

### Round 3

The review returned one BLOCKING finding and four LOW; every premise held against the code and every finding was fixed.

- F1: the origin reconcile applies D10's marker test, and the move record check, before it moves a prime's or pair's weft, leaving it unmoved and reporting it stopped; one `fabricengine` function holds the comparison, and D10's rationale lists every path that can change a checked-out marker.
- F2: checkout's way forward names a warp update first when the current warp checkout does not track the target's anchor; the reason the `--set` cost is accepted moved to Rejected alternatives.
- F3: a replayed local commit's markerless parent tree takes D5's value.
- F4: every run of the verb writes its pid and start time into the move record, which the move-lock refusal reads.
- F5: the criterion names the live dobby's pass gate and `EnsureLive`'s report.
- Self-check: the four fix-check forks' findings were all fixed; `lyx adopt --drop-local`'s weft reset takes the same marker test, the origin reconcile stops on an empty marker and a prime stop names the sessions and runs to clear before the `--set`, and the checkout's warp update names its cost.

## Question ledger

- Q1 marker read path and `lyxcwd` imports — resolved, auto-pick: admit `weftname` (D1).
- Q2 fallback for a markerless weft — resolved, auto-pick: own, Bolt legacy, `.` (D2).
- Q3 Bolt marker removal — resolved, auto-pick: after a census over fetched refs, orphans uncounted (D8, revised in the lens review).
- Q4 another worktree's Location — resolved, auto-pick: `lyxcwd` constructor and scan (D3).
- Q5 change verb shape — resolved, auto-pick: `lyx fabric anchor [<slug>] [--set <dir>]` (D7).
- Q6 live sessions and runs — resolved, auto-pick: refuse (D7).
- Q7 launcher layout — resolved, auto-pick: anchor mirror per worktree (D7).
- Q8 checkout and pull across anchors — resolved, auto-pick: both refuse (D10, pull revised from a rewire in the lens review).
- Q12 anchor of a markerless branch — resolved, auto-pick: shallowest directory holding a wired name, else D2's resolution (D5, from the lens review).
- Q13 recovery of an interrupted anchor change — resolved, auto-pick: a re-run of the verb, driven by a move record in the weft's lock directory (D7, revised in round 1 from a `Lyx-Anchor-From` trailer).
- Q14 a change of a worktree other machines check out — resolved, auto-pick: their pulls and pushes stop, and an integrating `--set` on each carries it across (D7, D10, round 1).
- Q9 promote of a re-anchored task — resolved, auto-pick: refuse (D10).
- Q10 children of a re-anchored worktree — resolved, auto-pick: keep their own anchors (D7).
- Q11 the entry's "kept out of the merge as raddle is" — resolved by fact: the weft never merges (Constraints).
