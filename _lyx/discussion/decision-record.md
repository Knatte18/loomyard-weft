# Decision record — run-process-ownership

## Goal

Every process lyx starts for a run belongs to one owner record, keyed by the slug it serves or by the hub scope, so `lyx tender status` lists it and the verbs that end or tear down a run end all of it.
A pause leaves no network work parked as a live process, a teardown never waits on a run that has nothing left to do, and background work that is safe to freeze or that is not a session's work product never withholds a hibernate or holds a row.

## Scope

In:

- The owner record, `internal/runproc`, and registration of detached and long-lived run processes.
- Ending owned processes from pair teardown, `lyx reed down`, the origin reconcile's owned-elsewhere pause and a new `lyx tender reap` for orphans.
- Reed's pair-session end killing what its panes leave behind.
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

Decision: `End` takes the record directory, an owner and the caller's pid, and ends every live recorded process except the caller and its ancestors.
On Linux, for every record at once, it first reads the descendants of each non-leader record by parent pid, then sends SIGTERM to each leader's group and to each non-leader with its descendants, waits up to five seconds in all for them to be gone, and sends SIGKILL to what is left.
A group is signalled only while a live process still reports that group id with a start time at or after the recorded one, as `proc.KillTree` checks, and a pid only after its start time matches.
`End` then re-reads the owner's records and repeats on any live record it did not end, at most three passes, so a child registered during an end is reached.
On Windows, where `proc.StartTime` and `proc.IsAncestor` read nothing, `End` ends no recorded pid; the step loop still dies through its named job in the loop end, as today.
`End` returns each process it ended with its pid and role.
The five seconds and three passes are engine constants.
Rationale: git removes its lock files on SIGTERM, so a push, sync or commit ended mid-write leaves no `index.lock`; the loop's step child keeps its SIGKILL path through the loop end, as today.
A self-registered process that does not lead its group may share a group with the shell or agent that started it, so its group is never signalled.
Excluding the caller's ancestry lets batten's own Worktree-Teardown, run inside a registered `lyx batten run`, end the rest of its run.
Without a start time a pid cannot be told from a reused one, so Windows ends nothing it cannot identify.
Bound: `End` signals only processes recorded under the named owner whose identity still matches, and the descendants read from them before the first signal.

### D4 — Pair teardown and `lyx reed down` end the pair's owned processes first

Decision: `internal/pairteardown`'s session end calls `runproc.End` for the pair's slug before its loop end, so `lyx fabric remove`, batten's Worktree-Teardown and the origin reconcile's local removal end batten, the driver, the step loop and the pair's pushes before the session and the worktrees go.
After `End`, a driver lock still held means a process `End` could not reach, such as one an older binary started, and the teardown refuses with `ErrDriverBusy` naming the lock, with nothing removed.
`lyx reed down` in a pair calls `runproc.End` for the pair's slug before it ends the session, under the same readings as D5: it ends them at once when the run is paused, ended, owned elsewhere or not running, and refuses a running unpaused run with the way forward `lyx tender pause <slug>`, then `lyx reed down` again.
Each envelope carries the ended processes under `ended_processes`.
No lyx run process handles SIGTERM, so an ended driver, loop or batten exits at once and writes no drive report and posts no stop notice.
A killed batten leaves its run record reading `running` with no process; ending that record is `batten-cancel-verb`'s.
Rationale: ending batten and the driver before the strands keeps batten's dead-strand revival and the driver's re-step from racing the session end.
The driver-lock re-check keeps an unregistered process from driving on against a removed pair.
Bound: a running unpaused run is never ended by `fabric remove` (D5) or `reed down`; batten's own teardown passes no refusal, as today.

### D5 — `lyx fabric remove` and a run with nothing left to do

Decision: `lyx fabric remove` runs the run-scope origin reconcile for the slug first (`PATTERN-origin-reconcile`).
A `stopped` verdict refuses with the reconcile's way forward, as for every other run-scope caller, with nothing ended.
An `ended` or `owned_elsewhere` verdict removes the pair through `pairteardown`'s local mode: no archive-tag push, no remote branch step, no board write, and a `--remote` it drops is named in `remote_skipped_reason`.
A reconcile that already removed the pair itself is read as done, with its ended processes in the envelope.
The quiet wait's driver checks are skipped, and the owned processes ended, when the verdict is `ended` or `owned_elsewhere`, when a live pause entry covers the slug, or when the local board entry is done or the status history holds a Finalize Done and the batten record does not read `running`.
The run-lock wait still runs first, bounded as today, and a run lock still held at its bound is ended only when the holder is a step waiting at the network gate; any other holder refuses, naming its pid.
Otherwise the bounded quiet wait and its `ErrDriverBusy` refusal stand, and the refusal names the owned processes and the way forward `lyx tender pause <slug>`, then the removal again.
`lyx fabric remove` passes the network gate for its own calls under any pause, through `SetGateExempt`'s mechanism extended to every entry for that one process.
With origin unreachable, the local Bolt's owner and status decide, as for every other reconcile caller.
Rationale: a paused driver starts nothing until a resume, and an ended or foreign run's driver has nothing left to do here, so waiting on either only blocks the operator; a step still doing local work under a pause, such as a merge under the parent lock, is waited for through the run lock.
Batten still runs Land and deploy after Finalize Done, so that reading counts only once batten is no longer running.
Local mode is the reconcile's existing removal for ended entries, so a machine that does not own the run pushes no archive tag of stale records (#741).
An explicit removal is the operator acting through the pause, so its own calls pass; the reconcile it adds is a read of origin, the same kind as `fabric add`'s branch probe that `fabric-remote-async` keeps synchronous, and `PATTERN-origin-reconcile` requires it.
Bound: the driver-check skip lets through only the readings above; the gate exemption covers only the remove process's own calls, never its children; `--force` still answers dirtiness only.

### D6 — Under a pause, detached pushes and board syncs exit, and every lift spawns what they owe

Decision: a pause leaves the Go driver, the step loop, batten and a session's background shells running and parked; only detached pushes and board syncs stop.
A detached push or board-sync child, marked by a hidden flag its spawner sets (the push's existing pause-slug flag, a new hidden flag on the sync), checks its coverage before it marks a pending flag or takes its lock, and exits at once when covered.
A child that meets a covering pause at a later network call puts back the pending flags of the sides it has not pushed and exits.
An exit under a pause pushes nothing more, leaves the commits local, writes one log line and no failed-push record or creel mail, since the push is deferred, not failed.
Every other process keeps the gate's waiting behaviour.
`tenderengine`'s lift sequence, which `lyx tender resume`, the hibernate wake and cancel, `lyx adopt` and the cleanup reconcile all run, calls a spawn seam after it lifts its entries: one status push for each pair a lifted entry covered that still exists, no live entry still covers and no other machine owns, one prime records push, and one board sync when Bolt is ahead of its upstream.
Rationale: the parked driver chain is idle and is what carries a resumed agent's completion, and ending it on every pause would need every resume path to restart drivers mid-step; owned, listed and ended by every end verb, it no longer blocks or outlives anything.
This departs from the board entry's line that a pause "stops work instead of parking processes", which its Expected list allows as long as `fabric remove` ends them (D4, D5).
A parked push is a live process holding a stale write; left as commits and flags, it is a record the next push carries through the records replay, which sets aside an ended entry's commits.
A removed pair's owed push goes with the pair.
Bound: the no-wait behaviour covers only the two marked child kinds; the gate's contract that it only waits and fails no call holds for every other process.

### D7 — An owned-elsewhere pause ends the run's processes on the machine it left

Decision: where the origin reconcile reads a run `owned_elsewhere` and pauses it, at the hibernate wake and at `lyx orch start`, it also calls `runproc.End` for the slug.
Rationale: `0d953f140` left ending a run's processes on the machine it left to this task; a driver chain parked under the owner pause on that machine would otherwise outlive the adoption, the incident's pattern.
Bound: only for a slug the reconcile's own verdict names owned elsewhere.

### D8 — `lyx tender status` lists owned processes and orphans, and `lyx tender reap` ends orphans

Decision: `lyx tender status` gains a processes part: each live owned process per owner, with role, pid and age.
A record is an orphan when its owner is a slug, not the hub scope, that has no pair (no registered linked task worktree) and whose batten record does not read `running`.
The part also lists, as orphans, unregistered processes running a lyx executable whose working directory is a deleted directory inside the hub, leaving out those the replaced-binary part lists.
`lyx tender reap` ends every orphan of one fresh reading: registered ones through `runproc.End`, unregistered ones by pid after a start-time match from that reading.
It takes `lyx tender pause`'s caller check with no slug: the orch and an external caller may run it, an agent is refused with the way forward of asking its parent.
On a platform that cannot list processes both report the part unsupported, as the replaced-binary part does.
Rationale: once a pair is removed, no `remove` or `reed down` target is left, so one verb ends what survived, and status names it as the way forward.
Bound: `reap` never ends a hub-scope process, a process whose pair exists, or one whose batten record reads `running`; those are ended by the pair's end verbs or by `batten-cancel-verb`.

### D9 — Reed's pair-session end kills what its panes leave behind

Decision: reed's end of a pair's session, both `Engine.Down` and `EndSessionByName`, reads on Linux each pane's process descendants with their start times before it kills the tmux session, and afterwards sends SIGKILL to each that is still alive with the same start time, except a process running a lyx executable and a process registered in any owner record.
The prime's session is left as today.
Rationale: an agent's background shells, such as the `find /` of issue #796, are the Claude process's children and survive the pane's hangup; reed owns the panes, so it is the one that can name them.
Lyx processes are owned by the record and ended through `End`, and the prime session's panes start hub-level daemons this task leaves alone.
Bound: only non-lyx, unregistered descendants of a pair session's pane processes at the reading, matched by start time.

### D10 — Hibernate confirmation reads safe background work as settled

Decision: a confirmation reading counts a session that reads busy only on background work as settled when every outstanding task is a shell.
A session with an outstanding fork still withholds.
Confirmation fails at ten minutes unless every withholding reason of that reading is fork work, and at sixty minutes in all whatever the reasons, both engine constants.
When the request's requester is the orch, the orch's own session is left out of the reading.
Rationale: a suspend freezes a shell and the wait loop it holds and resumes them on wake; a fork is never interrupted (operator view, 2026-10-15) and its next batch is already held by the pause (`fac869bb3`), so confirmation lets it finish, and sixty minutes covers the longest fork seen.
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
- `PATTERN-owner-machine`: D5's local mode, D6's owner exclusion and D7 take the reconcile's or `boardengine`'s owner verdict; no owner is set here.
- `PATTERN-spawn-observability`: `Register` and `End` log at Info; a detached spawn still logs its spawn at its site.
- `PATTERN-told-geometry` and `PATTERN-lyxdirs-single-declarer`: the record directory is declared beside `HubScratchDir` and handed to `runproc`.
- `PATTERN-tolerant-records`: a record file is read tolerantly and written through `internal/state`.
- `PATTERN-shuttle-provider-seam`: D11 decides over provider-neutral `BackgroundTask` kinds.
- `PATTERN-shuttle-stop` and `PATTERN-guarded-interrupt`: no strand is stopped and no key is sent outside shuttle and reed; D9 kills processes after reed's own session kill.
- `PATTERN-pause-holds-prompts`: D6 changes only the two marked child kinds.
- `PATTERN-push-failure-visible`: a push deferred under a pause is not a failure; a push that fails after a lift follows the rule.
- Tests inject clocks for D3's grace and D10's bounds.
- Fact: Linux keeps a process-group id unassigned while any member lives, so a group match with a start time at or after the recorded one is the recorded group (`internal/proc/tied_linux.go`, `killTreePlatform`).
- Fact: `proc.Detach` on Linux starts the child with `Setsid`, so a detached child leads its own group (`internal/proc/proc_linux.go`).
- Fact: `proc.StartTime` reads nothing and `proc.IsAncestor` is false on Windows (`internal/proc/proc_windows.go`).
- Fact: git's lockfile code removes held lock files on a fatal signal it catches, SIGTERM included (git `lockfile.c`/`tempfile.c` sigchain cleanup; git 2.43 on the incident host).
- Fact: no lyx run process installs a SIGTERM handler; only the orch watcher, creel and the gate child do.
- Fact: the pending-flag holder takes its owed flags before its network step (`internal/fabricengine/pendingflags.go`), so D6's check sits before the flag is marked.
- Fact: the Claude Stop payload's `background_tasks` entries carry a type, `subagent` for a fork and any other for a shell (`internal/shuttleengine/claudeengine/background.go`).
- Fact: `tenderengine.Lift` is called by tender resume, the hibernate wake and cancel, `lyx adopt` and the cleanup reconcile, so D6's spawn lives in the lift sequence.
- Fact: a hibernate request records its requester (`internal/hibernateengine/request.go`).

## Auto-mode assumptions

- The board entry's "one verb ends them all" is met for a live run by `batten-cancel-verb`'s verb over `runproc.End`, and for a removed pair's survivors by `lyx tender reap`.
- `lyx tender reap` is the only new verb; the board sync's no-wait marker is a hidden flag, not a verb.

## Open risks

- A process an older binary started is never registered; D4's driver-lock re-check refuses a teardown it would outrun, and D8's deleted-directory rule or `ps` finds the rest.
- A self-registered `lyx batten run` that does not lead its group is ended with the descendants read at the end; a descendant that already reparented is missed.
- D9 reads pane descendants once before the kill; a process spawned between the reading and the kill survives.
- D11 finishes a gated run while an unawaited shell still runs; the shell lives until the pair's session end (D9) or its own exit.
- An ungated run waiting on a long shell no awaited prefix names now stalls after `BackgroundShellWaitMin`.
- A push holder ended by `End` on a live pair loses the pending flag it took; its commits wait for the next push of that worktree.
- On Windows, `End` ends only the step loop; the other owned processes are listed but never ended there.

## Acceptance criteria

- A detached pair push, a prime records push, a board sync, `lyx loom drive`, the detached step loop and a seeded `lyx batten run` each leave a record under the owner D2 names, and a dead process's record is pruned on the next read.
- `runproc.End` ends a recorded group leader with its group and a non-leader with its descendants, skips the caller and its ancestors, leaves a reused pid alone, and reaches a child registered during the end.
- `lyx fabric remove` on a paused run with a parked driver, step loop and batten, under a slug pause and under a hub pause, ends them and removes the pair, naming them in `ended_processes`.
- `lyx fabric remove` under a pause while a step holds the run lock doing local work waits for it; a step waiting at the network gate is ended.
- `lyx fabric remove` on an entry the reconcile reads `ended` or `owned_elsewhere` removes the pair in local mode, and on `stopped` refuses.
- `lyx fabric remove` after Finalize Done while batten reads `running` does not skip the quiet wait.
- A teardown whose driver lock stays held after `End` refuses with nothing removed.
- `lyx reed down` in a paused pair ends its owned processes before the session, so no strand is revived; on a running unpaused run it refuses.
- Batten's Worktree-Teardown ends the rest of its run without ending its own process, and never a prime records push.
- A detached push or board sync under a covering pause exits without pushing, without a failed-push record, and with its pending flags in place; each lift path spawns the owed pushes for the pairs it uncovered, none for a removed pair or one owned elsewhere.
- A reconcile owned-elsewhere pause at wake or `lyx orch start` ends that run's owned processes.
- `lyx tender status` marks an orphan only for a slug with no pair and no running batten record, never the hub scope, and lists an unregistered lyx process in a deleted hub directory; `lyx tender reap` ends exactly those and refuses an agent.
- A pair session's end kills a surviving non-lyx pane descendant, matched by start time, and no lyx or registered process.
- Hibernate confirmation confirms with sessions busy only on shells, withholds on a fork up to sixty minutes, and leaves out the orch's session when the orch requested.
- A gated shuttle run whose writer wrote a passing verdict and then started a non-awaited shell finishes Done at that turn end; with a fork outstanding it still waits.
- An ungated run with missing outputs and only non-awaited shells outstanding past `BackgroundShellWaitMin` finalizes stalled.
- `internal/runproc` gets its `doc.go`, every package whose behaviour a Decision changes updates its doc, and `docs/overview.md` its module table.

## PATTERN changes

- `PATTERN-run-process-owner` — adds: Starting a detached or long-lived process for a run: register it in `internal/runproc`'s owner record; ending a run's processes: only via `runproc.End`.
- `PATTERN-pair-teardown` — changes: Every teardown of a pair goes through `internal/pairteardown`, which ends the pair's owned processes and then its reed session before any worktree is removed; its driver checks are skipped for a pair that is paused, ended or owned by another machine, never its run-lock wait.
- `PATTERN-leaf-packages` — changes: adds `internal/runproc`, importing only the standard library, `internal/proc`, `internal/state` and `internal/logger`.

## Notes for the plan writer

- The loop end in `pairteardown/loop.go` already kills the loop and its step tree; `runproc.End` runs before it and changes neither the loop pid file nor the teardown mark.
- `OriginSeams`' `LivePair` already reads a slug under a live hub pause as not live; D5's paused reading can share that rule, and the reconcile's `Teardown` seam takes the same skip, since it lives in `pairteardown`.
- `pauserecord.SetGateExempt` is the mechanism D5's remove exemption extends.
- `orch-strand-cycle` depends on this task and changes reed's strand liveness beside D9's session end.
- `umbrella-daemon`'s dobby will read the owner record to find dead drivers.
