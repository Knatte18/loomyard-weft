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
The start-row exception of `926b5c28e`, and the marker's `push_pending` forcing a push, retire into this rule.

Rationale: the operator's decision, relayed by `ly:orch` (creel #2373, #2379).
Bound: a transition no longer proves origin holds the run before the run acts; D3, D4 and D6 carry that duty.

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

- It pushes through `pairteardown.PushRun` and so through `fabricengine.PushPairAnchored`: the records side first, the code branch only once the records side landed.
- At most one pusher runs per records worktree: it holds a lock named under `.lym` through `lymdirs` and taken through `lock` while it runs.
- After each push it reads the local records tip again and pushes again while the tip is ahead of origin.
- To exit, it releases its lock and then reads the tip once more; finding it ahead, it takes the lock again and goes on, or leaves it to a pusher that took the lock first.
- The seam starts a pusher after each commit when the lock is free; the check is local.
- The pusher's spawn is logged per `PATTERN-child-processes`.
  It registers in `internal/runproc` under its worktree, not under a run's slug, so ending a run's processes does not end it.

Rationale: one pusher per worktree pushes the latest tip, so a burst of skipped rows costs one push, not one per row;
keying it by worktree covers batten's records in the prime and a loom run's in its pair, which a slug key mixes up.
Bound: a pusher pushes only its own worktree and only the sides `PushPairAnchored` already pushes.

### D4 — A transition commit excludes a replay

A transition commit takes the records worktree's weft write lock, the lock `retryWeftPush` in `internal/fabricengine/pushclassified.go` try-acquires before it replays local commits onto origin's tip.
A replay moves the commits onto origin's tip in their order, with new SHAs.

Rationale: the pusher runs in another process than the driver, so without the lock a transition commit could land in the middle of a replay and be lost or reparented.
Bound: a replay that finds the lock held defers, as it does today, and the pusher's next pass retries it.

### D5 — Nothing stays unpushed

- An owed push, origin unreachable or a teardown lock held, is retried inside the pusher with `statuscommit`'s owed backoff, from 1 s doubling to 60 s, without end.
  The pusher holds no lock while it sleeps, so teardown's exclusive take of the slug's teardown lock never waits on the backoff.
  An owed retry writes no failed-push record.
- A refused push other than a takeover leaves the failed-push record and creel notice `PATTERN-pair-remote` requires, and the pusher exits.
  While that record stands for the worktree, neither a commit nor the dobby starts its pusher again; the record's way forward clears it.
- The dobby, on each tick, starts a pusher for the prime and for every pair of its runs whose local records branch holds a commit its origin tracking ref lacks, when the worktree's pusher lock is free and no refused-push record stands.
  The read is local.
- A resume starts the pusher of the run's worktree before the run acts.
- Worktree-Teardown pushes the pair through `pairteardown.PushRun` to completion before it removes the pair.
  An owed push holds the teardown in the owed backoff;
  a refused push leaves the failed-push record and notice and halts the teardown with the pair kept.
  Its archive push stays as it is.

Rationale: every path that can leave a commit behind has a restart, and the one step that removes local commits pushes them first.
Bound: the restarts are the next commit, the dobby's tick and a resume; a run with git or pushes off starts no pusher.
Worktree-Teardown is the one row that waits for a push: it removes the commits, and nothing follows it.

### D6 — A takeover hands the original machine over from its pusher

A machine that adopts a run reads the run's record at origin's tip, as `lym adopt` does today, and re-runs the row that record names from the top.
The original machine's pusher meets the takeover entry on its next push: the records push is rejected,
fabric reads the takeover from the fetched tip, and `pairteardown.PushRun` hands the run over through `pairteardown.HandOver`, with the effect it has today for the worktree's kind.
The unpushed transitions stay under the owner archive tags and are never replayed.
A pusher runs from the hub, outside the pair, so a hand-over's removal of the pair leaves it standing; it exits once the hand-over returns.

Rationale: `PATTERN-run-fsm`'s hand-over path already turns a rejected push into a hand-over; only its caller moves.
Bound: the original machine acts until its next commit's pusher, the dobby's catch-up or an owner check meets the takeover.
The acts that harm outside the run when doubled keep their own origin checks:
PR-Review finds an existing pull request before it opens one,
and Finalize runs its owner check against origin, its parent-holds-work check, its already-landed path and its hand-over stop.
A records push that lands before its code side leaves origin naming a `Warp-SHA` it lacks; `runfsm.CodeTipGrace` and the dobby's code-tip settle cover that as they do today.

### D7 — The loom pre-push reconcile runs in the pusher

The run-scope origin reconcile that loom's commit runs before its push today runs in a pair's pusher, before each push, with the same exceptions and verdicts.
An entry owned elsewhere is handed over, as the catch-up does today.
An entry a branch stops, or that origin ended while the parent lacks its work, is paused with the pause record `lym loom pause` writes and the orch noticed, as the pre-push check does today;
the pusher pushes nothing more for that pair until the run resumes.
The driver honours that pause where it honours `lym loom pause`.

Rationale: the reconcile is a fetch, so in the transition it would make the step wait; in the pusher it still guards every push of the run.
Bound: while origin is unreachable no reconcile runs, and the run goes on locally until origin answers.

### D8 — An owed push no longer holds the run

The run goes on while its push is owed; the pusher retries it.
Since an owed retry writes no failed-push record (D5), `runfsm`'s push-owed input reads no owed state of the pusher, and the act rule's dobby, pre-run and loom readers keep their behaviour.
A refused push's record keeps its refusal and way forward as today.

Rationale: holding the run at its transition is a wait for a push, which D1 rules out.
Bound: a machine cut off from origin goes on locally, and an adopt elsewhere re-runs from origin's last transition, as D2 accepts.

## Constraints

- `PATTERN-fabric-boundary` and `PATTERN-gitrepo-client-boundary`: the pusher runs git only through `internal/fabricengine`, under the remote-refs lock.
- `PATTERN-pair-remote`: records side first, code side only once it landed; a pair push holds the slug's teardown lock shared.
- `PATTERN-daemon-wakeups`: the pusher's backoff and re-reads wake at most once a second and back off while idle.
- `PATTERN-shed-driver`: the seams stay the shed's `CommitStatus` hook; no producer learns of the pusher.
- `PATTERN-tolerant-records`: the batten commit-status marker still reads an older binary's `push_pending`.

## Auto-mode assumptions

- A refused push stops its worktree's pusher until the refused-push record's way forward clears it, rather than retrying on every commit.
- Worktree-Teardown's push before removal is the one wait D1 allows, since teardown removes the commits it would otherwise lose.

## Open risks

None.

## Acceptance criteria

- D1: a transition of a batten run and of a loom run commits with its trailers and returns without a push, a fetch, an origin reconcile or a network-gate admission, for a running row, a skipped row, a start row and the seeded Worktree-Create commit.
- D2: a run adopted by another machine after unpushed transitions resumes at the last transition origin holds and re-runs that row from the top.
- D3: after a commit with the worktree's pusher lock free, one pusher starts; with it held, none starts.
- D3: a batten transition starts the prime's pusher and a loom transition the pair's.
- D3: the pusher pushes the records side before the code side, pushes again when the local tip moved during its push, and exits once origin holds the local tip.
- D3: a commit made while the pusher is exiting is pushed, by that pusher or by one the commit starts.
- D3: ending a run's processes leaves its worktree's pusher running.
- D4: a transition commit made while a pusher replays local commits onto origin's tip is kept, and origin receives every commit in its order.
- D5: an owed push is retried by the pusher until it lands, writes no failed-push record, and the run's next transition commits meanwhile.
- D5: a refused push leaves a failed-push record and creel notice, and no commit or dobby tick starts that worktree's pusher until the record is cleared.
- D5: a dobby tick starts a pusher for a prime or pair ahead of its tracking ref with its lock free, and none for one that is level.
- D5: a resume starts the pusher before the run acts.
- D5: Worktree-Teardown removes the pair only after its records and code reach origin, and a refused push halts it with the pair kept.
- D6: after another machine's takeover, the original machine's pusher hands the run over on its rejected push and the unpushed tips are under owner archive tags.
- D7: a pair's pusher runs the reconcile before its push; an entry owned elsewhere is handed over, and a stopped or ended entry pauses the run and the pusher pushes nothing more.
- D8: with origin unreachable, the run walks its next rows and the dobby still schedules it.
- Review: `PATTERN-run-fsm`, `PATTERN-pair-remote` and `PATTERN-owner-machine` and their backgrounds state D1 to D8 and no longer state a transition push before the run acts or the start-row exception.
- Review: `internal/heddleengine/glossary.md` carries the pusher line D3 gives.

## PATTERN changes

- `PATTERN-run-fsm` — changes: Reading or acting on a run's lifecycle state: only `internal/runfsm`'s states and act rule; a transition commits and the run acts at once, a background pusher bringing every commit to origin in order; a run whose record holds another machine's takeover hands over, never replays.
- `PATTERN-pair-remote` — changes: Pushing run records or tearing down a pair: the code branch goes along after the records side, from the worktree's background pusher, which no transition waits for; a pair's remote steps hold the slug's teardown lock; teardown only via `internal/pairteardown`.
- `PATTERN-owner-machine` — changes: Starting, resuming, pushing or landing a run, a Finalize past its merge included, or picking a pair's parent: only on the entry's owner machine, read via hubgeom's `ParentOf` and Bolt's machine registry; a loom pusher reconciles first; after absence, a catch-up from origin first.

## Notes for the plan writer

- `fabricengine.SpawnStatusPush` and the unused `spawnStatusPush` variable in `internal/battencli/commitstatus.go` are the existing detached-push seam to build the pusher on or replace.
- The change description's "to check by hand" compares the time between skipped loom rows and between batten's later transitions against the entry's figures.
