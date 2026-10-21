# Decision record — origin-reconcile

## Goal

A machine never acts on a stale view of origin, and a push that origin refuses is never silent.
Before every step where a stale view does harm, one origin reconcile compares this machine's branches and Bolt statuses with origin and fast-forwards, cleans up a run that landed elsewhere, or stops naming the diverged branch.
A rejected push recovers by itself where it safely can, and otherwise reaches the orch and stays visible until it is settled.
Two places act on an unverified view on purpose, each bounded in its decision: an unreachable origin (Auto-mode assumptions) and the orch start, which reports a stop and never refuses (D8).

## Scope

In:

- The origin reconcile in `internal/fabricengine`: two parallel `git ls-remote origin` calls (warp repo, records repo), one fetch per repo of only the moved heads, and the outcomes of D4 and D7.
- Its call sites: `lyx orch start`, the hibernate daemon's wake, batten's resume of an existing run, the Merge-In and Finalize rows, and `lyx cleanup`.
- A records replay that replays local commits onto origin and drops those that become empty, by commit on a weft and by board entry on Bolt, shared by the reconcile, `Fabric.Pull` and the push retry.
- `lyx fabric pull` failing, with a way forward, when the weft cannot be brought level with origin, and its warp arm judging unpushed commits and a rewind by D4's reflog tests.
- A rejected push: fetch, records replay, retry; a push still failing writes a failed-push record and one creel notice to the orch.
- `lyx cleanup`, `lyx fabric status` and `lyx tender status` printing outstanding failed pushes; `lyx cleanup --apply` no longer reporting a spawned push as landed.
- A local mode of `internal/pairteardown` that runs no origin step, which `run-process-ownership` reuses for its own non-owner removal.
- The Preflight row's records push going detached (folded note `preflight-weft-push-sync`).

Out:

- Owner machines, `lyx adopt` and the owner outcome: `machine-handover`, which slots one more outcome into D7's classifier and calls this reconcile for `adopt`'s bring-up-to-date step.
- Ending a run's orphaned processes and dropping parked pushes under a pause: `run-process-ownership`.
- Making the other synchronous remote operations async: `fabric-remote-async`, which reuses D10's failed-push record and notice for its async failures.
- Dobby's calls of the reconcile, and whether its re-entry of `lyx batten run` skips D8's resume reconcile: `umbrella-daemon`.
- Adopting origin-only branches this machine lacks.
- Serialising the board sync and the weft push into one records-repo push: D11 removes the synchronous half of that pair, and the existing backoff retries the bare `(failed)` refusal.

## Decisions

### D1. One reconcile, in fabricengine, over two ls-remote calls

The reconcile is a fabricengine entry point taking a hub location and a scope, and returns a typed report with one outcome per branch and per entry it judged.
It runs `git ls-remote origin` once for the warp repo and once for the records repo, in parallel, a pair only the wake reconcile repeats, within its bound, while origin is unreachable (D8), then fetches only the heads that differ from the local tracking refs, in one fetch per repo.
Its Bolt step reuses that fetch, never `Bolt.pullFirst`'s own.
Nothing else in the reconcile touches the network.

fabricengine owns the git judgment, the moves and the replays.
Every step that needs a package importing fabricengine is a seam the caller injects:
a live-pair probe over the driver strand, the run and loop locks and the hub pause (D4),
a teardown func over `internal/pairteardown`'s local mode (D6),
a notify func over `orchcli.NotifyPrime` (D10),
and the Bolt entry merge over `internal/boardengine` (D5).
Each call site's CLI package wires them, and `fabriccli` wires the push path's seams for the detached push child.

Rationale: PATTERN-fabric-git puts git on warp and weft, Bolt included, in fabricengine alone, and the board entry fixes two calls per reconcile.
It is the synchronous remote operation `fabric-remote-async` allows: a check of what exists on origin, and the fetch of what was found there.
`pairteardown`, `reedengine`, `creelcli`, `orchcli` and `boardengine` import fabricengine, so fabricengine reaches them only through seams.

### D2. Scope: the hub, or one run

- Hub scope (orch start, wake, `lyx cleanup`): Bolt, the prime's warp and weft branches, and every local pair's warp and weft branches.
- Run scope (batten resume, Merge-In, Finalize): Bolt, the run's parent branch on both sides, and the run's own pair.

Both scopes make the same two ls-remote calls; the scope only bounds which branches the reconcile judges.

### D3. Order of work

1. Both ls-remote calls in parallel, then the fetches.
2. The ended set: every slug whose entry on the fetched origin Bolt tip is `done`, or removed from origin, read from origin and not from local Bolt, so it is known before any replay.
   An entry is removed from origin when it is absent from the fetched origin Bolt tip and present in a commit of that tip's own history, read only for each slug at issue: one this machine holds a batten run record of (D6), or one a local records commit origin lacks touches (D5).
   Where local Bolt also holds an entry of that slug, the historic entry must carry its `created` stamp, so a slug pruned on origin and reused by an entry only local Bolt holds is never ended.
   The evidence is otherwise origin's alone, so it holds whatever local Bolt and its tracking ref did before, a pull-first's fast-forward past the removal included.
   An entry only local Bolt holds, such as one created while Bolt pushes were refused, matches no entry of origin's history and is never ended.
3. Bolt: fast-forward, or the records replay of D5.
4. The prime's branches, then each scoped pair, judged per D4, with D6 applied to an entry before its branches are judged.

A stop on Bolt or on the prime's records branch stops every scoped entry: each gets the entry outcome `stopped` with that cause, so no call site acts on an entry the reconcile could not judge.
A busy Bolt or prime weft (D4) is no such stop.

Rationale: judging runs from a stale Bolt is the failure the task fixes, and the replays of steps 3 and 4 need the ended set first.

### D4. Per-branch outcomes and who may move what

For each scoped local branch with an origin tracking ref:

- **level**: nothing to do.
- **fast-forward**: the local tip is an ancestor of origin's head; the branch moves to it.
- **ahead**: origin's head is an ancestor of the local tip; the next push carries it.
- **rewound**: a code branch was rewound on origin, as `ResetPairCode`'s leased update does on another machine, when an earlier value of its tracking ref is not an ancestor of origin's head but origin's head is an ancestor of that value, and the local tip holds commits of that value origin's head lacks; it is reported naming the rewind and the commits it dropped, and its entry is `stopped` (D7), so no call site drives it and the next push does not restore them.
  The earlier values are read from the tracking ref's reflog, every value from the last entry this machine's own push wrote onward, so a fetch outside the reconcile, such as `lyx fabric pull`'s, the push retry's or Bolt's pull-first, moves the ref without erasing the evidence.
  This test takes precedence over **ahead**, which a rewound branch whose tracking ref was already fetched to origin's head would otherwise read as.
  The way forward is `ResetPairCode`'s own: rewind the local branch the same way, or the `ours` merge its refusal names.
  A rewind this machine made, and the push after that `ours` merge, wrote a push entry to the reflog, so the test sees only another machine's rewind not yet settled; a checkout rewrite that failed after this machine's own remote update is `ResetPairCode`'s error to report, and re-running it converges.
- **rewritten**: a code branch whose upstream was rewritten and that holds no unpushed commit, every local commit being reachable from one of the tracking ref's reflog values **rewound** reads, and origin's head an ancestor of none of those values; its entry is `stopped` (D7), and the way forward is `lyx fabric pull` in the pair's worktree, whose anchor walk and reset recover it once its unpushed test reads the reflog as this test does (D9).
  Whether origin's head is an ancestor of an earlier value separates the two, so they never overlap: a rewind, which `ResetPairCode`'s leased update makes, is **rewound** whether or not the branch holds an unpushed commit, and any other rewrite is **rewritten**.
  A rewritten branch never reads as **ahead**, since origin's head being an ancestor of the local tip would make it an ancestor of the earlier value that reaches that tip.
- **diverged**: neither is an ancestor, and the branch is not rewritten.
  A records branch (a weft or Bolt) first gets D5's replay and is diverged only when that replay stops; a code branch is diverged at once, and is never rebased.
- **gone**: the local branch's upstream no longer exists on origin; the reconcile recreates nothing.

A branch moves only where no live writer can meet it:

- A branch checked out in a worktree moves by a fast-forward of that worktree; a tracked change on a path the move touches stops it as **dirty**, naming the paths.
- The prime's branches move only while the reconcile holds the prime lock, taken by one try-acquire with no wait, and one read of the prime branch's parent lock record finds no live owner.
  The reconcile is no run, so it takes no parent lock ticket.
  A held lock reports the prime **busy** at once, judged and reported but not moved, since the landing or Land row holding it pulls the prime itself.
  A run that takes the parent lock after the read meets the prime's branches at origin's head, which is where Merge-In's own fast-forward moves them.
- The run scope never moves the parent branch: it judges it, and Merge-In's existing fast-forward under the parent lock moves it.
- A pair whose run lock or loop lock is held, or whose driver strand is live, outside the run scope's own pair, is judged and reported **live**, never moved.
  Under a live hub pause entry a pair the pause covers is not live for the reconcile: its driver is interrupted, so its branches are judged and moved like any quiet pair's.
- A records branch moves under its write lock, then its push lock: on Bolt `board.lock` then `.gitrepo-push.lock`, the order `PullThenCommitWritten` holds, and on every weft the weft's write lock then its push lock.
  Each is taken by one try-acquire with no wait, as the prime lock is: a held write or push lock reports the branch **busy**, judged and not moved.
  A busy pair weft stops its entry (D7), outside the run scope's own pair; a busy Bolt, prime weft or run-scope own weft is reported and stops nothing, since its holder's own push meets D10's retry.
  At wake this covers a push parked under the hub pause, which holds its weft's push lock while it waits at the network gate.

Bound: the reconcile moves a local branch only forward to origin's head or by D5's replay; it never resets away a commit origin lacks, never force-pushes, never deletes or recreates a remote branch, never waits on the prime lock, a parent lock, a records write lock, a push lock, a pair's teardown lock, the hub workspace lock or a driver's quiet wait, runs no store gc, waits only for its own teardown's loop kill to take effect, at most two `runLockWait` waits per ended pair and their sum over a hub's ended pairs (D6), and never moves a branch a live run, a landing or a held lock is using.

### D5. Records replay: replay, drop empties, stop on conflict

One fabricengine helper brings a records branch level with origin when both moved.
Its caller passes the ended set:

- the reconcile passes D3's set;
- `Fabric.Pull`, the push retry and Bolt's pull-first pass the set D3 step 2's rule reads from the origin Bolt tracking ref's tip and that tip's own history, as their own fetch left it, with no network call for it.
  The push retry's fetch includes Bolt's branch of the records repo, so a weft push reads a Bolt tip as fresh as its own fetch, and pull-first's fetch is of Bolt itself.

In every case the helper leaves out of the set a run whose local status history shows Finalize entered (D6), so this machine's own landing is never set aside.

It first sets aside the local commits that belong to an ended entry:

- on a weft, a commit whose every path lies under an ended run's record directory;
- on Bolt, a commit whose every changed `board.json` entry, and every `design-<slug>.md` it touches, belongs to an ended entry; `README.md` does not count, since every board write renders it.

It replays the other local commits origin lacks onto origin's head, oldest first, drops one that becomes empty and keeps each other one:

- on a weft, by cherry-pick;
- on Bolt, by entry: each kept commit's entry-level changes against its parent, every entry it adds, removes or changes a field of and every design file it touches, are re-applied onto origin's `board.json` through the Bolt entry merge (D1), which re-renders `README.md` and the design files, so `README.md` and an entry only one side changed never conflict.
  A Bolt commit is empty when its re-applied entry-level and design-file changes are empty, since every re-render stamps `README.md` with the time; it is dropped with no re-render.
  The merge runs inside the `board.lock` and `.gitrepo-push.lock` its caller already holds, takes neither itself and spawns no detached sync, so it is a boardengine entry point beside `boardCriticalSection`, never a call to it.
  It loads a Store from origin's `board.json` and applies the board's own checks, `validateWrite` and `checkRunLocks`, never bypassing the run lock, and an entry it adds takes a fresh id from origin's `board.json`.
  The unit of a refusal is one change: each entry change and each design-file change is validated and applied on its own, against the board as the changes kept before it left it.

On a weft a conflict, a path both sides changed such as a `status.json`, aborts the replay and restores the starting tip; the branch is diverged, and the stop names the branch, the conflicting paths and the way forward.
On Bolt no entry change stops the replay: where both sides changed the same field of the same entry, or the same design file, since the merge base, or the local commit changes an entry origin removed, or the board's validation refuses the change, origin's side is kept for that change alone, and the local commit is also kept under a set-aside tag and reported with the values it lost and any refusal, which `lyx board` can write again, after the run holding the entry ends where the refusal was the run lock's.
The board's refusal classes are the run lock on an entry origin holds under a run status, a dangling or cyclic `depends_on`, the isolated and kind rules, an invalid issue number and an unknown label.
A later kept change that depends on a refused one, such as a `depends_on` naming an entry the refused change added, is refused too and reported, so the set-aside tag carries exactly the refused changes and every other change of the same commit is kept.
Each set-aside commit is kept under a local `set-aside/<slug>/<sha>` tag, a prefix no archive reader matches, and reported.
It refuses a tracked change on a path the replayed commits touch.

The reconcile, `Fabric.Pull` (D9), the push retry (D10) and Bolt's pull-first call it, and Bolt's push recovery and seed-commit drop become cases of it, replacing `gitrepo`'s pull-rebase on every fabric push path.

Bound: it replays only records branches, only commits origin lacks, keeps every non-empty commit it does not set aside, sets aside only an ended entry's commits, never sets aside a run landing on this machine, changes on Bolt only the entries and design files the kept commits changed, applies each Bolt change on its own through the board's validation with the run lock in force, under the locks its caller holds, and gives an added entry a fresh id from origin's `board.json`, keeps a Bolt commit that lost a value or met a refusal under its tag, and never rewrites a commit origin holds.
Rationale: this reconciles the board entry's "diverged: stop" with #737's "rebase the weft, stop only on a real conflict": a weft divergence is a stop only when the replay meets a conflict.
Every Bolt commit rewrites the shared `board.json` and the time-stamped `README.md`, so a textual replay of Bolt conflicts on nearly every two-machine divergence; Bolt replays entries instead.
A Bolt stop would also leave no way forward, since every Bolt push meets the same replay, so origin's value wins a Bolt field and the lost value stays reported and recoverable.
A second machine driving the same run conflicts on its `status.json` within one transition, since every transition writes it, so concurrent driving still stops.

### D6. Ended entries

An entry is ended on this machine when it is in D3's ended set and this machine holds a batten run record of that slug, judged from the local records before any move.
A pair with no batten run record is never classified, only its branches judged, and an entry only local Bolt holds is never ended (D3).
Its own landing is not ended: a run whose local status history shows Finalize entered is landing here, and batten's own rows finish it.
The run scope classifies its own run as ended like any other but never runs D6's teardown steps on it, since its driver is the caller.

For an ended entry the reconcile:

1. Removes the pair through `internal/pairteardown` with `RefuseWhenBusy` and a quiet wait of zero, so a busy driver is reported busy at once, in the local mode, through D1's teardown seam: Remove commits the pending records as today, then puts a local `set-aside/<slug>/<sha>` tag on the weft tip instead of pushing an archive tag, and runs no other origin step and no board write.
   The local mode waits on no other writer: it takes the pair's teardown lock and the weft write lock `commitPendingRecords` needs each by one try-acquire, and reports the pair busy when either is held, where today's Remove waits up to `StatusPushLockWait` on the teardown lock and blocks on the weft write lock.
   It regenerates the hub workspace file under one try-acquire of the hub workspace lock, where today's Remove waits up to `hubWorkspaceLockWait`; a held lock skips the regeneration with the warning that names `lyx fabric reconcile`, as a failed regeneration does today.
   It skips Remove's store housekeeping, whose foreground `git gc --auto` has no time bound, and leaves the gc to the next ordinary Remove's housekeeping.
   It keeps `EndSession`'s two waits for its own loop kill to take effect, the loop lock's seizure and the run lock's release, each bounded by `runLockWait`, since those locks are held by the pair's own loop and step tree, which the kill ends, and a try-acquire before the kill would read every paused pair as busy.
   These two are the local mode's only waits, so one ended pair waits at most twice `runLockWait`, and a hub reconcile tearing down several ended pairs in turn waits at most that sum over them.
   A warp branch Remove keeps for unlanded work is reported with Remove's kept reason.
2. Writes no commit of its own: the board entry asks for the run record to be removed locally, and D6 departs from that, since a local removal commit would be pushed over origin by the next push.
   The prime weft reaches origin's state of the record through D5, and removing origin's copy stays with `lyx batten cleanup` on the machine that landed it.
3. Reports the entry as landed (`done`) or removed from the board (gone), with every tag.

A pair whose driver is busy, or whose detached push holds its teardown lock, is reported **busy**, not torn down.
An ended entry is never resumed on this machine.

One case ends a busy driver: `lyx cleanup --apply`'s teardown of an ended entry holding a pair pause entry wake wrote (D8), one whose caller is the hibernate daemon's, runs without `RefuseWhenBusy`, as batten's own teardown does, since an ended entry's driver has nothing left to land.
A driver the pause interrupted mid-step usually leaves its task worktree dirty, and `EndSession` probes the removal refusals before it ends the driver, so this case passes force, which answers the dirtiness checks alone: a merge in progress still refuses.
Once the driver is ended, and before it tries the teardown lock or removes anything, the local mode commits every uncommitted path of the task worktree, and every one of the weft worktree outside the record pathspec, untracked ones included, into a commit on no branch, keeps it under a local `set-aside/<slug>/<sha>` tag and reports those paths, so force discards nothing the tag does not keep but ignored files, which every removal discards.
The snapshot builds each commit through a temporary index, so it moves no branch and touches neither worktree's index, and holds the weft write lock, taken by one try-acquire of its own and released once the tag is written, so a held lock reports the pair busy with nothing removed, as `commitPendingRecords`' later try-acquire does.
The case also covers an ended entry whose driver an earlier run of it already ended, which that snapshot tag marks, since that run lifted the pair pause entry; each run takes a fresh snapshot before it passes force.
Once the snapshot tag is written it lifts that pair pause entry, so a snapshot refused busy leaves the entry in place and a re-run still qualifies for the case, so a push parked under it runs to its end, taking D10's gone path where origin dropped the branch; a push still holding the teardown lock reports the pair busy, and `lyx cleanup --apply` run again after the push ends removes it, with the driver already ended.

Bound: the local mode skips only the origin steps, the board write and the store housekeeping, waits only for its own loop kill, every pairteardown refusal still applies outside the one case above, and the commits the pushed archive tag would have carried are kept under the local tag.
Only an ended entry holding a wake-written pair pause entry, or the snapshot tag an earlier run of this case left, has its busy driver ended, its dirtiness answered by force and that entry lifted before removal, its uncommitted paths kept under the local tag; every other pairteardown refusal, a merge in progress and a detached push holding the teardown lock included, still applies to it.

### D7. The outcome classifier is one switch

Each judged entry gets exactly one entry outcome from one function: `active`, `ended`, or `stopped` (a diverged, rewound, rewritten, dirty, gone or busy branch of it, a live pair whose branch origin moved past or diverged from, or a D3 stop).
`ended` takes precedence over every branch cause of `stopped`, so an ended entry whose driver or branch is busy is `ended` with a busy flag; a D3 stop still stops every scoped entry.
`machine-handover` adds its owner outcome as one more case, ahead of `active`.

### D8. What each call site does with the report

- **`lyx orch start`**: runs the hub reconcile before launching the session, prints the report and the outstanding failed-push records on its envelope and passes a summary of both into the orch's start note.
  It never refuses, since the orch is the session that resolves a stop; a run already driving is reported live, or stopped when origin moved its branch, and acting on it is the orch's.
  Under an operator's live hub pause its reconcile waits at the network gate until the lift, as every other network call of the hub does, and the gate's wait line names the pause; `lyx cleanup` waits the same way.
- **Wake from `lyx hibernate`**: the daemon runs the hub reconcile after the wake and before it lifts its pause entry, so each paused pair is judged and moved as a quiet one (D4), so a push parked under the pause resumes only after it (D10).
  The reconcile's network calls are exempt from the hub pause entry the daemon's own request owns: the daemon passes the network gate that entry's reference for the reconcile's duration, and the gate lets a call of the daemon's process pass that entry alone.
  Bound: only the reconcile's two ls-remote calls, repeated by the retry below, and its fetches pass, every other process and every other entry stays held until the lift, and the exemption ends with the reconcile.
  A hub entry the request does not own is never passed, since an operator's hub pause holds the hub's network calls: when another hub entry is live at the reconcile's start, as when the request found an operator's hub pause live and so owns none, or its owned entry was lifted and another hub entry added, the wake reconcile is skipped, the daemon lifts only an entry its request owns, and the notice names the standing entry, `lyx resume` as its way forward, and the reconcile `lyx orch start` or `lyx cleanup --apply` runs after that lift.
  With no hub entry live at all the reconcile runs with nothing to pass.
  A woken machine often has no network yet, since the daemon detects the wake from the clock gap and waits for no connectivity, so the wake reconcile retries a failed ls-remote pair, both calls again together, every 10 seconds within a fixed bound of 2 minutes, each attempt under its own 20-second deadline so a stalled connect or DNS lookup cannot outlast the bound, all three values injectable.
  Still unreachable at the bound, the reconcile moves nothing, the daemon lifts its owned entry as it does today, and the notice names `unreachable` and `lyx cleanup --apply` as the reconcile to run once the network is back, which moves Bolt, the prime and every quiet pair, and reports a pair whose driver the lift resumed as live; such a pair is caught up by its own run-scope reconcile at Merge-In or Finalize.
  A D3 stop keeps the hub entry live and the notice names the stop and `lyx resume` as the way forward once settled, before any `lyx orch start` or `lyx cleanup --apply`, whose reconcile waits at the gate while that entry stands.
  Otherwise each `stopped` entry gets a plain pair pause entry, not owned by the request, so it outlives the request and the hub entry's lift; the notice names, for each of those runs, the way forward of its stop and then `lyx resume <slug>`, and lists the ended entries.
  A pair stopped busy because a parked push holds its weft's push lock has no stop of its own to settle: its way forward is `lyx resume <slug>`, after which the push meets D10's retry.
  An ended entry with a busy flag gets such a pair pause entry too, and its notice names the landing or removal and `lyx cleanup --apply`, whose teardown of that entry ends its driver under the hold and keeps a dirty worktree's uncommitted paths under a local `set-aside/` tag before it removes the pair (D6), never `lyx resume <slug>`.
  D6's teardown of that pair lifts the pair pause entry wake gave it.
- **Batten resume** (`lyx batten run <slug>` over an existing run record): runs the run-scope reconcile first; `ended` refuses naming the landing or removal, `stopped` refuses naming the branch and the way forward, `active` resumes.
  A fresh start runs no reconcile: `Add` probes origin for the pair, and a parent behind origin costs only Merge-In's merge.
- **Merge-In and Finalize**: Merge-In runs the run-scope reconcile before taking the parent lock, and Finalize runs it at entry under the parent lock the run already holds; `ended` is a Stuck with Halt naming the landing or removal, and `stopped` is a Stuck naming the branch and the way forward.
- **`lyx cleanup`**: runs the hub reconcile before its segments.
  Without `--apply` the reconcile runs judge-only: both ls-remote calls, judging against the heads they return, and a fetch of the moved heads' objects into no ref (`git fetch --refmap= --no-write-fetch-head` with no destination), then no move, no replay and no teardown, and its report is printed.
  Bound: the dry run moves no ref, tracking refs included.
  Only `--apply` runs the moving reconcile, and its segments then compute their candidates from the reconciled state, so they may list fewer than the dry run did; the dry run's report names each move and teardown that accounts for the difference.
  With `--apply` a D3 stop refuses the verb with the report; the dry run, which replays nothing, reports a diverged Bolt or prime records branch as needing a replay and refuses nothing.
  A pair stop is only reported.

These are the only callers.
A driver revival, a parked driver's resume through `lyx loom start` and `lyx resume`'s lift act on a run this machine drives without having been away, and stay as they are; a manual lift of a held pair is the operator's call.

### D9. `Fabric.Pull` brings the weft level or says why not

The weft arm calls D5's helper when a fast-forward is not possible.
The warp arm still runs after a weft stop, as today, and the pull then returns an error naming the weft branch, the conflicting paths and the way forward, never `ok: true` with `weft_pulled: false`.
The doc's manual `git -C <weft> reset --hard origin/<branch>` recovery is removed.

The warp arm's unpushed test, today `@{u}..HEAD` read before the pull's own fetch, becomes D4's: after the fetch, a local commit counts as unpushed only when no value of the tracking ref's reflog, read as D4's **rewound** test reads it, reaches it.
Otherwise the reconcile's own fetch, which already moved the tracking ref to a rewritten head, would make every commit of the earlier value read as unpushed, and the pull would refuse the very **rewritten** branch D4 sends to it with `ErrWarpDivergedUnpushed`.
A pair's task branch D4's test reads as **rewound** is refused before the reset, naming `ResetPairCode`'s way forward and changing nothing, so the pull never settles a rewind by itself and the next reconcile still reports it.
The prime's warp branch, which `ResetPairCode` never touches, takes no such refusal, so a pull of the prime, Land's included, resets a rewind of it as it does today.
A **rewritten** branch then takes the existing anchor walk and reset, and a local commit no reflog value reaches still refuses with `ErrWarpDivergedUnpushed`, as today.
Batten's Land row, which pulls the prime after a landing, deploys after a weft stop as it does today and names the stop in its landing notice to the orch.

Bound: the pull never discards a non-empty local records commit, except an ended entry's commits D5 sets aside, which stay under their local tag, and its warp arm resets away only commits a value of the tracking ref's reflog reaches, never one origin never held, and never a rewound task branch.

### D10. A rejected push retries, then is recorded and reported

Every push fabricengine runs (the detached coalesced push, `PushAnchored`, `PushPairAnchored`, Bolt's pushes and syncs) classifies a failed push:

- **not fast-forward**, records side: fetch, Bolt's branch included (D5), D5's replay, push again, once.
  The retry try-acquires the records write lock while it holds the push lock, never waits for it; a Bolt retry, which holds `board.push.lock`, try-acquires `board.lock` and then takes `.gitrepo-push.lock`.
  A held write lock or D5's dirtiness refusal is a deferral, not a failure: the holder writes no pending flag of its own side and ends its loop, leaving the commits to the next commit's push, and writes the failed-push record with the class `deferred` and no mail, which the next landed push of the branch removes.
  The way forward of `deferred` is `lyx fabric push` in the branch's worktree, for when no later commit comes.
  A replay that stops, or a second rejection, is a failed push.
- **not fast-forward**, code side: a failed push at once.
- **bare `(failed)` refusal or transport failure**: the existing backoff; still failing at the end is a failed push.
- **auth or permission refusal**: a failed push at once, no retry.
- **gone**: a branch whose upstream names an origin branch that no longer exists is not pushed, so a parked push never recreates a branch landed or deleted elsewhere; a failed push, after the backoff below where its entry is not yet ended on origin.
  Every push of a weft or a task branch with an upstream first runs one `git ls-remote origin` of that branch, since the local tracking ref is only as fresh as the last fetch, which costs one round trip per push; Bolt's pushes skip it, since Bolt's branch is never deleted.
  A transport failure of that ls-remote is classed as the push's own transport failure, under the existing backoff.
  A push parked under a pause carries its pause slug and resumes only at the lift, which at wake the daemon makes after the wake reconcile for an entry its request owns, and the operator's `lyx resume` makes for any other, the wake reconcile then skipped (D8).
  Before judging ended membership, the gone path fetches origin's Bolt head, one fetch of Bolt's branch, since nothing else on it refreshes Bolt's tracking ref, and a transport failure of that fetch is classed as the push's own transport failure, as the ls-remote's is; a gone branch of an entry in the ended set D3 step 2's rule reads from that tip and its own history writes no failed-push record and is left to D6's teardown.
  A gone branch whose entry that tip shows neither done nor removed is classed as the push's own transport failure: the landing machine deletes the branch before it settles the board claim, and Bolt's push of that claim is detached, so origin can lose the branch before it shows the entry done.
  The existing backoff re-runs the ls-remote, the Bolt fetch and this judgment, and only a branch still gone with its entry neither done nor removed at the end of the backoff writes the failed-push record and mails; where origin's done mark lands later still, the next reconcile finds the entry ended and D6's teardown removes the record.

A failed push writes a failed-push record, one file per repo and branch, under the hub's board-level `.lyx` directory, holding the branch, the side, the class, git's last error line, the first and last time seen, the count and a `notified` flag.
Every failure of a branch updates its record under the record's lock, creating it on the first; while `notified` is unset, the updater sends one creel mail to the orch batten's notices reach, and sets `notified` only once the mail was posted.
The mail is signed as the program sender `FABRIC`, new with this task, or as the calling strand when the push runs in an agent pane, as `orchcli.NotifyPrime` does.
A `deferred` record sends no mail.
A push of that branch that lands removes the record, and so do a pair's teardown and a reconcile that finds the branch no longer local.

`lyx fabric status`, `lyx tender status` and `lyx cleanup` print the outstanding records with the way forward for their class.
`lyx cleanup --apply`'s push result reports the outstanding records instead of an empty `push_error` for a push it only spawned.

Bound: the retry replays only records branches, once per push, and never waits on the write lock; a deferral costs at most one rejected push per commit; the notice is one posted mail per branch per episode, so a read-only account sends one mail per branch, not one per push, and an episode that starts while no orch is recorded mails at its first failure after one is.
Rationale: a record outlives the detached child, and the mail reaches the session that acts; it answers `fabric-remote-async`'s open question of how an async remote failure reaches the orch.

### D11. Preflight's records push goes detached

The Preflight row's records push becomes the pair's detached push, as every other status transition's push is.

Rationale: nothing reads the remote weft branch until Publish, and D10 makes a refused detached push visible, which removes the reason to wait on it.

### Added after Discussion (arbiter, 2026-10-21): Rewound keep-local choice is a plain push

Decision: D4's second way forward for a task branch rewound on origin is `git push origin <branch>` in the task worktree, not the `ours` merge of `origin/<branch>`; `lyx fabric push` stays refused until the rewind is settled by that push or by `git reset --keep <origin head>`.

Rationale: A branch reads rewound only when origin's head is already an ancestor of the local tip, so the `ours` merge changes nothing and the hold-back still reads the tip rewound. The plain push is the fast-forward D4 meant the merge to enable, and it writes the push entry that ends the reflog test. Plan review round 4, F1.

## Constraints

- PATTERN-fabric-git: every git call of the reconcile, the replay and the push classification is in `internal/fabricengine`; other packages call it in-process, and reach it back only through D1's seams.
- PATTERN-fabric-destruction-chokepoint: the replay's abort and restore go through `destroy.go`'s executors, as `dropSeedCommits` does today.
- PATTERN-pair-teardown: the ended-entry removal goes through `internal/pairteardown` only.
- PATTERN-batten-bookend: the run scope never tears down its own pair (D6).
- PATTERN-parent-lock: the reconcile never merges into or lands on a parent branch, moves the prime's branches only while the parent lock record names no live owner, and at run scope never moves the parent (D4).
- PATTERN-told-geometry: battenshed, landingshed and hibernateengine receive the reconcile as an injected seam over told paths.
- PATTERN-durable-vs-ephemeral-state: the failed-push record is machine-local and nothing a run needs, so it lives under `.lyx`, declared in `internal/lyxdirs`.
- PATTERN-tolerant-records: the failed-push record is written through `internal/state` and refuses no unknown key.
- PATTERN-creel-store and PATTERN-orch-pane-single-writer: the push notice is creel mail, never typed into the orch pane.
- PATTERN-refusal-way-forward and PATTERN-no-denied-recovery: every stop, refusal and failed-push class names an allowed way forward, and a test reaches each.
- PATTERN-mutation-record: the reconcile fills a `*Mutations` record and its result embeds `MutationRecord`.
- Git's reflog of remote-tracking refs: D4's rewound and rewritten tests read it, so they rely on `core.logAllRefUpdates`, on by default in a non-bare repo, and on the entries outliving `gc.reflogExpireUnreachable`; a rewind left unsettled past that expiry loses its evidence.

## Auto-mode assumptions

- An unreachable origin (a transport failure of either ls-remote) is the outcome `unreachable`, warned on the envelope or notice, and every call site carries on: refusing would block all offline work, and D10 still reports the later rejected push; Merge-In and Finalize then fail on their own remote steps as today.
  Wake is the one site where this resumes drivers on an unverified view, after D8's 2-minute retry: a resumed driver commits locally and pushes only fast-forwards, apart from `ResetPairCode`'s leased update, whose lease on the pre-sleep tracking ref refuses once origin moved, so a push over a moved origin meets D10's replay, which stops on a conflicting `status.json`, or is a failed push on the code side, never an overwrite; its Merge-In and Finalize run their own run-scope reconcile before acting on the parent; and what it can still do on stale records is the first Open risk's replay of untouched paths.
- The set-aside tags of D5 and D6 stay local.

## Open risks

- Until `machine-handover`, a stale driver on a woken machine that commits only paths the other machine left untouched gets them replayed and pushed; D5 relies on `status.json` conflicting within one transition to stop it.
- Right after a handover, replays may stop on many stale status files, each needing a human, until `machine-handover` lands.
- GitHub's error text is the only signal for the auth class; a wording change moves a 403 into the transport class, recorded only after the backoff.
- Until `run-process-ownership` ends orphaned driver chains and parked pushes, an ended entry's pair at wake is usually busy and stays local, held by a pair pause; `lyx cleanup --apply` ends its driver and releases its parked push (D6), and a second run removes the pair once that push ends.
- `tender-pause-holds-reprompt`: a paused pair still takes gate and driver re-prompts, so D8's pair hold is weaker until that task lands, and a re-prompted writer mid-step when the wake reconcile moves its paused pair (D4) is caught only by the dirty check of what is already on disk.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 4: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/discussion/round-4-review.md; fixer report: _lyx/reviews/discussion/round-4-fixer-report.md.
  - records-lock-waits-unbounded (class design, severity MEDIUM, open in rounds 2, 3, 4)
  - rewritten-way-forward-pull-refuses (class design, severity BLOCKING, open in rounds 4)
  - wake-unreachable-origin-lifts-pause (class decision, severity MEDIUM, open in rounds 4)
<!-- /lyx:carry-over Discussion-Review -->

## Acceptance criteria

- Two clones of one origin: origin moved on a weft, Bolt and warp `main` with nothing local only; a hub reconcile fast-forwards all three and reports each.
- A pair worktree whose weft origin moved, clean on the touched paths, is fast-forwarded; a tracked change on a touched path stops it as dirty with nothing moved.
- A live pair under hub scope is reported live and not moved, its entry `active` while its branches are level or ahead and `stopped` once origin moved past or diverged from one; the prime with its prime lock or parent lock held is reported busy at once and not moved.
- Run scope at Merge-In with the parent behind origin judges and does not move the parent.
- A records branch with a local status commit and newer unrelated origin commits is replayed, the empty commit dropped; a `status.json` changed on both sides stops as diverged, tip restored, way forward named.
- Two machines each with one Bolt status commit of a different entry: the Bolt replay succeeds with no stop and `README.md` re-rendered; the same field of one entry changed on both sides keeps origin's value and reports the lost local value under a `set-aside/` tag.
- A code branch diverged stops and is never rebased.
- A task branch rewound on origin by another machine is reported rewound, stops its entry and names `ResetPairCode`'s way forward, with or without an unpushed local commit, and both before and after its tracking ref was fetched to origin's head, where it would otherwise read as ahead.
- A branch whose origin head is an ancestor of an earlier tracking-ref value, while its local tip holds none of that value's commits origin's head lacks and neither tip is an ancestor of the other, is reported diverged, neither rewound nor rewritten.
- A warp upstream with no unpushed commit, rewritten to a head that is no ancestor of any earlier tracking-ref value, is reported rewritten and names `lyx fabric pull`.
- After a moving reconcile reported a pair rewritten, `lyx fabric pull` in its worktree takes the anchor walk and reset and ends with the branch at origin's head and the weft re-anchored; the same pull over a rewound task branch refuses naming `ResetPairCode`'s way forward and changes nothing, and over a local commit no reflog value reaches refuses with `ErrWarpDivergedUnpushed`.
- A task branch rewound on origin, then fetched by `lyx cleanup` without `--apply` or by `lyx fabric pull`, is still reported rewound by the next reconcile, and its push does not restore the dropped commits.
- A local Bolt body edit of an entry origin holds under a run status is set aside and reported with the run lock's refusal, and the replay does not stop.
- One local Bolt commit changing an entry origin holds under a run status and an unrelated entry: only the run-held change is set aside, and the unrelated change lands on Bolt.
- A Bolt replay under the reconcile's `board.lock` and `.gitrepo-push.lock` completes without waiting on either and spawns no sync.
- Neve's case: local Bolt and prime-weft status commits of an entry ended on origin are set aside under local `set-aside/` tags, the replay of the rest succeeds, and `lyx report` lists no run for them.
- An entry only local Bolt holds, with a parked pair, is not ended and its pair is not torn down.
- An entry pruned on origin, with local Bolt fast-forwarded past the removal by a pull-first before the reconcile runs, is still ended.
- An ended entry with a local pair, local-only weft commits and pending records: pending records committed, tagged locally, pair removed with no push and no board write, resume refused naming the landing.
- The same entry with a busy driver or a parked push is reported ended and busy and not removed; at wake its pair pause notice names `lyx cleanup --apply`, not `lyx resume <slug>`.
- After wake, `lyx cleanup --apply` removes the ended pair whose driver strand is live under its wake-written pair pause entry, its task worktree dirty: the uncommitted paths are kept under a local `set-aside/` tag and reported; where a parked push held the teardown lock, the run reports busy and a second run after the push ends removes the still-dirty pair.
- An ended pair whose teardown lock or weft write lock is held is reported busy at once, with no wait.
- Wake with the hub pause live completes the reconcile and lifts the hub entry.
- Wake under an operator's hub pause, which the request does not own, skips the reconcile without waiting at the gate, moves nothing and leaves that entry live, the notice naming the entry and `lyx resume`.
- Wake with a parked push holding a paused pair's push lock reports that pair busy and lifts the hub entry.
- A run whose local history shows Finalize entered is not ended; Finalize re-entered after its done mark proceeds.
- A prime-weft push rejected right after this machine's Finalize pushed `done` keeps the landing's commits.
- A pair with no batten run record and no board entry is never removed.
- A Bolt stop, a tracked change on a path the replay touches, stops every scoped entry; at wake the hub pause stays live.
- Merge-In of a run ended on origin, with Finalize not entered, is a Stuck with Halt naming the landing.
- Wake with a paused run whose weft moved on origin fast-forwards it, or gives it a pair pause entry when it cannot; that entry outlives the request, and the notice names `lyx resume <slug>`.
- Batten resume over an active run reconciles and resumes; a fresh start runs no reconcile.
- `lyx cleanup --apply`: a Bolt or prime-weft stop refuses the verb; a run deleted on origin's records is not listed as paused afterwards.
- `lyx cleanup` without `--apply` prints the reconcile's report, reports a diverged Bolt as needing a replay without refusing, and moves, replays and tears down nothing.
- `lyx fabric pull` with a conflicting local weft commit pulls warp and fails with the way forward; with only commits that replay empty it succeeds.
- Land after a weft stop deploys and names the stop in its notice.
- A detached push rejected as not fast-forward on a weft is fetched, replayed and pushed; with the write lock held it defers, writes a `deferred` record with no mail and sets no pending flag, and the next commit's push lands it and removes the record.
- A 403 writes one record and one mail; a concurrent second failure updates it and sends no mail; a landed push or the pair's teardown removes it.
- A 403 while no orch is recorded writes the record with `notified` unset, and the first failure after an orch is recorded mails; `lyx orch start` prints the outstanding record.
- A parked push of a branch gone on origin does not recreate it; one of an ended entry writes no record.
- A parked weft push of a run whose entry became done on origin since this machine's last Bolt fetch writes no failed-push record.
- A parked weft push whose branch origin deleted before its Bolt shows the entry done backs off, and writes no record once the done mark arrives within the backoff.
- `lyx fabric status`, `lyx tender status` and `lyx cleanup` print an outstanding failed push with its way forward.
- An unreachable origin: every call site carries on with an `unreachable` warning.
- Wake with origin unreachable at first: the reconcile retries within its bound and runs once origin answers; unreachable through the whole bound, it moves nothing, the hub entry is lifted, and the notice names `unreachable` and `lyx cleanup --apply` as the reconcile to run once the network is back.
- An ended pair whose hub workspace lock is held is removed without waiting on it, the regeneration skipped with its warning, and the local mode runs no store gc.
- The Preflight row spawns its records push detached and does not wait on it.
- Each reconcile with a reachable origin makes exactly two ls-remote calls and fetches only moved heads.

## PATTERN changes

- `PATTERN-origin-reconcile` — adds: Acting on a run or the hub after this machine may have been away (orch start, wake, batten resume, Merge-In, Finalize, cleanup): first `fabricengine`'s origin reconcile, the sole judge of local branches against origin.
- `PATTERN-push-failure-visible` — adds: A push origin still refuses after its retry: a failed-push record and one creel notice to the orch, never only a log line. (test)

## Notes for the plan writer

- The hibernate daemon reaches the reconcile and the pair pause through its `Deps` seams, as it reaches tender today.
- The folded note's trace (`creel-c13-stacked-rewrite`, 2026-10-17) places the synchronous push in the Preflight row's records commit; find the call that pushes there.
