# Decision record — dobby-and-adopt-hardening

## Goal

A dobby survives every deploy by re-exec and never ends silently.
Its log records only changes plus a ten-minute heartbeat, and stays within a size bound.
A dobby counts and tidies only the landings its own machine made, so no landing is tidied on two machines.
The adopt machine check runs under its own gate key `adopt`, reports each tier and its end to the orch, and shows its progress in `lym beam status`.

## Scope

In:

- Part 1, issue #880: the dobby's start path after a re-exec, the lock wait at start, a log line and an orch notice for every way a dobby ends or fails to re-exec, and a tier1 test that replaces the binary under a running dobby.
- Part 2: `dobby.log` holding only the dobby's own lines, changes only, a heartbeat every ten minutes, rotated to one previous file at a size bound; the detached process's raw stdout and stderr in a file of their own.
- Part 3: the landing machine recorded on the batten run's Land history entry, and the dobby's landing count reading only this machine's landings.
- Part 4: the `adopt` gate key, a hidden check verb that runs it and reports per tier, a progress record, and its line in `lym beam status`.
- The docs of every touched package (`internal/dobbyengine`, `internal/beamcli`, `internal/adoptcli`, `internal/gateslot`, `internal/gatecli`, `internal/shedengine`) in the same commits.

Out:

- A trace file for the dobby through `internal/logger`: the dobby's own events go to `dobby.log` (Decision 5).
- Keeping the flock held across the exec by fd inheritance (support log, Rejected alternatives).
- A window or strand for the adopt check, ruled out by the entry's operator decision for part 4.
- Tidying on one machine the landings another machine made and never tidied (Decision 8's bound).

## Decisions

### 1. A dobby takes its lock before anything else, with a grace, and the bare pre-check goes

`runDobby` no longer calls `dobbyengine.Live` before `Run`.
`Run`'s own graced acquisition is the one decision whether this process is the dobby, and the run-process registration (`registerDobby`) runs only once the lock is held, through a seam `Run` calls after acquiring.
A process that loses the lock still prints `{"already_running":true,"ok":true}` and exits 0, and its run-process record is never written.

Rationale: the investigation in the entry places the lost dobby at the bare `TryAcquireWriteLock` pre-check, which any momentary probe defeats, while `Run` already waits `lock.ProbeGrace` for exactly that reason.
Every lock prober (`lym batten overview`, `lym orch watch`, `lym beam status`, a pass child's dobby state read, `EnsureLive`) holds the lock only between a try and a release.
Bound: `hubDobbyDeps` still runs before `Run`, so a fresh dobby is unregistered from its spawn until it holds the lock, at most the wiring time plus `lock.ProbeGrace`.
A beam dobby spawned inside a teardown of its own beam escapes that teardown.
Today the root's absence cannot mark the removal: every `internal/state` read and write calls `os.MkdirAll` on the record's directory, so the next tick's `UpdateState` recreates `<worktree>/<anchorRel>/.lym/beam`, and with it the removed root, and succeeds.
So the dobby tests, at the top of every loop iteration and not only on a probe tick, a marker its own writes cannot recreate, the worktree's git link at `Geometry.WorktreeRoot` joined with `.git`, and ends when that is gone, before the owner read and the tick.
Its own `dobby.log` went with the worktree, so this exit line goes to the prime's `dobby.log`, at a path the told Geometry gains beside `LogPath`, naming the beam's slug, appended without rotating since this process does not hold the prime's lock.
The `Deps.Log` seam and every other append of a line never create a missing log directory, so the exit's own writes create nothing at the removed worktree's path.
Bound: a tick already in flight when the worktree is removed can leave an empty `.lym/beam` stub there, holding the `state.json` and lock file its state access wrote, and the rest of that tick drives passes, starts tasks and runs the tidy step over the fresh empty `State` it read there.
Before it returns, the exit stops a board-tidy agent in flight through `TidyShuttle.StopStrand`, taking its strand from the `State.Tidy` the dobby keeps in memory, since the record went with the worktree, and logs the stop, or a failed stop with its error, to the prime's `dobby.log`.
The kept `State.Tidy`, nil included, is the one read by the last tick that ended with the marker present: the dobby tests the marker again as each tick ends and replaces the kept value with that tick's read only when the marker is there, since a tick that ends with it gone may have read a fresh empty `State` from a recreated record.
Bound: a tidy that the in-flight tick spawned is not stopped, and one it settled is stopped again, which removes a strand only when reed still tracks it, as after a failed stop in the settle, and logs a failed stop when reed's status probe fails.
The stop's shuttle runner is built over the prime's location, since the removed worktree's config is gone; finding no run record under it, `StopStrand` removes the strand alone.
A failed stop's line names `lym reed remove <guid>` as the way forward.
A re-exec keeps the old image's record, keyed on the same pid and start ticks, so only a fresh start has this window.

### 2. A re-exec'd dobby waits longer for its lock

The state record gains a start-ticks field, holding the dobby's own start ticks (`proc.StartTime`, which an exec keeps).
The re-exec test reads the state record and passes when it holds this process's pid and either this process's start ticks or none; a record with this pid and no start ticks is written by a binary from before this change, so the first re-exec onto the binary that ships this change is exactly that case.
Before acquiring, `Run` runs the test, and on a pass the start is a re-exec and the lock wait is `reexecLockWait`, ten seconds, instead of `lock.ProbeGrace`.
Any other start keeps `lock.ProbeGrace`.
Every notice of Decision 3 gated on the test treats the no-ticks record as a re-exec too.
Bound: a fresh dobby that reused the pid of a record from before this change after a reboot, and is then refused or fails, posts a re-exec notice its start does not deserve, a rare case that needs that record and that pid reuse together.

`recordStart` uses the same test in place of today's pid-only test, and is its one use that tells the no-ticks case apart: a start that fails the test owes a full pass and rewrites the pid, start time and start ticks.
A record holding this pid and no start ticks cannot tell a re-exec from a fresh dobby that reused the pid after a reboot, so `recordStart` gives it a full pass too, a harmless extra pass after a re-exec, and writes this process's start ticks into it, leaving every other field as Decision 4 says.
Every writer of `State.PID` writes the start ticks beside it, including `tick`'s refill of a record found without a pid.

Rationale: a re-exec releases the lock only through its own exec, so no other dobby can be legitimately live with this pid and start ticks in the record, and a deploy's `go build` loads the machine, so a descheduled prober can hold the lock past 250 ms.
The start ticks keep a fresh dobby that reuses a dead dobby's pid after a reboot from reading itself as a re-exec.
Bound: a second dobby that `EnsureLive` spawned inside the exec gap and that wins the lock still wins it.
A refused re-exec reads the state record again when its wait ends, and when the record then holds another pid it takes the refused-fresh path and posts nothing.
So if the re-exec'd process first reads the record before the second dobby's `recordStart`, it waits at most ten seconds, finds the second dobby's pid on the second read and posts nothing, since one dobby runs.
If it reads the record after, it finds another dobby's pid at once and takes the refused-fresh path.
Only a refused re-exec whose second read still holds its own pid posts the refused-re-exec notice.
Either way the hub keeps one dobby.

### 3. Every way a placed dobby ends leaves a line in `dobby.log`, and the abnormal ones post to the orch

Through the log seam of Decision 5 the dobby writes one line for each of: a start (fresh or re-exec, with pid), the re-exec onto a changed binary just before the exec, a failed exec with its error, a start refused because another dobby holds the lock, the exit because another machine owns the beam, the exit because its worktree is gone (Decision 1, a line in the prime's `dobby.log`), the stop on a cancelled context, and a fatal error out of `Run`.
A failed exec, a refused re-exec start whose second read of the state record still holds its own pid (Decision 2), a wiring failure after a re-exec, a `placeDobby` refusal after a re-exec and a fatal error out of `Run` after a re-exec also post one notice to the nearest live orch: a failed exec names the error, a refused re-exec names `lym beam start` as the way forward, and each of the other three names the error and `lym beam start` once its cause is fixed.
A fatal error out of `Run` posts through the `Deps.Notice` seam `Run` already holds, when the start passed Decision 2's re-exec test; on a fresh start, or when the state read is the step that failed so no test could run, it keeps its line only.
A refused fresh start posts nothing, since it is the idempotent case `EnsureLive` relies on.

`runDobby` has two exits before `Run`, and each leaves its trace:

- A `hubDobbyDeps` wiring failure has its Geometry, so `runDobby` writes its line with the error to `Geometry.LogPath` directly, without the `Deps.Log` seam it failed to build.
  When the state record passes Decision 2's re-exec test, the failure follows a re-exec, and `runDobby` also posts the failed-re-exec notice.
  It posts through `orchcli.NotifyFromScoped` over the worktree it resolves from `Geometry.WorktreeRoot` itself, outside `hubDobbyDeps`; when that resolution is the step that failed, the `dobby.log` line is the only trace.
- A `placeDobby` refusal made after `lymcwd.ResolveWorktree` succeeded, at `RequireDrivableWorktree`, `PrimeName`, `ReadOrigin`, the neither-prime-nor-beam check, the prime's resolve or `hubgeom.DobbyGeometry`, still holds the resolved location.
  `runDobby` derives that worktree's own `.lym/beam` directory from it by the rule `hubgeom.DobbyGeometry` uses, through one `hubgeom` function both share, and appends the refusal line with its error to that `dobby.log` without rotating.
  It runs Decision 2's re-exec test on that directory's `state.json`, and on a re-exec posts the failed-re-exec notice through `orchcli.NotifyFromScoped` over that location, naming the refusal's error and its way forward.
  A worktree with no `.lym/beam` directory, such as a pair that was never a beam, gets no line, since no append creates a directory (Decision 1), and holds no record, so it posts nothing.
- A `placeDobby` refusal at `filepath.Abs` or `lymcwd.ResolveWorktree` has no location and no log path, so its error envelope on its stdout is its only trace; this is the bound for those two steps.
  That is `dobby.out` for a process this binary spawned, and `dobby.log` for a re-exec'd dobby whose fds a binary from before this change pointed there, since its reopen (Decision 5) comes only after `placeDobby` returns.

The pre-exec line needs `internal/selfexec` to tell its caller that an exec is about to happen: `Watcher` gains a form of `Check` taking a callback run right before the exec and one run with the error after a failed exec.
`selfexec` stays a leaf over the standard library and `internal/logger`.

Rationale: the entry's Expected section asks that a failed re-exec is logged with its error and posted, never a silent exit, and the investigation found no line that named the exit path.

### 4. The tidy in flight survives the re-exec, as the state record already provides

No change to the tidy record: any start keeps `State.Tidy`, since a start rewrites only the record's pid, start time and start ticks, and the tidy step settles a tidy in flight whatever pid wrote it, so a re-exec spawns no second agent and settles the first.
Decision 1 makes the re-exec'd process reach `Run`, which is the part that failed in the incident.
The worktree-gone exit, which ends the dobby for good, stops the tidy in flight instead (Decision 1).

### 5. `dobby.log` holds only the dobby's own lines and is rotated at a size bound

The `Deps.Log` seam opens `dobby.log` for append per line and closes it; Decision 3's wiring failure, which has no seam, appends its one line the same way, and no raw process output reaches the file.
Before a write that would take the file past 1 MiB, it renames `dobby.log` to `dobby.log.1`, replacing an older `.1`, and starts a fresh file.
Only the holder of a file's own dobby lock rotates `dobby.log`, and `dobby.out` where `selfexec` execs; any other writer, such as Decision 3's refused start or wiring failure and a beam dobby's worktree-gone line in the prime's file (Decision 1), appends its line without rotating, so two renames never race and lose a `.1`.
`SpawnDetached` sends the detached process's own stdout and stderr, which carry its final JSON envelope, `logger` lines that reach stderr and a runtime panic, to `dobby.out` beside it instead of `dobby.log`, opened for append.
Where `selfexec` execs, `runDobby`, once `placeDobby` returns its Geometry and before anything else writes, reopens its own stdout and stderr onto `dobby.out` for append, so a dobby whose fds a binary from before this change pointed at `dobby.log`, and which lives on by re-exec without a respawn, moves them off `dobby.log` at its first start on this binary.
There the process that takes the lock, at each start, fresh or re-exec, rotates `dobby.out` by the same rule when it is past the bound and reopens its stdout and stderr onto the fresh file, a rename these platforms allow on a file the process holds open.
Where `selfexec` does not exec, as on Windows, the dobby holds `dobby.out` open for its whole life and Windows refuses to rename an open file, so the spawner rotates it instead, before opening it and only when `Live` reads the dobby lock free.
That spawner holds no lock, so this is the one rotation a process without the dobby lock makes, the exception to the lock-holder rule above; reading the lock free through `Live` bounds it, and the rename race the Bound names is what the exception leaves.
Both paths come from `dobbyengine.Geometry`, as `LogPath` does now.

Rationale: a file the process holds as its fd 1 and 2 cannot be rotated by rename, since writes follow the old inode; a per-line open of a separate file can.
`dobby.log` stays at its fixed place beside `state.json`, where the operator and the investigation read it.
Bound: `dobby.out` is bounded only at a start of the lock holder, or on Windows at a spawn; between them it grows by the process's stderr alone, which holds no per-pass line and whose `logger` lines follow the once-per-episode policy.
A process that loses the lock, or is refused before it, writes its envelope to the file it holds, which a winner's rotation may have renamed to `dobby.out.1`.
On Windows, two spawners that both read the lock free can race their renames and lose a `.1`, and a rename that fails, because a refused process still holds the file, is logged through `logger` and leaves the file to the next spawn.

### 6. The log records changes only, plus a ten-minute heartbeat

The `pass <slug> <kind>` line per pass goes.
The dobby logs, besides Decision 3's lines: a task start, each change of a run's batten state it drives as a tick reads it (to `halted`, `paused`, `done` and so on, with the producer for a halt), the first failing pass of an episode with its error text and the pass that ends the episode, the first failing tick of an episode with its error and the tick that ends it, the first unreadable owner read of an episode with its error and the read that recovers, and a board-tidy spawn, its report and its timeout.
The tick and owner-read lines come from the episodes the loop already keeps (`failures`, `ownerRead`), which today reach `dobby.log` only through stderr.
The run states it last logged are held in memory, so a re-exec logs no change for a state it did not see change.
A heartbeat line names each run the dobby drives with its batten state and producer, or `no runs`; it is written at each start and by the first tick ten or more minutes after the last heartbeat.
The interval is a package constant, not a `beam.yaml` key.

Rationale: the entry's Idea for part 2 lists these events and the ten-minute heartbeat, so a dead dobby shows in its log within the interval plus the longest gap between ticks, the larger of the loop's one-minute idle ceiling and the probe interval: eleven minutes at the recipe's `notice_probe_s` of 30.
The loop already wakes at least once a minute, so the heartbeat adds no wake (PATTERN-daemon-wakeups), and the failing-pass episode reuses the count the notice step already keeps.

### 7. A run's Land history entry names the machine that ran it

`shedengine` gains a told machine name on the shed it drives, stamped as a `machine` field on each history entry it appends when the name is non-empty; `shedengine` reads no machine name itself.
Batten's wiring tells it `machinename.Name()`; loom and darn tell none, so their status files are unchanged.
When `machinename.Name` fails, batten's wiring refuses the run with the error and the way forward to fix the machine's hostname, and never tells the shed an empty name.
The field is tolerant: a status file written before it decodes with an empty machine (PATTERN-tolerant-records).

Rationale: the Land Done entry carries no machine today (its `output` is empty), and the batten run record is the one landing record that reaches every machine through the records pull.

### 8. A dobby counts only the landings its own machine made

`battencli.DobbyLandings` returns only runs whose Land Done history entry names this machine.
An entry with no machine, written before Decision 7, is never counted.
Since a landing is counted on its landing machine alone, a tidy there covers it, and an adopt onto another machine never tidies it again.
A state record from before this change can already hold other machines' landings in `LandedSinceTidy`, and its entries carry no machine to tell them apart.
So the first landing count on this binary with no tidy in flight empties `LandedSinceTidy` once, before deciding a tidy is due, and sets a tolerant flag in the record that marks the list as holding only this machine's landings; a tidy in flight when the binary arrives settles its own slugs first, as Decision 4 says.

Rationale: the entry's Idea for part 3 asks that a dobby counts only the landings of a root or beam its machine owns, and that a landing is tidied once for every machine; Land runs only on the owning machine (PATTERN-owner-machine), so the landing machine is the owner at the time of landing, and no shared tidy mark has to be written to Bolt.
Bound: landings a machine made and had not yet tidied when ownership moved away stay untidied until that machine owns the worktree again; its next tidy there includes them, since its state record still lists them as landed since the last tidy.
Only a finished tidy (`settleTidy`) and the one emptying at the first count on this binary clear `LandedSinceTidy`; no start and no owned-elsewhere exit resets it.
That emptying also drops this machine's own landings counted before the deploy and not yet tidied; no dobby tidies them, as none tidies a landing whose entry names no machine.
The emptying is a one-time step rather than a drop, at every count, of each slug the machine-filtered landings no longer return: run cleanup removes a run's directory, so such a drop would also lose this machine's own landings whose runs were cleaned up before their tidy.

### 9. The adopt check runs under the gate key `adopt`

`gateslot` reads a new key `adopt`, with `scope: all`, its template value `{tiers: [0], scope: all}`.
`gateslot`'s gate reconcile migration writes `adopt` with the value of the hub's `finalize` key into a hub `gate.yaml` that tunes `finalize`, whenever the file holds no `adopt`, so a hub's check keeps the tiers it runs today.
`lym adopt` runs the check under `adopt`, and every text that names `--gate finalize` for it names `--gate adopt`.

Rationale: the entry asks for a gate key of the adopt check's own, by the rule one gate key per producer, named after it.
Copying `finalize`'s tuned value is a unit the hub tuned, never a template fill (PATTERN-sparse-hub-base), and it lands through the reconcile only (PATTERN-config-layers).

### 10. A hidden `lym adopt check` runs the gate, records progress and reports to the orch

`lym adopt` spawns the hidden `lym adopt check` detached in the prime, the spawn logged through `logger`, in place of the bare `lym gate test --gate finalize --wait`.
It spawns the check with its own stdout and stderr on `check.out` beside `check.json`, opened for append and never `suite.log`, so the gate's output alone reaches `suite.log`, a losing check writes its envelope to `check.out` and leaves `suite.log` untouched, and a panic of the check is on disk.
When the check does not start, `lym adopt`'s warning names `lym fabric reconcile` from the prime before `lym gate test --gate adopt`, since no reconcile of the hub's config ran and `lym gate test` runs none, while `lym fabric reconcile` runs `gateslot`'s migration, so Decision 9's migration has written `adopt` before the hand run reads it.
The verb resolves the prime from the hub whatever its cwd.
Before it touches anything, it takes a non-blocking lock beside `check.json` in `fabricengine.AdoptScratchDir`.
A check that finds the lock held exits 0 without writing `suite.log`, `check.json` or any orch post.
Holding the lock, the verb opens `suite.log` itself, truncated, and writes a fresh running record; the spawner no longer truncates `suite.log`.
It then calls `hubreconcile.Ensure` over the hub before it loads the gate key, so Decision 9's migration has run whenever the check reads `adopt`.
A reconcile that fails ends the check as `failed` before any tier runs: the record and the end notice name the reconcile's error and its way forward, so no check goes green on the template's tiers.
The verb runs the gate in-process through `gatecli`'s own test function, waiting for slots in `gateslot` as today, told a per-tier observer.
It writes the progress record `check.json` beside `suite.log`, through `internal/state`: its pid, its process start ticks (`proc.StartTime`, as `runproc` keys its records), start time, the gate key, each tier with its outcome, counts and duration as it ends, the tier running, and the end state `green` or `failed` with its time.
After each tier it posts one line to the nearest live orch of the prime: the tier, green or failed, the counts and the duration.
At the end it posts one notice, green or failed, a failure naming the failing tier, `suite.log` and the rerun `lym gate test --gate adopt` in the prime.
The gate's output still goes to `suite.log`.
A post that fails is logged and changes nothing; the record still carries the progress.
The adopt envelope's `suite` names the check's pid, the log and the record.
When `check.json` reads running and its process is alive by Decision 11's test, `lym adopt` spawns no check and its envelope's `suite` names the running one.

Rationale: the operator's decision for part 4 sets a line per tier to the orch, a notice at the end and progress in `lym beam status`, with no window or strand.
Running the gate in-process keeps one gate function (PATTERN-gate-slot) and parses no gate output.
Bound: the hidden verb adds an entry point that runs the hub's gate; it runs only the `adopt` key, in the prime.
The gate slots do not serialize two checks, since a hub may hold more than one slot; the lock beside `check.json` does, so one check at a time writes the record, the log and the posts.
Between a spawn and the child's taking the lock, `lym beam status` can still show the previous record, and a second `lym adopt` can spawn a check that loses the lock and exits; either spawned check can win the lock, so either envelope's pid can name the check that lost it, while both name the same log and record.

### 11. `lym beam status` shows the adopt check

`lym beam status` reads the progress record and shows an adopt check section: running with the tier in flight and the tiers done, or green or failed with the end time, and the log path.
A record that reads running shows `ended without a verdict`, naming `suite.log`, `check.out` and the rerun, when its pid is not alive or the live process's start ticks differ from the record's, as after a reboot that reused the pid.
No record shows nothing.

Rationale: the operator's decision for part 4.

## Constraints

- PATTERN-daemon-wakeups: the heartbeat and the logging ride the loop's existing ticks; no new wait or wake.
- PATTERN-leaf-packages: `internal/selfexec` keeps its closed import set.
- PATTERN-cwd-resolution: `dobbyengine` takes the new `dobby.out` path and the prime's `dobby.log` path from its told Geometry and imports no `lymcwd`.
- PATTERN-durable-vs-ephemeral-state: `dobby.out`, `dobby.out.1`, `dobby.log.1`, `check.json`, `check.out` and the check's lock file are never-tracked `.lym` files, named only where their siblings are named now.
- PATTERN-tolerant-records: the history entry's `machine` field, the dobby state record's start-ticks field and landings flag, and `check.json` are tolerant, written through `internal/state`.
- PATTERN-spawn-observability: the detached `lym adopt check` spawn is logged.
- PATTERN-pane-text-injection: every orch notice goes through the existing creel notice path.
- PATTERN-gate-slot: the check runs the gate through the gate's one function, in held slots.
- PATTERN-config-layers and PATTERN-sparse-hub-base: the `adopt` key reaches a hub file only through the reconcile migration.
- PATTERN-test-speed: the binary-replacing test needs a real exec, which `internal/selfexec` refuses under `go test`, and the prebuilt lym, so it is tier1, whose tier sets `prebuilt_lym`; the fd-reopen test needs a real detached process and the reopen applies only where `selfexec` execs, so it rides that tier1 test; everything else is tier0 with injected clocks.

## Auto-mode assumptions

- The ten-second re-exec lock wait (Decision 2), the 1 MiB rotation bound (Decision 5) and the ten-minute heartbeat (Decision 6) are package constants; no config key was asked for.
- `dobby.out` is the name of the raw output file; the plan may rename it.
- A refused fresh start posts no orch notice (Decision 3).
- Landings with no machine on their Land entry are never counted (Decision 8).

## Open risks

- The investigation put a 40 percent confidence on a momentary probe being what the pre-check saw held; a SIGKILL or OOM kill could not be ruled out from the logs.
  Decisions 1 to 3 cover the probe and the second-dobby cases and leave a line for every exit the process itself takes once its worktree resolves, a refusal at `filepath.Abs` or `lymcwd.ResolveWorktree` leaving only its envelope on its stdout (Decision 3); a kill from outside still leaves only the heartbeat's silence.
- A hub that tunes `finalize` gets `adopt` with the same tiers through the reconcile; a hub whose operator wants a different adopt scope sets it through `lym config gate` afterwards.
  A hub returns to the template's `adopt` by setting it through `lym config gate`, never by deleting it, since the next reconcile copies `finalize`'s value back into a file with no `adopt`.

<!-- lym:carry-over Discussion-Review -->
- Discussion-Review, round 3: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lym/reviews/discussion/round-3-review.md; fixer report: _lym/reviews/discussion/round-3-fixer-report.md.
  - worktree-gone-root-recreated-by-state-access (class design, severity MEDIUM, open in rounds 3)
<!-- /lym:carry-over Discussion-Review -->

## Acceptance criteria

- A running dobby, started from a copy of the prebuilt lym, whose binary is replaced on disk while a goroutine loops on `dobbyengine.Live`, re-execs with its pid unchanged, holds its lock again, keeps a `State.Tidy` it held before, and writes the re-exec line and a re-exec start line to `dobby.log` (tier1).
- A re-exec'd dobby whose lock is held by another holder for longer than `lock.ProbeGrace` and shorter than the re-exec wait takes the lock and runs.
- A dobby whose lock another holder keeps past the wait exits, writes the refused line to `dobby.log`, posts the orch notice when it is a re-exec and posts none when it is a fresh start, and writes no run-process record.
  A refused re-exec whose second read of the state record finds another dobby's pid posts nothing.
- A re-exec'd dobby that reads another dobby's pid in the state record takes the refused-fresh path and posts nothing; a fresh dobby whose pid matches the record's but whose start ticks differ waits only `lock.ProbeGrace`, posts no re-exec notice, rewrites the record's start ticks and owes a full pass; a record with this pid and no start ticks takes the re-exec lock wait, owes a full pass and is left holding the start ticks, and a refused re-exec or a wiring failure on such a record posts its notice.
- A tick that refills a record found without a pid writes the start ticks beside it.
- A beam dobby whose worktree is removed exits at the top of its next loop iteration, writes the exit line naming its slug to the prime's `dobby.log`, creates nothing at the removed worktree's path through its exit, and stops a tidy agent it held in flight through a runner over the prime, logging the stop; a failed stop's line names `lym reed remove <guid>`.
- A beam dobby woken by a file event, not a probe tick, after its worktree is removed still exits, and a tidy held in `State.Tidy` is stopped.
- A failed exec writes its error to `dobby.log` and posts one orch notice, and the dobby keeps running.
- Each exit path of Decision 3 writes its line, including a `hubDobbyDeps` wiring failure, whose line reaches `dobby.log` and which, after a re-exec whose worktree resolves, posts the failed-re-exec notice naming `lym beam start`, and a fatal error out of `Run` after a re-exec, which writes its line and posts its notice.
- A re-exec'd dobby that `placeDobby` refuses after its worktree resolves writes its line to that worktree's `dobby.log` and posts one notice; a refusal with no resolved worktree, in a process this binary spawned, leaves its envelope in `dobby.out` and nothing in `dobby.log`.
- A dobby started with its stdout and stderr on `dobby.log` writes its later envelope and stderr lines to `dobby.out` after a start (tier1, in the binary-replacing test).
- A process holding no lock appends to a `dobby.log` past the bound without rotating it.
- A pass writes no line; a task start, a run's state change, the first failing pass of an episode and its recovery, the first failing tick of an episode and the tick that ends it, the first unreadable owner read of an episode and its recovery, a tidy spawn, report and timeout each write one.
- After a re-exec, the first tick logs no state-change line for a run whose state is unchanged, and the start heartbeat names that run.
- A heartbeat is written at start and again on the first tick ten minutes after the last, naming every run driven with its state, with an injected clock.
- `dobby.log` crossing 1 MiB rotates to `dobby.log.1`, replacing an older one; `dobby.out` past the bound at a start of the lock holder rotates the same way where `selfexec` execs, and on Windows at a spawn that reads the lock free, never at one that reads it held.
- A batten history entry carries the told machine; a loom shed told none writes no `machine`; a status file without the field decodes; batten's wiring refuses the run when `machinename.Name` fails.
- `DobbyLandings` returns a run whose Land Done names this machine, and skips one naming another machine or none.
- A state record from before this change holding another machine's landing in `LandedSinceTidy` spawns no tidy for it: the first count empties the list once and sets the flag, and a later count leaves the list as it is.
- A hub `gate.yaml` that tunes `finalize` and holds no `adopt` gains `adopt` with `finalize`'s value through the migration, and a second pass changes nothing; one that holds `adopt` keeps it.
  A check on a hub whose build stamp is stale runs `finalize`'s tuned tiers, and a check whose reconcile fails ends `failed` naming the reconcile's error, with no tier run.
- `lym gate test --gate adopt` runs the key's tiers over the whole tree.
- The check verb writes the progress record per tier, posts a line per tier and an end notice for a green and for a failing run, and a failed post leaves the record correct.
- A check started over an existing `check.json` and `suite.log` writes a fresh record and an emptied log.
- A second check started while one runs exits 0, writes its envelope to `check.out` and leaves the first's record and `suite.log` untouched, with no orch post.
- `lym beam status` shows a running, a green, a failed and a dead-pid check record as Decision 11 says, a running record whose pid is alive but whose start ticks differ as ended without a verdict, and nothing with no record.
- `lym adopt` with an owner write spawns the check, and its envelope names the pid, the log and the record; with a running check alive, it spawns none and its envelope names the running one; a check that does not start leaves a warning naming `lym fabric reconcile` before `lym gate test --gate adopt`.

## Notes for the plan writer

- The pre-exec callback of Decision 3 can be a variant of `Watcher.Check`; `Replaced` already shows the shape of a non-exec'ing read.
- The dobby's `CheckBinary` seam is wired in `internal/beamcli/dobbydeps.go`; the notice for a failed exec can use the same `Notice` seam the loop already holds.
- `envkit`'s gate table and `gatecli`'s gate key lists name the keys a test hub declares.
