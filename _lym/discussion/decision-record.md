# Decision record — dobby-and-adopt-hardening

## Goal

A dobby survives every deploy by re-exec and never ends silently.
Its log records only changes plus a ten-minute heartbeat, and stays within a size bound.
A dobby counts and tidies only the landings its own machine made, so no landing is tidied on two machines.
The adopt machine check runs under its own gate key `adopt`, reports each tier and its end to the orch, and shows its progress in `lym beam status`.

## Scope

In:

- Part 1, issue #880: the dobby's start path after a re-exec, the lock wait at start, a log line and an orch notice for every way a dobby ends or fails to re-exec, and a tier1 test that replaces the binary under a running dobby.
- Part 2: `dobby.log` written only by the dobby's own log seam, changes only, a heartbeat every ten minutes, rotated to one previous file at a size bound; the detached process's raw stdout and stderr in a file of their own.
- Part 3: the landing machine recorded on the batten run's Land history entry, and the dobby's landing count reading only this machine's landings.
- Part 4: the `adopt` gate key, a hidden check verb that runs it and reports per tier, a progress record, and its line in `lym beam status`.
- The docs of every touched package (`internal/dobbyengine`, `internal/beamcli`, `internal/adoptcli`, `internal/gateslot`, `internal/gatecli`, `internal/shedengine`) in the same commits.

Out:

- A trace file for the dobby through `internal/logger`: the dobby's own events go to `dobby.log` (Decision 5), and the trace sink's placement under the hub's `.lym/logs` is the subject of `test-logs-leak-into-bolt`.
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

### 2. A re-exec'd dobby waits longer for its lock

Before acquiring, `Run` reads the state record; when it holds this process's pid, the start is a re-exec and the lock wait is `reexecLockWait`, ten seconds, instead of `lock.ProbeGrace`.
Any other start keeps `lock.ProbeGrace`.

Rationale: a re-exec releases the lock only through its own exec, so no other dobby can be legitimately live with this pid in the record, and a deploy's `go build` loads the machine, so a descheduled prober can hold the lock past 250 ms.
Bound: a second dobby that `EnsureLive` spawned inside the exec gap and that wins the lock still wins it; the re-exec'd process then exits after at most ten seconds through Decision 3's refused-re-exec path, and the hub keeps one dobby.

### 3. Every way a dobby ends leaves a line in `dobby.log`, and the abnormal ones post to the orch

Through the log seam of Decision 5 the dobby writes one line for each of: a start (fresh or re-exec, with pid), the re-exec onto a changed binary just before the exec, a failed exec with its error, a start refused because another dobby holds the lock, the exit because another machine owns the beam, the stop on a cancelled context, and a fatal error out of `Run`.
A failed exec and a refused re-exec start also post one notice to the nearest live orch, naming `lym beam start` as the way forward for a refused re-exec and the error for a failed exec.
A refused fresh start posts nothing, since it is the idempotent case `EnsureLive` relies on.

The pre-exec line needs `internal/selfexec` to tell its caller that an exec is about to happen: `Watcher` gains a form of `Check` taking a callback run right before the exec and one run with the error after a failed exec.
`selfexec` stays a leaf over the standard library and `internal/logger`.

Rationale: the entry's Expected section asks that a failed re-exec is logged with its error and posted, never a silent exit, and the investigation found no line that named the exit path.

### 4. The tidy in flight survives the re-exec, as the state record already provides

No change to the tidy record: a re-exec reads its own pid in the record and keeps `State.Tidy`, so it spawns no second agent and settles the first.
Decision 1 makes the re-exec'd process reach `Run`, which is the part that failed in the incident.

### 5. `dobby.log` is written only by the dobby's log seam and rotated at a size bound

The `Deps.Log` seam opens `dobby.log` for append per line and closes it.
Before a write that would take the file past 1 MiB, it renames `dobby.log` to `dobby.log.1`, replacing an older `.1`, and starts a fresh file.
`SpawnDetached` sends the detached process's own stdout and stderr, which carry its final JSON envelope, `logger` lines that reach stderr and a runtime panic, to `dobby.out` beside it instead of `dobby.log`, and applies the same rotation to `dobby.out` once at each spawn, before opening it.
Both paths come from `dobbyengine.Geometry`, as `LogPath` does now.

Rationale: a file the process holds as its fd 1 and 2 cannot be rotated by rename, since writes follow the old inode; a per-line open of a separate file can.
`dobby.log` stays at its fixed place beside `state.json`, where the operator and the investigation read it.
Bound: `dobby.out` is bounded only at a spawn; between spawns it grows by the process's stderr alone, which holds no per-pass line and whose `logger` lines follow the once-per-episode policy.

### 6. The log records changes only, plus a ten-minute heartbeat

The `pass <slug> <kind>` line per pass goes.
The dobby logs, besides Decision 3's lines: a task start, each change of a run's batten state it drives as a tick reads it (to `halted`, `paused`, `done` and so on, with the producer for a halt), the first failing pass of an episode with its error text and the pass that ends the episode, and a board-tidy spawn, its report and its timeout.
The run states it last logged are held in memory, so a re-exec logs no change for a state it did not see change.
A heartbeat line names each run the dobby drives with its batten state and producer, or `no runs`; it is written at each start and by the first tick ten or more minutes after the last heartbeat.
The interval is a package constant, not a `beam.yaml` key.

Rationale: the entry's Idea for part 2 lists these events and the ten-minute heartbeat, so a dead dobby shows in its log within ten minutes.
The loop already wakes at least once a minute, so the heartbeat adds no wake (PATTERN-daemon-wakeups), and the failing-pass episode reuses the count the notice step already keeps.

### 7. A run's Land history entry names the machine that ran it

`shedengine` gains a told machine name on the shed it drives, stamped as a `machine` field on each history entry it appends when the name is non-empty; `shedengine` reads no machine name itself.
Batten's wiring tells it `machinename.Name()`; loom and darn tell none, so their status files are unchanged.
The field is tolerant: a status file written before it decodes with an empty machine (PATTERN-tolerant-records).

Rationale: the Land Done entry carries no machine today (its `output` is empty), and the batten run record is the one landing record that reaches every machine through the records pull.

### 8. A dobby counts only the landings its own machine made

`battencli.DobbyLandings` returns only runs whose Land Done history entry names this machine.
An entry with no machine, written before Decision 7, is never counted.
Since a landing is counted on its landing machine alone, a tidy there covers it, and an adopt onto another machine never tidies it again.

Rationale: the entry's Idea for part 3 asks that a dobby counts only the landings of a root or beam its machine owns, and that a landing is tidied once for every machine; Land runs only on the owning machine (PATTERN-owner-machine), so the landing machine is the owner at the time of landing, and no shared tidy mark has to be written to Bolt.
Bound: landings a machine made and had not yet tidied when ownership moved away stay untidied until that machine owns the worktree again; its next tidy there includes them, since its state record still lists them as landed since the last tidy.

### 9. The adopt check runs under the gate key `adopt`

`gateslot` reads a new key `adopt`, with `scope: all`, its template value `{tiers: [0], scope: all}`.
`gateslot`'s gate reconcile migration writes `adopt` with the value of the hub's `finalize` key into a hub `gate.yaml` that tunes `finalize` and holds no `adopt`, once, so a hub's check keeps the tiers it runs today.
`lym adopt` runs the check under `adopt`, and every text that names `--gate finalize` for it names `--gate adopt`.

Rationale: the entry asks for a gate key of the adopt check's own, by the rule one gate key per producer, named after it.
Copying `finalize`'s tuned value is a unit the hub tuned, never a template fill (PATTERN-sparse-hub-base), and it lands through the reconcile only (PATTERN-config-layers).

### 10. A hidden `lym adopt check` runs the gate, records progress and reports to the orch

`lym adopt` spawns the hidden `lym adopt check` detached in the prime, the spawn logged through `logger`, in place of the bare `lym gate test --gate finalize --wait`.
The verb runs the gate in-process through `gatecli`'s own test function, waiting for slots in `gateslot` as today, told a per-tier observer.
It writes the progress record `check.json` beside `suite.log` in `fabricengine.AdoptScratchDir`, through `internal/state`: its pid, start time, the gate key, each tier with its outcome, counts and duration as it ends, the tier running, and the end state `green` or `failed` with its time.
After each tier it posts one line to the nearest live orch of the prime: the tier, green or failed, the counts and the duration.
At the end it posts one notice, green or failed, a failure naming the failing tier, `suite.log` and the rerun `lym gate test --gate adopt` in the prime.
The gate's output still goes to `suite.log`, truncated at each start; the record is rewritten fresh at each start.
A post that fails is logged and changes nothing; the record still carries the progress.
The adopt envelope's `suite` names the check's pid, the log and the record.

Rationale: the operator's decision for part 4 sets a line per tier to the orch, a notice at the end and progress in `lym beam status`, with no window or strand.
Running the gate in-process keeps one gate function (PATTERN-gate-slot) and parses no gate output.
Bound: the hidden verb adds an entry point that runs the hub's gate; it runs only the `adopt` key, in the prime, and the gate slots serialize it with every other gate run as today.

### 11. `lym beam status` shows the adopt check

`lym beam status` reads the progress record and shows an adopt check section: running with the tier in flight and the tiers done, or green or failed with the end time, and the log path.
A record that reads running whose pid is not alive shows `ended without a verdict`, naming `suite.log` and the rerun.
No record shows nothing.

Rationale: the operator's decision for part 4.

## Constraints

- PATTERN-daemon-wakeups: the heartbeat and the logging ride the loop's existing ticks; no new wait or wake.
- PATTERN-leaf-packages: `internal/selfexec` keeps its closed import set.
- PATTERN-cwd-resolution: `dobbyengine` takes the new `dobby.out` path from its told Geometry and imports no `lymcwd`.
- PATTERN-durable-vs-ephemeral-state: `dobby.out`, `dobby.log.1` and `check.json` are never-tracked `.lym` files, named only where their siblings are named now.
- PATTERN-tolerant-records: the history entry's `machine` field and `check.json` are tolerant, written through `internal/state`.
- PATTERN-spawn-observability: the detached `lym adopt check` spawn is logged.
- PATTERN-pane-text-injection: every orch notice goes through the existing creel notice path.
- PATTERN-gate-slot: the check runs the gate through the gate's one function, in held slots.
- PATTERN-config-layers and PATTERN-sparse-hub-base: the `adopt` key reaches a hub file only through the reconcile migration.
- PATTERN-test-speed: the binary-replacing test needs a real exec and the prebuilt lym, so it is tier1, whose tier sets `prebuilt_lym`; everything else is tier0 with injected clocks.
- `internal/selfexec` refuses to exec under `go test`, so only a test that runs the built `lym` binary as the dobby process can cover a re-exec.

## Auto-mode assumptions

- The ten-second re-exec lock wait (Decision 2), the 1 MiB rotation bound (Decision 5) and the ten-minute heartbeat (Decision 6) are package constants; no config key was asked for.
- `dobby.out` is the name of the raw output file; the plan may rename it.
- A refused fresh start posts no orch notice (Decision 3).
- Landings with no machine on their Land entry are never counted (Decision 8).

## Open risks

- The investigation put a 40 percent confidence on a momentary probe being what the pre-check saw held; a SIGKILL or OOM kill could not be ruled out from the logs.
  Decisions 1 to 3 cover the probe and the second-dobby cases and leave a line for every exit the process itself takes; a kill from outside still leaves only the heartbeat's silence.
- A hub that tunes `finalize` gets `adopt` with the same tiers through the reconcile; a hub whose operator wants a different adopt scope sets it through `lym config gate` afterwards.

## Acceptance criteria

- A running dobby, started from a copy of the prebuilt lym, whose binary is replaced on disk while a goroutine loops on `dobbyengine.Live`, re-execs with its pid unchanged, holds its lock again, keeps a `State.Tidy` it held before, and writes the re-exec line and a re-exec start line to `dobby.log` (tier1).
- A re-exec'd dobby whose lock is held by another holder for longer than `lock.ProbeGrace` and shorter than the re-exec wait takes the lock and runs.
- A dobby whose lock another holder keeps past the wait exits, writes the refused line to `dobby.log`, posts the orch notice when it is a re-exec and posts none when it is a fresh start, and writes no run-process record.
- A failed exec writes its error to `dobby.log` and posts one orch notice, and the dobby keeps running.
- Each exit path of Decision 3 writes its line.
- A pass writes no line; a task start, a run's state change, the first failing pass of an episode and its recovery, a tidy spawn, report and timeout each write one.
- A heartbeat is written at start and again on the first tick ten minutes after the last, naming every run driven with its state, with an injected clock.
- `dobby.log` crossing 1 MiB rotates to `dobby.log.1`, replacing an older one; `dobby.out` past the bound at a spawn rotates the same way.
- A batten history entry carries the told machine; a loom shed told none writes no `machine`; a status file without the field decodes.
- `DobbyLandings` returns a run whose Land Done names this machine, and skips one naming another machine or none.
- A hub `gate.yaml` that tunes `finalize` and holds no `adopt` gains `adopt` with `finalize`'s value through the migration, and a second pass changes nothing; one that holds `adopt` keeps it.
- `lym gate test --gate adopt` runs the key's tiers over the whole tree.
- The check verb writes the progress record per tier, posts a line per tier and an end notice for a green and for a failing run, and a failed post leaves the record correct.
- `lym beam status` shows a running, a green, a failed and a dead-pid check record as Decision 11 says, and nothing with no record.
- `lym adopt` with an owner write spawns the check, and its envelope names the pid, the log and the record.

## Notes for the plan writer

- The pre-exec callback of Decision 3 can be a variant of `Watcher.Check`; `Replaced` already shows the shape of a non-exec'ing read.
- The dobby's `CheckBinary` seam is wired in `internal/beamcli/dobbydeps.go`; the notice for a failed exec can use the same `Notice` seam the loop already holds.
- `envkit`'s gate table and `gatecli`'s gate key lists name the keys a test hub declares.
