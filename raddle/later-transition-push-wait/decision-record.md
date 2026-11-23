# Later run transitions still wait for a records push to origin

## Goal

No run transition waits for a push to origin, in batten or loom runs.
Every transition commits its records locally and the run's next step starts at once.
A background pusher brings every commit to origin, in order and with none left out,
and a takeover re-runs the run from what origin holds.

## Scope

In:

- Both transition seams, batten's and loom's, built on `internal/statuscommit`.
- The batten start-row exception of `926b5c28e`, which this design replaces.
- The loom pre-push origin reconcile that runs inside each transition's commit today.
- The run's record commits at driver stops:
  batten's `commitRunRecords` (`internal/battencli/commitstatus.go`) and loom's `commit-records` verb (`internal/loomcli/commitrecords.go`).
- The pusher: its start, its restarts, its failure handling and its hand-over on a takeover.
- Worktree-Teardown's push before it removes a pair.
- `PATTERN-run-fsm`, `PATTERN-pair-remote` and `PATTERN-owner-machine`.

Out:

- Pushes that are a row's own act: PR-Review's push of the task branch, Finalize's parent push, records push and board `done`, the tracked intents and the pull-request record.
- `lym adopt`'s own steps.
- Board writes: a transition's board status write saves locally and spawns a detached sync, so it waits for no push already (`internal/boardengine/board.go`).

## Decisions

### D1 — Operator decision (2026-11-22): every commit reaches origin asynchronously, no step waits

Everything a run does reaches origin in order, with nothing left out, pushed asynchronously; no step waits for a push.
A transition of a batten or loom run commits its records with its `Run-Transition` and `Warp-SHA` trailers and returns.
It pushes nothing, fetches nothing, runs no origin reconcile and takes no network-gate admission (`pauserecord.Admit`).
This holds for every row, skipped rows and batten's start rows included, and for the seeded Worktree-Create commit.
It holds for the run's record commits at driver stops too:
`commitRunRecords` and the `commit-records` verb commit and start the worktree's pusher (D3) instead of pushing,
and the verb's pre-push check moves into the pusher (D7).
The start-row exception of `926b5c28e`, and the marker's `push_pending` forcing a push, retire into this rule.
Worktree-Teardown's push before it removes a pair is the one exception, D5.

Rationale: the operator's decision, relayed by `ly:orch` (creel #2373, #2379).
Bound: a transition no longer proves origin holds the run before the run acts; D3 and D6 carry that duty.

### D2 — Operator decision (2026-11-22): a takeover re-runs from what origin holds

A takeover re-runs the run from what origin holds, and redone work is acceptable.
Only acts that harm outside the run when doubled are guarded, by their own origin checks.
No machinery saves redone work.

Rationale: the operator's decision, relayed by `ly:orch` (creel #2377, #2379).
Bound: rows whose act stays on the machine may run on two machines at once until the original machine hands over (D6).

### D3 — One pusher per records worktree

A pusher is a detached process that pushes one records worktree to origin:
a task pair's records side and then its code branch, or a prime's records side alone.
A batten run commits into the prime's records, so its pusher is the prime's; a loom run's is its pair's.
Glossary line: **pusher**: the background process that brings one worktree's run commits to origin in order.

- It pushes through `pairteardown.PushRun` and so through `fabricengine.PushPairAnchored`, in the mode D5 names:
  the records side first, the code branch only once the records side landed.
- At most one pusher runs per records worktree: it holds a lock named under `.lym` through `lymdirs` and taken through `lock` while it runs.
- After each push it reads the local records tip again and pushes again while the tip is ahead of origin.
- A pusher starts or goes on only while its start conditions hold:
  the worktree's pusher lock is free, no refused-push record stands for its records branch or, for a pair, its code branch (D5),
  and, for a pair, no pause covers its run (D7) and no hand-over mark stands for the run (D6); the checks are local.
- To exit, it releases its lock and then reads the tip once more.
  Finding it ahead with the start conditions holding, it takes the lock again and goes on, or leaves it to a pusher that took the lock first.
  A pusher whose worktree is gone exits.
- The seam starts a pusher after each commit when the start conditions hold.
- The pusher's spawn is logged per `PATTERN-child-processes`.
  It registers in `internal/runproc` under `runproc.HubOwner` with role `runproc.RolePush`, never under the run's slug, which is also a pair's worktree name.
  A run's teardown, cancel or hand-over ends the processes registered under the run's slug, so it never reaches the pusher;
  `runproc.EndSubject` spares `RolePush` in its pass over the subject's own processes.

Rationale: one pusher per worktree pushes the latest tip, so a burst of skipped rows costs one push, not one per row;
keying it by worktree covers batten's records in the prime and a loom run's in its pair, which a slug key mixes up.
Bound: a pusher pushes only its own worktree: the records side and, for a pair, its code branch.

### D5 — Nothing stays unpushed

- An owed push, origin unreachable, a push deferred or a teardown lock held, is retried inside the pusher with `statuscommit`'s owed backoff, from 1 s doubling to 60 s, without end.
  The pusher's push path writes no failed-push record of an owed class (transport, deferred, teardown_lock).
  `fabricengine.PairPush` gains a mode in which `PushPairAnchored` returns the owed `*PushFailedError` without writing its record:
  at `recordOwedRecordsPush` and `teardownLockOwedError` in `internal/fabricengine/pushanchored.go`,
  and at the owed-class failures `pushClassified` records in `internal/fabricengine/pushclassified.go`.
  In that mode `recordsSideNotLanded` reads a deferral from the error `pushClassified` returned, since no deferred record stands to read.
  So `FailedPushesOwedBy` (`internal/fabricengine/failedpush.go`) returns no record of the pusher's,
  and the owed-push read in `internal/pairteardown/runread.go` leaves `PushOwed` unset.
  The pusher holds neither the slug's teardown lock nor the push lock while it sleeps; it keeps its pusher lock.
  So teardown's exclusive take of the slug's teardown lock never waits on the backoff.
  Once its backoff reaches 60 s, the pusher posts one creel notice naming the worktree and the last error, and no other until a push lands.
- A refused push other than a takeover leaves the failed-push record and creel notice `PATTERN-pair-remote` requires, and the pusher exits.
  A refused-push record is a failed-push record of a class other than transport, deferred and teardown_lock;
  the restart gate counts only those classes, so an owed-class record another caller wrote never stops the pusher.
  While a refused-push record stands for one of the worktree's branches, neither a commit nor the dobby starts its pusher again; the record's way forward clears it.
  Records are keyed by repo side and branch, and the prime's records branch carries every batten run,
  so a refused prime push holds every batten run's records on this machine until the way forward clears it.
  That is accepted: the refusal is a state of the shared branch, so any batten run's next push of it would meet it again;
  the runs go on acting locally (D1), and only their pushes wait.
- The dobby, on each tick, starts a pusher for the prime and for every pair of its runs that is ahead of origin,
  when the start conditions of D3 hold.
  The prime is ahead when its records branch is.
  A pair is ahead when its local records branch is, or when its local code branch holds the code tip its local records tip's `Warp-SHA` names
  and that branch's origin tracking ref lacks it.
  Code commits no transition names yet are not counted; the transition that names them starts its own pusher.
  A branch is ahead when it holds a commit its origin tracking ref lacks, or when it has no upstream configured and holds at least one commit;
  a code branch with no upstream lacks every tip.
  The pusher's push sets the branch's upstream when it creates the branch on origin.
  So a records or code branch whose upstream is set but whose tracking ref a fetch pruned was ended on origin;
  it reads as level and is left to its teardown.
  The read is local.
- A resume starts the pusher of the run's worktree before the run acts.
- Worktree-Teardown shuts the pair's session down under the hub's prime lock and then releases the lock (`internal/battenshed/teardown.go`).
  It then pushes the pair through `pairteardown.PushRun` to completion with no prime lock held, so the owed backoff never holds that lock.
  It takes the prime lock again for the removal, which it makes only while origin's tracking refs hold the pair's local records and code tips;
  a tip that moved since its push sends it back to push with the lock released.
  An owed push holds the teardown in the owed backoff;
  a refused push leaves the failed-push record and notice and halts the teardown with the pair kept.
  Its archive push stays as it is.

Rationale: every path that can leave a commit behind has a restart, and the one step that removes local commits pushes them first.
Bound: the restarts are the next commit, the dobby's tick and a resume; a run with git or pushes off starts no pusher.
Worktree-Teardown is the one row that waits for a push, the exception to D1: it removes the commits, and nothing follows it.

### D6 — A takeover hands the original machine over from its pusher

A machine that adopts a run reads the run's record at origin's tip, as `lym adopt` does today, and re-runs the row that record names from the top.
The original machine's pusher meets the takeover entry on its next push: the records push is rejected, and fabric reads the takeover from the fetched tip.

- A pair's pusher passes its run's slug, and `pairteardown.PushRun` hands the run over through `pairteardown.HandOver`
  with `RunPush.InsidePair` true (`internal/pairteardown/handover.go`), leaving the pair for the parent's dobby to remove, as loom's hand-over does today.
  The pusher then exits.
- The prime's pusher serves every batten run and passes no one slug.
  On a rejected push it reads, before any replay, the takeover entry of every batten run named by a `Run-Transition` trailer of a local commit origin lacks.
  It hands each taken-over run over through `pairteardown.HandOver` with `RunPush.InsidePair` false, which removes that run's task pair as batten's hand-over does today.
  Before that run's hand-over mark is written, the hand-over pushes the prime's local records tip under an owner archive tag, which holds the run's unpushed prime commits;
  this changes `fabricengine.HandOverTips` (`internal/fabricengine/handover.go`), which archives only the pair's tips today.
  The replay then leaves out every commit whose `Run-Transition` trailer names a handed-over run and carries the other runs' commits in their order;
  a batten commit stages only its own run's records (`battenRunCommitPaths`), so the commits part cleanly.
  A takeover read, archive push or hand-over that fails counts as owed: the pusher replays nothing and retries the whole pass in D5's backoff,
  and the retried pass hands the run over again, as a failed hand-over does today.
  The sites that change are `retryWeftPush` in `internal/fabricengine/pushclassified.go`: its `runSlug != ""` gate, which skips the takeover read for a push with no run slug, and its replay.

A taken-over run's unpushed transitions stay under the owner archive tags and are never replayed.
A pusher runs from the hub, outside the pair, so a hand-over's removal of the pair leaves it standing.

Rationale: `PATTERN-run-fsm`'s hand-over path already turns a rejected push into a hand-over; only its caller moves.
Bound: the original machine acts until its next commit's pusher, the dobby's catch-up or an owner check meets the takeover.
The acts that harm outside the run when doubled keep their own origin checks:
PR-Review finds an existing pull request before it opens one,
and Finalize runs its owner check against origin, its parent-holds-work check, its already-landed path and its hand-over stop.
A pair's records push that lands before its code side leaves origin naming a `Warp-SHA` it lacks until the pusher's code push lands.
A read of the run on its own machine in that window acts:
the code-tip read in `internal/pairteardown/runread.go` leaves `CodeTipMissing` unset when the run's pair stands here and its local code branch holds the tip,
since the run's own pusher owes it.
The owner dobby's code-tip settle then sees no `code_tip` verdict and pushes nothing,
so D5's dobby tick, which counts a pair whose local code branch holds a transition's code tip origin lacks, carries the restart of that push.
A read on another machine still sees `CodeTipMissing`, which `runfsm.CodeTipGrace` and the dobby's code-tip settle cover as they do today.

### D7 — The loom pre-push reconcile runs in the pusher

The run-scope origin reconcile that loom's commit runs before its push today runs in a pair's pusher, before each push, with the same verdicts.
Its exceptions keep but one change: in the pusher a pause covering the run stops the push and the pusher exits,
where today the `pauseCovers` branch of `skipsCheck` in `internal/loomcli/prepush.go` skips the check and lets the push through.
The pusher reads the pause before the Finalize skip;
a run at Finalize with no pause covering it skips the reconcile and pushes, as today, since Finalize judges origin and its owners itself.
The `commit-records` verb runs no check before its commit, so its commit always lands; the check runs in the pusher before the push.
An entry owned elsewhere is handed over, as the catch-up does today.
An entry a branch stops, or that origin ended while the parent lacks its work, is paused and the orch noticed, as the pre-push check does today:
the pause is `tendercli.PausePair` (`tenderengine.Pause`), a slug-scoped entry in the hub pause record (`internal/loomcli/prepush.go`, `internal/tenderengine/pause.go`).
A pause covers the run, here and in D3's and D5's start conditions, when that record's coverage of the run's slug reads covered,
read through `pauserecord.ReadCoverage` as `pauseCovered` in `internal/loomcli/landingdeps.go` reads it.
The pusher pushes nothing more for that pair until `lym tender resume` ends the entry:
no commit, exit re-take or dobby tick starts it while the pause covers the run (D3, D5).
The driver meets that entry as it meets any tender pause: `runfsm`'s act rule reads the coverage as `PauseCovers` and holds the run.

Rationale: the reconcile is a fetch, so in the transition it would make the step wait; in the pusher it still guards every push of the run.
Bound: while origin is unreachable no reconcile runs, and the run goes on locally until origin answers.

### D8 — An owed push no longer holds the run

The run goes on while its push is owed; the pusher retries it.
The pusher's push path writes no owed-class failed-push record (D5), so `FailedPushesOwedBy` names none of its retries
and `runfsm`'s push-owed input stays unset while the pusher retries;
the act rule's dobby, pre-run and loom readers keep their behaviour.
A refused push's record keeps its refusal and way forward as today.

Rationale: holding the run at its transition is a wait for a push, which D1 rules out.
Bound: a machine cut off from origin goes on locally, and an adopt elsewhere re-runs from origin's last transition, as D2 accepts.

## Constraints

- `PATTERN-fabric-boundary` and `PATTERN-gitrepo-client-boundary`: the pusher runs git only through `internal/fabricengine`, under the remote-refs lock.
- `PATTERN-pair-remote`: records side first, code side only once it landed; a pair push holds the slug's teardown lock shared.
- `PATTERN-daemon-wakeups`: the pusher's backoff and re-reads wake at most once a second and back off while idle.
- `PATTERN-shed-driver`: the seams stay the shed's `CommitStatus` hook; no producer learns of the pusher.
- `PATTERN-tolerant-records`: the batten commit-status marker still reads an older binary's `push_pending`.
- A transition commit keeps going through `fabricengine.CommitRecordsPaths`,
  which takes the weft write lock that `retryWeftPush` try-acquires (`internal/fabricengine/commitweftpaths.go`).

## Auto-mode assumptions

- A refused push stops its worktree's pusher until the refused-push record's way forward clears it, rather than retrying on every commit.

## Open risks

None.

## Acceptance criteria

- D1: a transition of a batten run and of a loom run commits with its trailers and returns without a push, a fetch, an origin reconcile or a network-gate admission, for a running row, a skipped row, a start row and the seeded Worktree-Create commit.
- D1: `commitRunRecords` and the `commit-records` verb commit and start the worktree's pusher without a push.
- D2: a run adopted by another machine after unpushed transitions resumes at the last transition origin holds and re-runs that row from the top.
- D3: after a commit with the start conditions holding, one pusher starts;
  with the pusher lock held, a refused-push record standing for the records branch, or, for a pair, a pause or a hand-over mark, none starts.
- D3: a pusher whose worktree was removed exits.
- D3: a batten transition starts the prime's pusher and a loom transition the pair's.
- D3: the pusher pushes the records side before the code side, pushes again when the local tip moved during its push, and exits once origin holds the local tip.
- D3: a commit made while the pusher is exiting is pushed, by that pusher or by one the commit starts.
- D3: ending a run's processes leaves its worktree's pusher, registered under `runproc.HubOwner` with role `runproc.RolePush`, running.
- Constraint, as a regression check: a transition commit made while a pusher replays local commits onto origin's tip is kept, and origin receives every commit in its order.
- D5: an owed push is retried by the pusher until it lands, writes no failed-push record, leaves `PushOwed` unset in the run's read, and the run's next transition commits meanwhile;
  a deferred push is told from the returned error, and one creel notice goes out once the backoff reaches 60 s.
- D5: a refused push, of the records or the code side, leaves a failed-push record and creel notice, and no commit or dobby tick starts that worktree's pusher until the record is cleared;
  a refused prime push holds every batten run's pushes on this machine, and an owed-class record another caller wrote stops no pusher.
- D5: a dobby tick starts a pusher for a prime or pair ahead of origin with its start conditions holding,
  a branch with commits and no upstream included,
  and a pair whose records branch is level and whose local code branch holds its records tip's code tip that origin lacks included;
  it starts none for one that is level, for a pair whose code branch is ahead only by commits no transition names,
  or for one whose upstream is set with its tracking ref pruned.
- D5: a resume starts the pusher before the run acts.
- D5: Worktree-Teardown shuts the session down, pushes with the prime lock released, removes the pair only after its records and code reach origin, and a refused push halts it with the pair kept.
- D6: after another machine's takeover of a loom run, its pair's pusher hands the run over on its rejected push with `InsidePair` true and exits, the unpushed tips are under owner archive tags, and no commit or tick starts that pair's pusher again.
- D6: a batten run taken over while the prime holds its unpushed transitions is handed over on the prime pusher's rejected push;
  its unpushed prime commits go under an owner archive tag and are never replayed, and the other batten runs' commits reach origin;
  a failed takeover read, archive push or hand-over replays nothing and is retried.
- D6: between a pair pusher's records push and its code push, a read of the run on its own machine acts, and one on another machine answers the code-tip wait.
- D7: a pair's pusher runs the reconcile before its push; an entry owned elsewhere is handed over, and a stopped or ended entry pauses the run and the pusher pushes nothing more.
- D7: a commit made after a pause covers the run is not pushed until the resume.
- D8: with origin unreachable, the run walks its next rows and the dobby still schedules it.
- Review: `PATTERN-run-fsm`, `PATTERN-pair-remote` and `PATTERN-owner-machine` and their backgrounds state the Decisions and no longer state a transition push before the run acts or the start-row exception.
- Review: `internal/heddleengine/glossary.md` carries the pusher line D3 gives.

## PATTERN changes

- `PATTERN-run-fsm` — changes: Reading or acting on a run's lifecycle state: only `internal/runfsm`'s states and act rule; a transition commits and the run acts at once, a background pusher bringing every commit to origin in order; a run whose record holds another machine's takeover hands over, never replays.
- `PATTERN-pair-remote` — changes: Pushing run records or tearing down a pair: the code branch goes along after the records side, from the worktree's background pusher, which no transition waits for; a pair's remote steps hold the slug's teardown lock; teardown only via `internal/pairteardown`.
- `PATTERN-owner-machine` — changes: Starting, resuming, pushing or landing a run, a Finalize past its merge included, or picking a pair's parent: only on the entry's owner machine, read via hubgeom's `ParentOf` and Bolt's machine registry; a loom pusher reconciles first; after absence, a catch-up from origin first.

## Notes for the plan writer

- `fabricengine.SpawnStatusPush` and the `spawnStatusPush` variable in `internal/battencli/commitstatus.go` are the existing detached-push seam to build the pusher on or replace;
  `lym cleanup`'s push spawn in `internal/battencli/cleanup.go` uses the variable.
- The change description's "to check by hand" compares the time between skipped loom rows and between batten's later transitions against the entry's figures.
