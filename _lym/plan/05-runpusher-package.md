# Card 5 — runpusher-package

**Create:**
- `internal/runpusher/doc.go`
- `internal/runpusher/testmain_test.go`
- `internal/runpusher#Target`
- `internal/runpusher#TargetOf`
- `internal/runpusher#LockPath`
- `internal/runpusher#WorktreeFlag`
- `internal/runpusher#SlugFlag`
- `internal/runpusher#Start`
- `internal/runpusher#StartIfAhead`
- `internal/runpusher#starter`
- `internal/runpusher#starter.start`
- `internal/runpusher#spawnRequest`
- `internal/runpusher#Deps`
- `internal/runpusher#NewDeps`
- `internal/runpusher#Run`
- `internal/runpusher#PushUntilLanded`
- `internal/runpusher#RestartPusher`
- `internal/runpusher#TestStart`
- `internal/runpusher_test#TestRun`
- `internal/lymdirs#PusherDirName`

**Edit:**
- `cmd/lym#unboundedWaitFiles`

**Extends:**
- `internal/lymdirs/doc.go`

**Uses:**
- `internal/fabricengine#RunCommitsAhead`
- `internal/fabricengine#RefusedPushStands`
- `internal/fabricengine#ReadHandOverMark`
- `internal/fabricengine#HubScratchDir`
- `internal/fabricengine#RunProcDir`
- `internal/fabricengine#BoltDir`
- `internal/fabricengine#IsPushOwed`
- `internal/fabricengine#IsPrimeWorktree`
- `internal/hubgeom#PauseGeometry`
- `internal/pauserecord#ReadCoverage`
- `internal/selfexec#LymBinary`
- `internal/logger#EpisodeSet`
- `internal/statuscommit/testmain_test.go`
- `internal/runproc#TestEndSubject_EndsTheRunsThenTheSubjectAndItsDobbyLast`

**Intent:**
Add `internal/runpusher`, the pusher of D3: one detached process per records worktree that brings its run commits to origin in order.
Glossary: the background process that brings one worktree's run commits to origin in order.

`Target` names one worktree: its `*lymcwd.Location`, the run slug a pair's pusher passes, and whether it is a pair.
`TargetOf` builds it, reading the prime from `fabricengine.IsPrimeWorktree`.
A prime's target always carries an empty slug, whatever slug the caller passes:
the prime's pusher carries every batten run's commits and names no one run, and its push reads every batten run's takeover (card 3).
So every transition seam passes its run's slug: a batten run commits into the prime and gets the prime's target,
a loom run in its pair gets the pair's target with that slug.
`LockPath` is `<HubScratchDir>/<lymdirs.PusherDirName>/<records worktree name>.lock`, one lock per records worktree, taken through `internal/lock`.

**Start.**
`Start` starts a pusher when D3's start conditions hold, all read locally:
the pusher lock is free (a try-acquire released at once);
no refused-push record stands (`fabricengine.RefusedPushStands`);
and, for a pair, no pause covers the run's slug (`hubgeom.PauseGeometry` and `pauserecord.ReadCoverage`) and no hand-over mark names it (`ReadHandOverMark`).
`StartIfAhead` adds `fabricengine.RunCommitsAhead` first, for the dobby's tick (card 8).
A process with `FABRIC_SKIP_GIT` or `FABRIC_SKIP_PUSH` set starts none, so a run with git or pushes off starts no pusher;
`starter` reads those flags through a seam, which `Start` fills from the environment.
A failed read returns its error and starts none.
`starter` carries these reads and the spawn as seams; `Start` and `StartIfAhead` fill it from the hub.
The spawn starts `lym loom pusher --worktree <code worktree> [--slug <slug>]` (card 6) through `selfexec.LymBinary`, detached,
in the hub's Bolt directory, outside the pair, so a hand-over's removal of the pair leaves it standing.
It is logged at Info per `PATTERN-child-processes`, reaped through `proc.Reap`,
and registered through `runproc.Register` under `runproc.HubOwner` with role `runproc.RolePush`, never under the run's slug.
So a run's teardown, cancel or hand-over, which end the processes under the slug, never reach it,
and `runproc.EndSubject` already spares `RolePush` in its pass over a subject's own processes,
as `TestEndSubject_EndsTheRunsThenTheSubjectAndItsDobbyLast` already holds for a `RolePush` record under `runproc.HubOwner`.
An anchor change of the prime ends the hub's `RolePush` processes (`internal/fabriccli/anchor.go`), a live pusher included;
its commits stay local, and the next commit, dobby tick or resume starts a pusher again.
Under `go test` the spawn returns an error and starts nothing.
Start reports whether it spawned one; two racing starts may both spawn, and the loser exits at the lock.

`RestartPusher` is the restart the dobby's tick and both resumes share (card 8): it runs `start` once and returns nothing.
A failed start logs one Warn per episode keyed by `key`, through a `logger.EpisodeSet`,
and a later successful start logs the recovery; it never fails or drops anything of its caller's.

**Run.**
`Run` is the pusher's body; `Deps` holds its seams:
`Lock` (try the pusher lock), `Ahead`, `Conditions` (the start conditions but the lock), `PauseCovers` (nil for the prime),
`Guard` (nil for the prime), `Push`, `WorktreeGone`, `Owed`, `Notify` and `Sleep`.
`NewDeps` fills every local seam from the hub; the verb (card 6) fills `Push`, `Guard` and `Notify`.
1. It takes the lock and returns at once, pushing nothing, when another pusher holds it.
   A failed lock take returns its error.
2. Each pass returns, releasing the lock, when the worktree is gone or the conditions fail.
   A failed read of `Ahead` or the conditions returns its error, releasing the lock; the next commit, tick or resume starts a pusher again.
3. It pushes through `PushUntilLanded`, whose every attempt reads `PauseCovers` first, then runs `Guard`, then `Push`.
   A pause covering the run stops the attempt before the guard and the push, and the pusher returns (D7):
   so a pause written during an owed backoff stops the next attempt, and no commit, re-take or tick starts the pusher while it covers the run.
   A `Guard` stop returns, pushing nothing; the guard has paused the run or posted its notice.
   A failure `Owed` (`fabricengine.IsPushOwed`) does not read as owed returns: a refused push, whose record and notice fabric wrote,
   or a hand-over, after which the run's slug carries a hand-over mark and nothing starts the pusher again.
4. After a landed push it reads `Ahead` again and, while the local tip is ahead, pushes again through the same attempt,
   at most one push a second.
5. Once level it releases the lock and reads `Ahead` and the conditions once more.
   Ahead with the conditions holding, it tries the lock again and goes on, or returns when another pusher took it first.
   So a commit made while it exits is pushed by it or by the pusher that commit started.

`PushUntilLanded` retries an owed push without end, sleeping from one second, doubling to a minute.
It holds no lock while it sleeps but the pusher lock its caller holds: the push releases the teardown and push locks on return.
The first time the wait reaches a minute it calls `atCeiling` with the last error, and not again until a push lands.
`Run` passes a `Notify` of one creel line naming the worktree and that error.
A cancelled context returns its error. Worktree-Teardown reuses it (card 10).

`lymdirs` gains `PusherDirName`, and its `doc.go` names it.
`cmd/lym#unboundedWaitFiles` gains the file holding `PushUntilLanded`, since its wait lives as long as a push stays owed.
`internal/runpusher/doc.go` documents the package; `internal/runpusher/testmain_test.go` is `untagged`, a `TestMain` as `internal/statuscommit/testmain_test.go` has.

`TestStart` is tier0 over a hand-built `starter` with fake seams.
Rows: every condition holds, one spawn whose request carries the verb's arguments, the Bolt directory, `runproc.HubOwner` and `runproc.RolePush`;
the lock held, a refused-push record, a pause covering a pair, a hand-over mark of a pair: no spawn;
a prime with the pause and mark seams reading covered: spawned, since neither applies to the prime;
the skip-flags seam reading git or pushes off: no spawn and no read;
`ifAhead` with the worktree level: no spawn; ahead: one spawn;
a read failing: its error and no spawn;
a prime target built with a run slug: the spawn's arguments carry no `--slug`; a pair's carry it.

`TestRun` is tier0 over fake `Deps`, a recording `Sleep` and a context the test controls.
Rows: level after one push: one push and the lock released;
the tip moved during the push: a second push;
a commit while exiting with the lock free again: a re-take and one more push;
the same with the lock taken by another: no more push;
the lock held at start, the worktree gone, the conditions failing: no push;
the lock take, `Ahead` or the conditions failing: their error, no push, the lock released when it was taken;
a pair with a pause covering it: no push, the guard never called;
a pause appearing during an owed backoff: no further push, the guard never called again;
a pause appearing after a landed push while the tip is still ahead: no second push;
a guard stop: no push;
owed failures then a landing: sleeps of 1, 2, 4 seconds and so on up to 60, one `Notify` once the wait reaches 60, none after;
a second owed episode after a landing: one more `Notify`;
a refused push and a handed-over push: one push and return;
a prime target: the guard and the pause seams are nil and never called;
a cancelled context during the backoff: the context's error.

**ImpactSummary:** A new leaf of fabric, pause and runproc reads with no caller yet, its restart helper, plus one lymdirs constant and one wake-scan list entry.
