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
7. Launchers: keep the anchor mirror keyed by each worktree's own anchor (recommended) or flatten to `_launchers/<slug>`. Picked the mirror (D7 step 5).
8. Checkout across anchors: refuse (recommended) or rewire. Picked refuse; a pull first rewired, revised to a refusal in the lens review (D10).
9. Promote of a re-anchored task: refuse (recommended) or re-anchor. Picked refuse (D10).
10. Children of a re-anchored worktree: keep their own anchors, no refusal (recommended). Picked that (D7).

### Turn 3 — self-check

`lyx loom validate-discussion` run after writing the three files.

### Turn 4 — lens review

All seven forks returned a report; duplicates across lenses are merged on one line.

- interactions 1, walkthrough 2, bound-breaker 1 (markerless value from the parent's anchor is wrong after a re-anchor): fixed; D5 takes the shallowest directory holding a wired name, else D2's resolution, for migration, adopt and checkout.
- interactions 2 (adopt value): fixed by the same D5 rule.
- interactions 3, bound-breaker 3, walkthrough 3 (dobby race and ending its passes): fixed; D7 step 1 ends the dobby process alone first, then checks, and restarts on a refusal; Windows refuses a live dobby.
- interactions 4, existing-paths 2, bound-breaker 4 (failure restore misses the working-tree marker): fixed; the marker is written last and the restore covers working tree, index, `.lyx` and marker, and a re-run undoes an uncommitted move.
- interactions 5, walkthrough 7, bound-breaker 11 (re-run cannot find `A`; inherited anchor looks like a resume): fixed; the change commit carries `Lyx-Anchor-From`, and no trailer means nothing to clean; a settled `--set` is a no-op.
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
- bound-breaker 6 (own log sink under `A/.lyx`): fixed; step 3 closes the sink and step 4 reopens it at `B`.
- bound-breaker 7, walkthrough 9, existing-paths LOW scan (scan misses joins and omitted fields; testkit literal): fixed; the scan bans every production Location literal outside `lyxcwd` with `internal/testkit` exempt, and slug-built joins are held by review.
- bound-breaker 8 (PATTERN line contradicts fabricengine's git-object reads): fixed; the line and D1 name `fabricengine` as the marker writer and git-object reader.
- bound-breaker 9 (old binary after Bolt removal): accepted as an open risk with the operator step; no lyx code can see another machine's binary.
- bound-breaker 14, walkthrough 6 (dobby restart scope and failure path): fixed; step 7 restarts exactly the dobbies step 1 ended, and a refusal restarts them.
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
- Recovering an interrupted anchor change through `lyx fabric reconcile`: reconcile treats a pair with no `_lyx` at its anchor as unmanaged.
- Rewiring on `fabric checkout` across anchors: checkout is all-or-nothing with rollback, and an anchor rewire inside it widens that rollback for a case only the change verb creates.
- Re-anchoring on `beam promote`: the reset to the parent's tree would need a move inside a reset; a refusal costs one verb call.
- `--anchor` on `lyx fabric add`: no one asked for it, and `lyx fabric anchor` covers a task pair.

## Review rounds

_No rounds yet._

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
- Q13 recovery of an interrupted anchor change — resolved, auto-pick: a re-run of the verb, with `A` from the `Lyx-Anchor-From` trailer (D7, from the lens review).
- Q9 promote of a re-anchored task — resolved, auto-pick: refuse (D10).
- Q10 children of a re-anchored worktree — resolved, auto-pick: keep their own anchors (D7).
- Q11 the entry's "kept out of the merge as raddle is" — resolved by fact: the weft never merges (Constraints).
