# Decision record — origin-reconcile

## Goal

A machine never acts on a stale view of origin, and a push that origin refuses is never silent.
Before every step where a stale view does harm, one origin reconcile compares this machine's branches and Bolt statuses with origin and fast-forwards, cleans up a run that landed elsewhere, or stops naming the diverged branch.
A rejected push recovers by itself where it safely can, and otherwise reaches the orch and stays visible until it is settled.
Two places act on an unverified view on purpose, each bounded in its decision: an unreachable origin (D8) and the orch start, which reports a stop and never refuses (D8).

## Scope

In:

- The origin reconcile in `internal/fabricengine`: two parallel `git ls-remote origin` calls (warp repo, records repo), one fetch per repo of only the moved heads, and the outcomes of D4 and D7.
- Its call sites: `lyx orch start`, the hibernate daemon's wake, batten's resume of an existing run, the Merge-In and Finalize rows, and `lyx cleanup`.
- A records replay that replays local commits onto origin and drops those that become empty, shared by the reconcile, `Fabric.Pull` and the push retry.
- `lyx fabric pull` failing, with a way forward, when the weft cannot be brought level with origin.
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
It runs `git ls-remote origin` once for the warp repo and once for the records repo, in parallel, then fetches only the heads that differ from the local tracking refs, in one fetch per repo.
Its Bolt step reuses that fetch, never `Bolt.pullFirst`'s own.
Nothing else in the reconcile touches the network.

Rationale: PATTERN-fabric-git puts git on warp and weft, Bolt included, in fabricengine alone, and the board entry fixes two calls per reconcile.
It is the synchronous remote operation `fabric-remote-async` allows: a check of what exists on origin, and the fetch of what was found there.

### D2. Scope: the hub, or one run

- Hub scope (orch start, wake, `lyx cleanup`): Bolt, the prime's warp and weft branches, and every local pair's warp and weft branches.
- Run scope (batten resume, Merge-In, Finalize): Bolt, the run's parent branch on both sides, and the run's own pair.

Both scopes make the same two ls-remote calls; the scope only bounds which branches the reconcile judges.

### D3. Order of work

1. Both ls-remote calls in parallel, then the fetches.
2. The ended set: every slug whose entry on the fetched origin Bolt tip is `done` or gone, read from origin's tip and not from local Bolt, so it is known before any replay.
3. Bolt: fast-forward, or the records replay of D5.
4. The prime's branches, then each scoped pair, judged per D4, with D6 applied to an entry before its branches are judged.

A stop on Bolt or on the prime's records branch stops every scoped entry: each gets the entry outcome `stopped` with that cause, so no call site acts on an entry the reconcile could not judge.

Rationale: judging runs from a stale Bolt is the failure the task fixes, and the replays of steps 3 and 4 need the ended set first.

### D4. Per-branch outcomes and who may move what

For each scoped local branch with an origin tracking ref:

- **level**: nothing to do.
- **fast-forward**: the local tip is an ancestor of origin's head; the branch moves to it.
- **ahead**: origin's head is an ancestor of the local tip; the next push carries it.
- **diverged**: neither is an ancestor.
  A records branch (a weft or Bolt) first gets D5's replay and is diverged only when that replay stops; a code branch is diverged at once, and is never rebased.
- **gone**: the local branch's upstream no longer exists on origin; the reconcile recreates nothing.

A branch moves only where no live writer can meet it:

- A branch checked out in a worktree moves by a fast-forward of that worktree; a tracked change on a path the move touches stops it as **dirty**, naming the paths.
- The prime's branches move only while the reconcile holds the prime lock and the prime branch's parent lock, each with batten's bounded wait; a held lock leaves them **busy**, judged and reported but not moved, since the landing or Land row holding it pulls the prime itself.
- The run scope never moves the parent branch: it judges it, and Merge-In's existing fast-forward under the parent lock moves it.
- A pair whose run lock or loop lock is held, or whose driver strand is live, outside the run scope's own pair, is judged and reported **live**, never moved.
- A records branch moves under its write lock, then its push lock, the order `PullThenCommitWritten` holds on Bolt, applied to every weft too.

Bound: the reconcile moves a local branch only forward to origin's head or by D5's replay; it never resets away a commit origin lacks, never force-pushes, never deletes or recreates a remote branch, and never moves a branch a live run, a landing or a held lock is using.

### D5. Records replay: replay, drop empties, stop on conflict

One fabricengine helper brings a records branch level with origin when both moved.
It first sets aside the local commits that belong to an ended entry: on a weft, a commit whose every path lies under an ended run's record directory; on Bolt, a commit whose every changed entry file is an ended entry's.
It replays the other local commits origin lacks onto origin's head, oldest first, drops one that becomes empty and keeps each other one.
A conflict, such as a `status.json` both sides changed, aborts the replay and restores the starting tip; the branch is diverged, and the stop names the branch, the conflicting paths and the way forward.
Each set-aside commit is kept under a local `archive/<slug>/<sha>` tag and reported.
It refuses a tracked change on a path the replayed commits touch.

The reconcile, `Fabric.Pull` (D9) and the push retry (D10) call it, and Bolt's push recovery and seed-commit drop become cases of it, replacing `gitrepo`'s pull-rebase on every fabric push path.

Bound: it replays only records branches, only commits origin lacks, keeps every non-empty commit it does not set aside, sets aside only an ended entry's commits, and never rewrites a commit origin holds.
Rationale: this reconciles the board entry's "diverged: stop" with #737's "rebase the weft, stop only on a real conflict": a records divergence is a stop only when the replay meets a conflict.
A second machine driving the same run conflicts on its `status.json` within one transition, since every transition writes it, so concurrent driving still stops.

### D6. Ended entries

An entry is ended on this machine when it is in D3's ended set and this machine holds a batten run record of that slug, judged from the local records before any move.
A pair with no batten run record is never classified, only its branches judged.
Its own landing is not ended: a run whose local status history shows Finalize entered is landing here, and batten's own rows finish it.
The run scope never applies this decision to its own run, whose driver is the caller.

For an ended entry the reconcile:

1. Removes the pair through `internal/pairteardown` with `RefuseWhenBusy`, in the local mode: Remove commits the pending records as today, then puts the archive tag on the weft tip locally instead of pushing it, and runs no other origin step and no board write.
   A warp branch Remove keeps for unlanded work is reported with Remove's kept reason.
2. Writes no commit of its own: the board entry asks for the run record to be removed locally, and D6 departs from that, since a local removal commit would be pushed over origin by the next push.
   The prime weft reaches origin's state of the record through D5, and removing origin's copy stays with `lyx batten cleanup` on the machine that landed it.
3. Reports the entry as landed (`done`) or removed from the board (gone), with every tag.

A pair whose driver is busy, or whose detached push holds its teardown lock, is reported **busy**, not torn down.
An ended entry is never resumed on this machine.

Bound: the local mode skips only the origin steps and the board write, every pairteardown refusal still applies, and the commits the pushed archive tag would have carried are kept under the local tag.

### D7. The outcome classifier is one switch

Each judged entry gets exactly one entry outcome from one function: `active`, `ended`, or `stopped` (a diverged, dirty, gone or busy branch of it, or a D3 stop).
`machine-handover` adds its owner outcome as one more case, ahead of `active`.

### D8. What each call site does with the report

- **`lyx orch start`**: runs the hub reconcile before launching the session, prints the report on its envelope and passes a summary into the orch's start note.
  It never refuses, since the orch is the session that resolves a stop; runs already driving are judged live and reported, and acting on them is the orch's.
- **Wake from `lyx hibernate`**: the daemon runs the hub reconcile after the wake and before it lifts its pause entry.
  A D3 stop keeps the hub entry live and the notice names the stop and `lyx resume` as the way forward once settled.
  Otherwise each `stopped` entry gets a plain pair pause entry, not owned by the request, so it outlives the request and the hub entry's lift; the notice names those runs with `lyx resume <slug>` as their way forward, and lists the ended entries.
- **Batten resume** (`lyx batten run <slug>` over an existing run record): runs the run-scope reconcile first; `ended` refuses naming the landing or removal, `stopped` refuses naming the branch and the way forward, `active` resumes.
  A fresh start runs no reconcile: `Add` probes origin for the pair, and a parent behind origin costs only Merge-In's merge.
- **Merge-In and Finalize**: run the run-scope reconcile before taking the parent lock; `stopped` is a Stuck naming the branch and the way forward.
- **`lyx cleanup`**: runs the hub reconcile before its segments; a D3 stop refuses the whole verb with the report, and a pair stop is only reported.

These are the only callers.
A driver revival, a parked driver's resume through `lyx loom start` and `lyx resume`'s lift act on a run this machine drives without having been away, and stay as they are; a manual lift of a held pair is the operator's call.

### D9. `Fabric.Pull` brings the weft level or says why not

The weft arm calls D5's helper when a fast-forward is not possible.
The warp arm still runs after a weft stop, as today, and the pull then returns an error naming the weft branch, the conflicting paths and the way forward, never `ok: true` with `weft_pulled: false`.
The doc's manual `git -C <weft> reset --hard origin/<branch>` recovery is removed.
Batten's Land row, which pulls the prime after a landing, deploys after a weft stop as it does today and names the stop in its landing notice to the orch.

Bound: the pull never discards a non-empty local records commit, and its warp arm is unchanged.

### D10. A rejected push retries, then is recorded and reported

Every push fabricengine runs (the detached coalesced push, `PushAnchored`, `PushPairAnchored`, Bolt's pushes and syncs) classifies a failed push:

- **not fast-forward**, records side: fetch, D5's replay, push again, once.
  The retry try-acquires the records write lock while it holds the push lock, never waits for it; a held write lock or D5's dirtiness refusal is a deferral, not a failure: the pending flag stays set and the next push retries.
  A replay that stops, or a second rejection, is a failed push.
- **not fast-forward**, code side: a failed push at once.
- **bare `(failed)` refusal or transport failure**: the existing backoff; still failing at the end is a failed push.
- **auth or permission refusal**: a failed push at once, no retry.
- **gone**: a branch whose upstream names an origin branch that no longer exists is not pushed, so a parked push never recreates a branch landed or deleted elsewhere; a failed push.

A failed push writes a failed-push record, one file per repo and branch, under the hub's board-level `.lyx` directory, holding the branch, the side, the class, git's last error line, the first and last time seen and the count.
The process that creates the record, by an exclusive create, sends one creel mail to the orch batten's notices reach, signed as a new program sender `FABRIC`; a later failure of the same branch updates the record and sends nothing.
A push of that branch that lands removes the record, and so do a pair's teardown and a reconcile that finds the branch no longer local.

`lyx fabric status`, `lyx tender status` and `lyx cleanup` print the outstanding records with the way forward for their class.
`lyx cleanup --apply`'s push result reports the outstanding records instead of an empty `push_error` for a push it only spawned.

Bound: the retry replays only records branches, once per push, and never waits on the write lock; the notice is one per branch per episode, so a read-only account sends one mail per branch, not one per push.
Rationale: a record outlives the detached child, and the mail reaches the session that acts; it answers `fabric-remote-async`'s open question of how an async remote failure reaches the orch.

### D11. Preflight's records push goes detached

The Preflight row's records push becomes the pair's detached push, as every other status transition's push is.

Rationale: nothing reads the remote weft branch until Publish, and D10 makes a refused detached push visible, which removes the reason to wait on it.

## Constraints

- PATTERN-fabric-git: every git call of the reconcile, the replay and the push classification is in `internal/fabricengine`; other packages call it in-process.
- PATTERN-fabric-destruction-chokepoint: the replay's abort and restore and the local tags go through `destroy.go`'s executors, as `dropSeedCommits` does today.
- PATTERN-pair-teardown: the ended-entry removal goes through `internal/pairteardown` only.
- PATTERN-batten-bookend: the run scope never tears down its own pair (D6).
- PATTERN-parent-lock: the reconcile moves a parent branch only under its parent lock, and the run scope never moves it (D4).
- PATTERN-told-geometry: battenshed, landingshed and hibernateengine receive the reconcile as an injected seam over told paths.
- PATTERN-durable-vs-ephemeral-state: the failed-push record is machine-local and nothing a run needs, so it lives under `.lyx`, declared in `internal/lyxdirs`.
- PATTERN-tolerant-records: the failed-push record is written through `internal/state` and refuses no unknown key.
- PATTERN-creel-store and PATTERN-orch-pane-single-writer: the push notice is creel mail, never typed into the orch pane.
- PATTERN-refusal-way-forward and PATTERN-no-denied-recovery: every stop, refusal and failed-push class names an allowed way forward, and a test reaches each.
- PATTERN-mutation-record: the reconcile fills a `*Mutations` record and its result embeds `MutationRecord`.

## Auto-mode assumptions

- An unreachable origin (a transport failure of either ls-remote) is the outcome `unreachable`, warned on the envelope or notice, and every call site carries on: refusing would block all offline work, and D10 still reports the later rejected push; Merge-In and Finalize then fail on their own remote steps as today.
- The archive tags of D5 and D6 stay local.

## Open risks

- Until `machine-handover`, a stale driver on a woken machine that commits only paths the other machine left untouched gets them replayed and pushed; D5 relies on `status.json` conflicting within one transition to stop it.
- Right after a handover, replays may stop on many stale status files, each needing a human, until `machine-handover` lands.
- GitHub's error text is the only signal for the auth class; a wording change moves a 403 into the transport class, recorded only after the backoff.
- Until `run-process-ownership` ends orphaned driver chains and parked pushes, an ended entry's pair at wake is usually busy and stays local, held by a pair pause.
- `tender-pause-holds-reprompt`: a paused pair still takes gate and driver re-prompts, so D8's pair hold is weaker until that task lands.

## Acceptance criteria

- Two clones of one origin: origin moved on a weft, Bolt and warp `main` with nothing local only; a hub reconcile fast-forwards all three and reports each.
- A pair worktree whose weft origin moved, clean on the touched paths, is fast-forwarded; a tracked change on a touched path stops it as dirty with nothing moved.
- A live pair under hub scope is reported live and not moved; the prime with its prime lock or parent lock held is reported busy and not moved.
- Run scope at Merge-In with the parent behind origin judges and does not move the parent.
- A records branch with a local status commit and newer unrelated origin commits is replayed, the empty commit dropped; a `status.json` changed on both sides stops as diverged, tip restored, way forward named.
- A code branch diverged stops and is never rebased.
- Neve's case: local Bolt and prime-weft status commits of an entry ended on origin are set aside under local tags, and the replay of the rest succeeds.
- An ended entry with a local pair, local-only weft commits and pending records: pending records committed, tagged locally, pair removed with no push and no board write, resume refused naming the landing.
- The same entry with a busy driver or a parked push is reported busy and not removed.
- A run whose local history shows Finalize entered is not ended; Finalize re-entered after its done mark proceeds.
- A pair with no batten run record and no board entry is never removed.
- A Bolt conflict stops every scoped entry; at wake the hub pause stays live.
- Wake with a stopped run: a pair pause entry outlives the request, and the notice names `lyx resume <slug>`.
- Batten resume over an active run reconciles and resumes; a fresh start runs no reconcile.
- `lyx cleanup`: a Bolt or prime-weft stop refuses the verb; a run deleted on origin's records is not listed as paused afterwards.
- `lyx fabric pull` with a conflicting local weft commit pulls warp and fails with the way forward; with only commits that replay empty it succeeds.
- Land after a weft stop deploys and names the stop in its notice.
- A detached push rejected as not fast-forward on a weft is fetched, replayed and pushed; with the write lock held it defers and the pending flag stays set.
- A 403 writes one record and one mail; a concurrent second failure updates it and sends no mail; a landed push or the pair's teardown removes it.
- A parked push of a branch gone on origin does not recreate it.
- `lyx fabric status`, `lyx tender status` and `lyx cleanup` print an outstanding failed push with its way forward.
- An unreachable origin: every call site carries on with an `unreachable` warning.
- The Preflight row spawns its records push detached and does not wait on it.
- Each reconcile makes exactly two ls-remote calls and fetches only moved heads.

## PATTERN changes

- `PATTERN-origin-reconcile` — adds: Acting on a run or the hub after this machine may have been away (orch start, wake, batten resume, Merge-In, Finalize, cleanup): first `fabricengine`'s origin reconcile, the sole judge of local branches against origin.
- `PATTERN-push-failure-visible` — adds: A push origin still refuses after its retry: a failed-push record and one creel notice to the orch, never only a log line. (test)

## Notes for the plan writer

- The hibernate daemon reaches the reconcile and the pair pause through its `Deps` seams, as it reaches tender today.
- The folded note's trace (`creel-c13-stacked-rewrite`, 2026-10-17) places the synchronous push in the Preflight row's records commit; find the call that pushes there.
