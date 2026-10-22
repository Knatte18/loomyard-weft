# Decision record — run-process-ownership

## Goal

Every process lyx starts for a run belongs to one owner record, keyed by the slug it serves or by the hub scope, so `lyx tender status` lists it and the verbs that end or tear down a run end all of it.
A pause leaves no network work parked as a live process, a teardown never waits on a run that has nothing left to do, and background work that is safe to freeze or that is not a session's work product never withholds a hibernate or holds a row.

## Scope

In:

- The owner record, `internal/runproc`, and registration of detached and long-lived run processes.
- Ending owned processes from pair teardown, `lyx reed down`, the owned-elsewhere pauses and a new `lyx tender reap` for orphans.
- `lyx fabric remove` against paused, ended and foreign runs.
- Detached pushes and board syncs under a pause, and the pushes owed after a lift.
- Owned processes and orphans in `lyx tender status`.
- Hibernate confirmation's reading of background work and the requesting orch.
- Shuttle's waiting turn end over shell-only background work.

Out:

- The verb that ends a live batten run for good: task `batten-cancel-verb`, which calls `runproc.End` and settles the batten record a killed batten leaves behind.
- The hub-wide spawn audit log: task `spawn-audit-log`, which extends the registration sites D2 introduces rather than adding a second launch path.
- Hub-level processes not started for a run: the orch, its watcher, the hibernate daemon, creel, reed's server and watchdog.
- Diverged local Bolt and prime weft after another machine landed: settled by `e610f9250` (origin reconcile, records replay).
- Starting, resuming or landing on a machine that does not own the entry: settled by `0d953f140`.
- A context bound on Webster forks: the operator's separate idea in note `pause-background-work`, not asked for here.
- Why a reviewer searched for PATTERN files: a separate issue, per issue #796.

## Decisions

### D1 — The owner record is one file per process under the hub scratch directory

Decision: `internal/runproc` keeps one JSON file per registered process in `<HubScratchDir>/runproc/<owner>/`, where `<owner>` is a run's slug or one reserved hub-scope name no slug can take.
A file holds the pid, the process-group id when the process leads its own group (zero otherwise), the start time, a role name, the argv and the registration time.
`Register` writes the file atomically through `internal/state`; a file for the same pid and start time is replaced.
No process removes its own file: every read prunes a file whose pid is dead or whose start time differs.
The package is told the directory and imports only the standard library, `internal/proc`, `internal/state` and `internal/logger`.
Rationale: a directory outside the pair's worktrees outlives the pair, as the teardown lock does, so an orphan stays visible after its pair is gone.
One file per process needs no lock between concurrent spawners, and pruning on read needs no cooperation from a child that is killed.

### D2 — What registers, and under which owner

Decision: a spawner that starts a detached child (`Start` with no `Wait`) registers it right after `Start`, in `SpawnDetachedPush` and `SpawnSync`, so every respawn path is covered.
A push of a task pair's sides registers under the pair's slug; a push of the prime's records side registers under the hub scope, whatever pause slug it carries; a board sync registers under the hub scope.
A long-lived run role verb registers itself under its run's slug, whichever way it was started: `lyx loom drive` and the detached step loop on start, `lyx batten run` once its run record is seeded.
The loop's step child stays recorded in the loop's pid file and tied to the loop, as today.
A synchronous child a CLI call waits for stays tied to its caller (`proc.StartTied`, `gitexec`), not registered.
A detached child of an interrupted CLI call is not tied to that call: it lives until it ends, or until an end verb reaches its owner, or `lyx tender reap` once its owner is gone.
Rationale: self-registration catches every launch path of a long-lived role, spawner registration catches short children that run no role code.
A prime records push serves the hub and must never be ended by a pair's teardown, as its missing teardown slug already says.
The board entry's "what should hold" asks that a child of an interrupted call die with it; this departs from that for detached children, because they are detached so they outlive their spawner (`fabric-remote-async`), and the owner record is what reaches them instead.
Registering the seeded batten, not the starting one, keeps a starting batten from reading as an orphan (D8).

### D3 — `runproc.End` ends an owner's processes, and only it does

Decision: `End` takes the record directory, an owner and the caller's pid, and ends every live recorded process and its descendants, except the caller and its ancestors.
On Linux, for every record at once, it first reads the descendants of each record, leader or not, by parent pid with their start times.
It then sends SIGTERM to each leader's group, to each non-leader, and to each read descendant outside a group it signals, with that descendant's own group when the descendant leads one.
It waits up to five seconds in all for them to be gone, sends SIGKILL to what is left, waits one further engine-constant bound, and returns an error naming the pid and role of each process still alive, beside the processes it ended.
A zombie, a process whose state reads exited but not yet reaped, counts as gone, since signal 0 still reaches it until its parent reaps it.
A group is signalled only while a live process still reports that group id with a start time at or after the recorded one, as `proc.KillTree` checks, and a pid only after its start time matches.
`End` then re-reads the owner's records and repeats on any live record it did not end, at most three passes, so a child registered during an end is reached.
On Windows, where `proc.StartTime` and `proc.IsAncestor` read nothing, `End` ends no recorded pid; the step loop still dies through its named job in the loop end, as today.
`End` returns each process it ended with its pid and role.
The five seconds, the bound after SIGKILL and the three passes are engine constants.
Rationale: `gitexec` starts a remote git subcommand under `proc.ConfigureGroupKill`, in a new process group with no parent-death signal and a kill only on its context's cancel, so a push's or step's `git push` and its `ssh` sit outside the group `End` signals and would outlive their parent until their deadline; reading every record's descendants before the first signal reaches them.
Because `End` returns without an error only once the pair's pushes are gone, and runs inside the session end before `RemovePair`'s remote branch step, no push of the pair outlives it to recreate a branch `remove --remote` then deletes; a push that finished before `End` is on origin, and the remote step deletes it as today.
git removes its lock files on SIGTERM, so a git process `End` signals leaves no `index.lock`.
The loop's step child, which `proc.StartTied` puts in its own group with a parent-death SIGKILL, is a read descendant of the loop, so `End` ends it with the loop, at the latest by that SIGKILL; that is why D5's run-lock wait runs before `End`.
A self-registered process that does not lead its group may share a group with the shell or agent that started it, so its group is never signalled.
Excluding the caller's ancestry lets batten's own Worktree-Teardown, run inside a registered `lyx batten run`, end the rest of its run.
Without a start time a pid cannot be told from a reused one, so Windows ends nothing it cannot identify.
Bound: `End` signals only processes recorded under the named owner whose identity still matches, and the descendants read from them before the first signal, each matched by start time.

### D4 — Pair teardown and `lyx reed down` end the pair's owned processes first

Decision: `internal/pairteardown`'s session end calls `runproc.End` for the pair's slug before its loop end, so `lyx fabric remove`, batten's Worktree-Teardown and the origin reconcile's removals end batten, the driver, the step loop and the pair's pushes before the session and the worktrees go.
Its order, when D5's readings skip the driver checks, is: D5's run-lock wait, the refusal probe, `End`, the driver-lock re-check, the loop end, the loop end's own run-lock wait, the session end.
Otherwise it is: the quiet wait as today, the refusal probe, `End`, the driver-lock re-check, the loop end, its run-lock wait, the session end.
A pair whose task worktree is gone skips the quiet wait, both run-lock waits and the loop end as today, and the driver-lock re-check, since none of them can read a gone worktree; it still runs the refusal probe, and calls `End` for the slug before the session end by exact name.
A removal the refusal probe refuses ends no owned process.
An `End` error naming a process still alive after SIGKILL makes a `RefuseWhenBusy` caller and `lyx reed down` refuse with nothing removed and the session live, naming each survivor's pid with the way forward of retrying once it is gone; batten's Worktree-Teardown and the reconcile's forced teardown go on and report the survivors in `SessionResult`, and the D7 sites log them at Warn with the pause standing.
After `End`, a driver lock still held means a process `End` could not reach, such as one an older binary started.
A caller passing `RefuseWhenBusy`, `fabric remove` and the reconcile's local removal, then refuses with `ErrDriverBusy` naming the lock, with no session ended and nothing removed.
Batten's Worktree-Teardown and the reconcile's forced teardown pass no `RefuseWhenBusy`: the loop end and session end run, and the re-check sets `SessionResult.DriverWasLive`, which today only a spent quiet wait sets.
On Windows `End` ends no recorded pid, so a `fabric remove` of a paused run whose Go driver is live refuses there with `ErrDriverBusy`; the refusal names the driver's pid from the owner record, and the way forward is `taskkill /PID <pid> /T /F`, then the removal again.
`lyx reed down` in a pair runs no origin reconcile and makes no network call, so it never waits at the gate.
It reads only local state: a live pause entry covering the slug, the local board entry done, a Finalize Done in the status history with batten not running, or a run not running, which is no driver lock held, the loop lock free, no live driver strand as the quiet rule reads it, and batten not running.
A run owned elsewhere reaches it as the pause entry its owner pause wrote (D7), so it takes no owner verdict of its own.
On any of these it runs D5's run-lock wait, calls `runproc.End` for the pair's slug and ends the session; otherwise it refuses with the way forward `lyx tender pause <slug>`, then `lyx reed down` again, with nothing ended.
Today `reed down` in a pair kills the session of a running run, whose strands batten then revives; this refusal withdraws that.
It has no `stopped` reading, since that is a reconcile verdict.
Each envelope carries the ended processes under `ended_processes`.
No lyx run process handles SIGTERM, so an ended driver, loop or batten exits at once and writes no drive report and posts no stop notice.
A killed batten leaves its run record reading `running` with no process; ending that record is `batten-cancel-verb`'s.
Until then every reader that a `running` batten record holds back reads batten as running only while a process registered under the slug with the batten role is alive: D5's skip readings, `reed down`'s readings, D8's orphan shield, and the board claim settle, which for a `running` record with no live registered batten keeps the claim and warns that batten's process is gone, naming `lyx board set-status <slug>` instead of `lyx batten pause`.
Rationale: ending batten and the driver before the strands keeps batten's dead-strand revival and the driver's re-step from racing the session end.
The driver-lock re-check keeps an unregistered process from driving on against a removed pair.
A local reading can lag origin; `reed down` refuses only a run that reads running and unpaused locally, where a pause is the way forward.
Bound: a running unpaused run is never ended by `reed down`, nor by `fabric remove` bar a run the reconcile reads ended or owned elsewhere (D5); the driver-lock refusal binds only `RefuseWhenBusy` callers.

### D5 — `lyx fabric remove` and a run with nothing left to do

Decision: `lyx fabric remove` runs the run-scope origin reconcile for the slug first (`PATTERN-origin-reconcile`).
A `stopped` verdict refuses with the reconcile's way forward, as for every other run-scope caller, with nothing ended.
An `ended` or `owned_elsewhere` verdict removes the pair through `pairteardown`'s local mode: no archive-tag push, no remote branch step, no board write, and a `--remote` it drops is named in `remote_skipped_reason`.
A reconcile that already removed the pair itself is read as done, with its ended processes in the envelope.
The quiet wait's driver checks are skipped, and the owned processes ended, when the verdict is `ended` or `owned_elsewhere`, when a live pause entry covers the slug, or when the local board entry is done or the status history holds a Finalize Done and batten is not running, as D4 reads it.
The PATTERN text's "done locally" names the last two readings.
With the checks skipped, the run-lock wait runs before `End` and before the loop end, bounded by `runLockWait` and naming its holder, so a step doing local work finishes before `End` ends the loop and the step with it.
A run lock still held at its bound is passed, and its holder ended by `End` and the loop end, in two cases.
One is a holder waiting at the network gate: a live waiting marker for the slug names the run's newest unfinished step's pid or a process registered under the slug.
The other is a live pause entry covering the slug while the run holds no parent lock record: the holder is then a step waiting on its interrupted agent, which the loop's watch never kills under a pause, and it does no local work a parent lock guards.
Any other holder, such as a step still merging under the parent lock past the bound, refuses with `ErrDriverBusy`, naming its pid, with nothing ended, and the way forward of retrying the removal once that pid ends.
To give that reading, `pauserecord.Network` writes a waiting marker, its pid, start time and slug, in its own directory beside the in-flight markers before it waits in `WaitUncovered`, and removes it when the wait returns, before it writes its in-flight marker.
A waiting marker whose pid is dead or whose start time differs reads as absent, and `ListNetOps`, so hibernate's in-flight reading, never lists one.
Otherwise the bounded quiet wait and its `ErrDriverBusy` refusal stand, and the refusal names the owned processes and the way forward `lyx tender pause <slug>`, then the removal again.
`lyx fabric remove` passes the network gate for its own calls under any pause, through `SetGateExempt`'s mechanism extended to every entry for that one process.
With origin unreachable, the local Bolt's owner and status decide, as for every other reconcile caller.
Rationale: a paused driver starts nothing until a resume, and an ended or foreign run's driver has nothing left to do here, so waiting on either only blocks the operator; a step still doing local work under a pause, such as a merge under the parent lock, is waited for through the run lock, while a step parked at the gate, the incident's case, holds it with nothing left to do here.
Batten still runs Land and deploy after Finalize Done, so that reading counts only once batten is no longer running.
Local mode is the reconcile's existing removal for ended entries, so a machine that does not own the run pushes no archive tag of stale records (#741).
An explicit removal is the operator acting through the pause, so its own calls pass; the reconcile it adds is a read of origin, the same kind as `fabric add`'s branch probe that `fabric-remote-async` keeps synchronous, and `PATTERN-origin-reconcile` requires it.
Bound: the driver-check skip lets through only the readings above; the gate exemption covers only the remove process's own calls, never its children; `--force` still answers dirtiness only.

### D6 — Under a pause, detached pushes and board syncs exit, and every lift spawns what they owe

Decision: a pause leaves the Go driver, the step loop, batten and a session's background shells running and parked; only detached pushes and board syncs stop.
A detached push or board-sync child, marked by a hidden flag its spawner sets (the push's existing pause-slug flag, a new hidden flag on the sync), checks its coverage before it marks a pending flag or takes its lock, and exits at once when covered.
The gate reads a parent lock record at each network call, through `HoldsParentLock` (`HeldBy` over the live records, `internal/tendercli/gate.go`), not at spawn, and the record is released or lapses once Finalize is Done, so a push that reached its call after the release would put back its flags and exit under an owner pause, and no lift spawn would bring it back for a pair another machine owns.
A push child spawned while a parent lock record names its slug therefore never exits for coverage: its spawner reads the record at spawn and sets a hidden flag, and for the child's whole life its coverage check reads the flag in place of the live records and its gate passes a slug entry carrying `pauserecord.OwnerCaller` of its own slug.
It keeps the gate's waiting behaviour otherwise: `HoldsParentLock` stays the live reading, so any other slug entry passes only while a record names the slug and the in-flight marker's `UnderParentLock` reports the live record, and a hub entry is waited out.
A Finalize finishing at an owner pause, whose pair the lift spawn excludes, still has its pushes made, also those that reach their call after the release, and an operator's slug pause set after the release still holds them.
A child that meets a covering pause at a later network call puts back the pending flags of the sides it has not pushed and exits.
An exit under a pause pushes nothing more, leaves the commits local, writes one log line and no failed-push record or creel mail, since the push is deferred, not failed.
Every other process keeps the gate's waiting behaviour.
`tenderengine`'s lift sequence, which `lyx tender resume`, the hibernate wake and cancel, `lyx adopt` and the cleanup reconcile all run, calls a spawn seam after it lifts its entries: one status push for each pair a lifted entry covered that still exists, no live entry still covers and no other machine owns, one prime records push, and one board sync when Bolt is ahead of its upstream.
Rationale: the parked driver chain is idle and is what carries a resumed agent's completion, and ending it on every pause would need every resume path to restart drivers mid-step; owned, listed and ended by every end verb, it no longer blocks or outlives anything.
This departs from the board entry's line that a pause "stops work instead of parking processes", which its Expected list allows as long as `fabric remove` ends them (D4, D5).
A parked push is a live process holding a stale write; left as commits and flags, it is a record the next push carries through the records replay, which sets aside an ended entry's commits.
A removed pair's owed push goes with the pair.
Bound: the no-wait behaviour covers only the two marked child kinds; the spawn-time lock flag covers only the pushes spawned while a parent lock record names their slug, which are the pushes a Finalize spawns past its parent-side merge, and lets them pass only their own slug's owner entry; the gate's contract that it only waits and fails no call holds for every other process.

### D7 — An owned-elsewhere pause ends the run's processes on the machine it left

Decision: every site where an owned-elsewhere verdict pauses a run also calls `runproc.End` for the slug.
Those are the origin reconcile at the hibernate wake and at `lyx orch start`, and batten's owner probe (`innerRunProducer.ownedElsewhere`, after `PauseOwned`), the path a live run takes when another machine adopts it.
A run whose Finalize is past its parent-side merge, read as `keepsLockForFinalize` reads it (current producer Finalize, running or failed transient with no re-step, and the parent holding the task's work, or a failed read of either seam), is not ended at any of these sites: it keeps its parent lock and finishes its landing, as `PATTERN-owner-machine` allows, and its processes are ended by the pair's removal.
The probe runs inside `lyx batten run`, so `End` spares batten and its ancestors and ends the driver chain and the pair's pushes.
Batten does not exit: it keeps its owner hold and probe as today, and the pair's removal on this machine, `fabric remove` or `reed down`, which read the run owned elsewhere or paused (D4, D5), ends it.
Batten adds no restart.
At the owner-probe site, a running child whose Go driver `End` ended holds neither driver lock nor strand, so the surviving batten sends its existing driver-dead notice once the run is owned here again and its owner pause is lifted, since batten sends no notice for a running child while a pause covers it, and the operator restarts the chain with `lyx loom start` run in the task worktree.
`lyx loom resume` refuses that child, since its Go spawn is for a halted run only: a child whose status reads running and that holds no driver gets its no-live-driver refusal, which names `lyx loom start`.
At the hibernate wake and `lyx orch start` sites, `End` ends batten with the driver chain, so no notice is sent and nothing restarts the chain on its own until `lyx adopt` runs its step 5, `battencli.ResumeInWindow`, which starts a batten that spawns again a running child whose spawn it has not confirmed, the driver lock refusing a second driver; once the run reads owned here, the operator may also restart it by hand with `lyx batten run <slug>` from the prime or `lyx loom start` in the task worktree, and a run left owned elsewhere gets no restart on this machine.
Rationale: `0d953f140` left ending a run's processes on the machine it left to this task; a driver chain parked under the owner pause on that machine would otherwise outlive the adoption, the incident's pattern.
Bound: only for a slug an owned-elsewhere verdict pauses at that site, never a Finalize past its parent-side merge.

### D8 — `lyx tender status` lists owned processes and orphans, and `lyx tender reap` ends orphans

Decision: `lyx tender status` gains a processes part: each live owned process per owner, with role, pid and age.
A record is an orphan when its owner is a slug, not the hub scope, that has no pair (no registered linked task worktree) and whose batten is not running, read as D4 says: a `running` batten record counts only while a process registered under the slug with the batten role is alive.
The part also lists, as orphans, unregistered processes running a lyx executable whose working directory is a deleted directory inside the hub and outside every worktree git still registers, prime and task worktrees alike, leaving out those the replaced-binary part lists.
`lyx tender reap` ends every orphan of one fresh reading: registered ones through `runproc.End`, unregistered ones after a start-time match from that reading with D3's sequence, SIGTERM, up to five seconds, then SIGKILL.
A process still alive after D3's bound past SIGKILL is reported with its pid, and its role when registered, and `reap` exits non-zero.
It takes `lyx tender pause`'s caller check with no slug: the orch and an external caller may run it, an agent is refused with the way forward of asking its parent.
On a platform that cannot list processes, status still lists the registered records with their pids, and reports the unregistered orphans unsupported, as the replaced-binary part does.
Rationale: once a pair is removed, no `remove` or `reed down` target is left, so one verb ends what survived, and status names it as the way forward.
Bound: `reap` never ends a hub-scope process, a process whose pair exists, a process whose working directory lies inside a registered worktree, or one of a slug whose batten reads running; those are ended by the pair's end verbs or by `batten-cancel-verb`.

### D9 — Reed's session end stays as it is

Decision: reed's end of a session is unchanged.
`Engine.Down`, and `EndSessionByName` through `reapSessionVia`, already read each live pane's descendant closure before `kill-session`, then wait up to `reapExitTimeout` and SIGKILL each survivor and each late member of a session one of them led (`reapPaneChildren`, `internal/reedengine/lifecycle.go` and `overlay.go`), for every session, the prime's included.
An agent's background shells are descendants of its Claude process, so they are reaped there, and so is an unregistered lyx process in the closure, such as an agent's foreground `lyx gate test`.
Rationale: the existing reap covers the agent-shell case; issue #796's shells held a row of a live session, not a session end, and D11 settles that.
A run's owned processes are ended by `End` before the session end (D4), so the reap reaches none the record must end itself.
Bound: no change to what reed's session end kills once it runs; `lyx reed down` in a pair no longer runs it for a run that reads running and unpaused (D4), so that session and its pane processes stay live until one of D4's readings holds, such as a pause.

### D10 — Hibernate confirmation reads safe background work as settled

Decision: a confirmation reading counts a session that reads busy only on background work as settled when every outstanding task is a shell.
A session with an outstanding fork still withholds.
Confirmation fails at ten minutes unless every withholding reason of that reading is fork work, and at sixty minutes in all whatever the reasons, both engine constants.
When the request's requester is the orch, the orch's own session is left out of the reading.
Rationale: a suspend freezes a shell and the wait loop it holds and resumes them on wake; a fork is never interrupted (operator view, 2026-10-15) and its next batch is already held by the pause (`fac869bb3`), so confirmation lets it finish.
Sixty minutes covers the longest fork seen, the 45-minute Webster fork of 2026-10-14 the board entry's `pause-background-work` note records, with fifteen minutes of margin.
The orch is not interrupted by a hub pause, and its writes that matter to a hibernate, a landing and network calls, are already withheld through the land lock and the in-flight network operations.
Bound: the land lock and in-flight network operations withhold as before; the longer bound applies only to a reading where forks are the only reason.

### D11 — Shell-only background work never holds a row its outputs settle

Decision: a gated run's waiting turn end whose outstanding work holds no fork and no shell `Spec.AwaitedShellPrefixes` names is an arrival: the gate is evaluated as at a plain turn end, and a passing gate finishes the run Done, with the shells in `Result.EndedShells`.
An ungated run whose output files are missing and whose outstanding work is only such shells, outstanding past `Config.BackgroundShellWaitMin`, finalizes as at an idle-stalled turn end, so the row's own stall handling surfaces it to the parent.
`BackgroundShellWaitMin` thereby becomes the expiry of that wait and stays the display threshold of every other wait.
Rationale: a gated waiting turn end is never an arrival because a fork may still write the outputs, as the lens forks do after the writer's files exist; a non-awaited shell is not the session's work product, so an output written before it is final.
The LLM loom driver, the one session that waits hours on a background shell, is never waited on by shuttle's `Wait` (`internal/loomcli/driverspec.go`), and Webster's Master names its awaited shells, so neither is cut off.
Bound: only shells no awaited prefix names; forks and awaited shells keep the current rule.

## Constraints

- `PATTERN-pair-teardown`: every teardown still goes through `internal/pairteardown`; D4 and D5 change its quiet-wait and session-end clauses, listed under PATTERN changes.
- `PATTERN-origin-reconcile`: D5 runs the run-scope reconcile and takes its verdict as the only judge of whether an entry ended.
  It meets `fabric-remote-async`'s rule that no command waits on the network; the PATTERN entry wins, and the reconcile is the same kind of origin read that entry keeps synchronous.
  `lyx reed down` runs no reconcile: it judges no branch, takes no owner verdict, and starts, resumes or lands nothing, so it only ends processes on local readings (D4).
- `PATTERN-owner-machine`: D5's local mode, D6's owner exclusion and D7 take the reconcile's or `boardengine`'s owner verdict; no owner is set here.
- `PATTERN-spawn-observability`: `Register` and `End` log at Info; a detached spawn still logs its spawn at its site.
- `PATTERN-told-geometry` and `PATTERN-lyxdirs-single-declarer`: the record directory is declared beside `HubScratchDir` and handed to `runproc`.
- `PATTERN-tolerant-records`: a record file is read tolerantly and written through `internal/state`.
- `PATTERN-shuttle-provider-seam`: D11 decides over provider-neutral `BackgroundTask` kinds.
- `PATTERN-shuttle-stop` and `PATTERN-guarded-interrupt`: no strand is stopped and no key is sent outside shuttle and reed.
- `PATTERN-pause-holds-prompts`: D6 changes only the two marked child kinds, and D5's waiting marker changes no gate decision, only what the gate records while it waits.
- `PATTERN-push-failure-visible`: a push deferred under a pause is not a failure; a push that fails after a lift follows the rule.
- Tests inject clocks for D3's grace and D10's bounds.
- Fact: Linux keeps a process-group id unassigned while any member lives, so a group match with a start time at or after the recorded one is the recorded group (`internal/proc/tied_linux.go`, `killTreePlatform`).
- Fact: `proc.Detach` on Linux starts the child with `Setsid`, so a detached child leads its own group (`internal/proc/proc_linux.go`).
- Fact: `proc.StartTime` reads nothing and `proc.IsAncestor` is false on Windows (`internal/proc/proc_windows.go`).
- Fact: git's lockfile code removes held lock files on a fatal signal it catches, SIGTERM included (git `lockfile.c`/`tempfile.c` sigchain cleanup; git 2.43 on the incident host).
- Fact: no lyx run process installs a SIGTERM handler; only the orch watcher, creel and the gate child do.
- Fact: the pending-flag holder takes its owed flags before its network step (`internal/fabricengine/pendingflags.go`), so D6's check sits before the flag is marked.
- Fact: the Claude Stop payload's `background_tasks` entries carry a type, `subagent` for a fork and any other for a shell (`internal/shuttleengine/claudeengine/background.go`).
- Fact: `lyx tender resume` (`Resume`) and `Lift` share `liftEntries`, which the hibernate wake and cancel, `lyx adopt` and the cleanup reconcile reach through `Lift`, so D6's spawn lives in `liftEntries` (`internal/tenderengine/resume.go`).
- Fact: `pauserecord.Network` writes its in-flight marker only after `WaitUncovered` returns, `ListNetOps` lists only calls past the gate, and `awaitRunLockNamingHolder` names only the newest unfinished step's pid (`internal/pauserecord/hold.go` `Network`, `internal/pairteardown/loop.go`), so a process waiting at the gate leaves no trace without D5's waiting marker.
- Fact: today's run-lock wait runs after the loop end, and the loop's step child, started with `proc.StartTied`, leads its own group (`Setpgid`) and gets a parent-death SIGKILL (`internal/pairteardown/doc.go` EndSession step 4, `internal/proc/tied_linux.go`), so a wait after `End` would find a mid-merge step already dead.
- Fact: `gitexec` starts remote git subcommands under `proc.ConfigureGroupKill`, a new process group killed only on context cancel, with no parent-death signal (`internal/gitexec/gitexec.go`, `internal/proc/procgroup_linux.go`).
- Fact: reed's session end already reaps each live pane's descendant closure, read before `kill-session`, with SIGKILL after `reapExitTimeout` (`internal/reedengine/lifecycle.go` `reapPaneChildren`, `overlay.go` `reapSessionVia`).
- Fact: while a pause covers the run, the step loop's watch judges nothing and never kills, so a step waiting on its interrupted agent holds the run lock until the pause lifts (`internal/shedverbs/doc.go`).
- Fact: batten's owner probe pauses a run owned elsewhere under the owner caller (`internal/battenshed/innerrun_owner.go` `ownedElsewhere`).
- Fact: `ownedElsewhere` keeps the parent lock for a Finalize past its parent-side merge, and a parent-lock holder passes a slug pause entry at the gate (`internal/battenshed/innerrun_owner.go` `keepsLockForFinalize`, `internal/pauserecord/hold.go` `SlugEntryExempt`).
- Fact: a batten resumed through `battencli.ResumeInWindow`, which `lyx adopt`'s step 5 runs for a task whose batten run record is not done, spawns again a running child whose spawn it has not confirmed and that holds no driver strand, run lock or Go driver lock (`internal/battenshed/doc.go`, `internal/adoptcli/doc.go`).
- Fact: `lyx loom resume` spawns a Go driver only for a child whose status is not running, and refuses a running child with no live driver, naming `lyx loom start`, whose spawn follows the locks and strand whatever the status (`internal/loomcli/resume.go` `noLiveDriverMessage`, `internal/loomcli/bootstrap.go` `mustSpawnDriver`).
- Fact: batten keeps a parent lock record handed to it while the child runs and releases it when the child is in any other state (`internal/battenshed/innerrun_parentlock.go`).
- Fact: a hibernate request records its requester (`internal/hibernateengine/request.go`).

## Auto-mode assumptions

- The board entry's "one verb ends them all" is met for a live run by `batten-cancel-verb`'s verb over `runproc.End`, and for a removed pair's survivors by `lyx tender reap`.
- `lyx tender reap` is the only new verb; the board sync's no-wait marker is a hidden flag, not a verb.

## Open risks

- A process an older binary started is never registered; D4's driver-lock re-check refuses a `RefuseWhenBusy` teardown it would outrun, and D8's deleted-directory rule or `ps` finds the rest.
  An older binary's `lyx batten run` is not registered either, so D4's batten reading counts it not running.
- A self-registered `lyx batten run` that does not lead its group is ended with the descendants read at the end; a descendant that already reparented is missed.
- With the driver checks skipped, a step a live loop starts between D5's run-lock wait and `End` dies with the loop at `End`.
- D11 finishes a gated run while an unawaited shell still runs; the shell lives until the run's strand is removed at its Done finalize, or the pair's session end when the pane is kept, and survives only when it left the pane's process tree.
- An ungated run waiting on a long shell no awaited prefix names now stalls after `BackgroundShellWaitMin`.
- A push holder ended by `End` on a live pair loses the pending flag it took; its commits wait for the next push of that worktree.
- On Windows, `End` ends only the step loop; the other owned processes are listed but never ended there, and a `fabric remove` of a paused run with a live Go driver refuses until the operator ends it (D4).

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 4: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/discussion/round-4-review.md; fixer report: _lyx/reviews/discussion/round-4-fixer-report.md.
  - d6-push-child-coverage-reading (class design, severity MEDIUM, open in rounds 3, 4)
  - d7-revival-does-not-restart-driver (class design, severity MEDIUM, open in rounds 3, 4)
<!-- /lyx:carry-over Discussion-Review -->

## Acceptance criteria

- A detached pair push, a prime records push, a board sync, `lyx loom drive`, the detached step loop and a seeded `lyx batten run` each leave a record under the owner D2 names, and a dead process's record is pruned on the next read.
- `runproc.End` ends a recorded group leader with its group and a non-leader with its descendants, ends a descendant leading its own group, such as a recorded push's remote git child, with that group, skips the caller and its ancestors, leaves a reused pid alone, and reaches a child registered during the end.
- `runproc.End` over a process that survives SIGKILL returns, once the bound after SIGKILL is spent, an error naming its pid and role, and counts a zombie as gone; `lyx fabric remove` and `lyx reed down` then refuse with nothing removed, naming the pid and the retry, batten's teardown goes on and reports it, and `lyx tender reap` reports it and exits non-zero.
- `lyx fabric remove` on a paused run with a parked driver, step loop and batten, under a slug pause and under a hub pause, ends them and removes the pair, naming them in `ended_processes`.
- `lyx fabric remove` under a pause while a step holds the run lock doing local work waits for it before ending anything; at the bound, a step whose live waiting marker shows it at the network gate is ended, and so is a step waiting on its interrupted agent under a covering pause with no parent lock record; a holder of the parent lock past the bound refuses, naming its pid, with nothing ended.
- `pauserecord.Network` leaves a waiting marker while it waits at the gate and none once it is past, and `ListNetOps` never lists a waiting marker.
- `lyx fabric remove` on a paused run whose removal is refused, such as a dirty pair without `--force`, ends no owned process and leaves the session live.
- `lyx fabric remove` on an entry the reconcile reads `ended` or `owned_elsewhere` removes the pair in local mode, and on `stopped` refuses.
- A teardown of a pair whose task worktree is gone runs the refusal probe, ends the processes registered under its slug and then the session by exact name, with no quiet wait, no run-lock wait and no loop end.
- `lyx fabric remove` after Finalize Done while a live registered batten runs does not skip the quiet wait; with the batten record reading `running` and no live registered batten, it skips it.
- A `RefuseWhenBusy` teardown whose driver lock stays held after `End` refuses with nothing removed; batten's teardown and the reconcile's forced teardown go on and report `DriverWasLive`.
- `lyx reed down` makes no network call; in a pair it ends the owned processes before the session, so no strand is revived, under a covering pause, an owner pause included, a local done entry, a Finalize Done with batten not running, or a run not running; on a running unpaused run, a run with a live loom driver strand included, it refuses with nothing ended.
- The board claim settle for a `running` batten record with no live registered batten keeps the claim and names `lyx board set-status`, not `lyx batten pause`.
- Batten's Worktree-Teardown ends the rest of its run without ending its own process, and never a prime records push.
- A detached push or board sync under a covering pause exits without pushing, without a failed-push record, and with its pending flags in place; a push spawned by a Finalize past its parent-side merge under an owner pause is not deferred, also when it reaches its network call after batten released the parent lock record at Finalize Done, and under a hub entry waits at the gate until the lift; each lift path spawns the owed pushes for the pairs it uncovered, none for a removed pair or one owned elsewhere.
- A reconcile owned-elsewhere pause at wake or `lyx orch start` ends that run's owned processes, batten included, and a run so ended and then adopted here has its Go driver chain spawned again by the batten `lyx adopt` resumes, with no second driver; batten's owner-probe pause ends them except batten itself, which keeps running and, once the run reads owned here again and its owner pause is lifted, sends its driver-dead notice for a Go driver `End` ended, after which `lyx loom start` in the task worktree restarts the chain and `lyx loom resume` refuses, naming `lyx loom start`; a Finalize past its parent-side merge is not ended at the probe, the wake or `lyx orch start`, and finishes its landing.
- `lyx tender status` marks an orphan only for a slug with no pair and no live registered batten, never the hub scope, and lists an unregistered lyx process in a deleted hub directory outside every registered worktree; a lyx process in a deleted directory inside a live pair is not listed and not reaped; `lyx tender reap` ends exactly the listed ones, unregistered ones by SIGTERM then SIGKILL, and refuses an agent.
- Hibernate confirmation confirms with sessions busy only on shells, withholds on a fork up to sixty minutes, and leaves out the orch's session when the orch requested.
- A gated shuttle run whose writer wrote a passing verdict and then started a non-awaited shell finishes Done at that turn end; with a fork outstanding it still waits.
- An ungated run with missing outputs and only non-awaited shells outstanding past `BackgroundShellWaitMin` finalizes stalled.
- `internal/runproc` gets its `doc.go`, every package whose behaviour a Decision changes updates its doc, and `docs/overview.md` its module table.

## PATTERN changes

- `PATTERN-run-process-owner` — adds: Starting a detached or long-lived process for a run: register it in `internal/runproc`'s owner record; ending a run's processes: only via `runproc.End`.
- `PATTERN-pair-teardown` — changes: Every teardown of a pair goes through `internal/pairteardown`, which ends the pair's owned processes and then its reed session before any worktree is removed; its driver checks are skipped for a pair that is paused, ended, owned by another machine, or done locally with batten no longer running, never its run-lock wait while the task worktree is present, which then runs before the owned processes are ended.
- `PATTERN-leaf-packages` — changes: adds `internal/runproc`, importing only the standard library, `internal/proc`, `internal/state` and `internal/logger`.

## Notes for the plan writer

- The loop end in `pairteardown/loop.go` already kills the loop and its step tree; `runproc.End` runs before it and changes neither the loop pid file nor the teardown mark, and with the driver checks skipped a run-lock wait over `awaitRunLockNamingHolder` also runs ahead of `End` (D4, D5).
- `OriginSeams`' `LivePair` already reads a slug under a live hub pause as not live; D5's paused reading can share that rule, and the reconcile's `Teardown` seam takes the same skip, since it lives in `pairteardown`.
- `pauserecord.SetGateExempt` is the mechanism D5's remove exemption extends.
- D7's Finalize exception at the wake's `holdRuns` and `lyx orch start`'s `pauseOwnedRuns` needs a status read and a parent-work read per slug there, since `keepsLockForFinalize` is a method on batten's `innerRunProducer` over its `Restepped` and `ParentHoldsTaskWork` seams, and those two sites carry only the report's slug, owner and verdict fields.
- `orch-strand-cycle` depends on this task and changes reed's strand liveness beside reed's session end, which D9 leaves as it is.
- `umbrella-daemon`'s dobby will read the owner record to find dead drivers.
