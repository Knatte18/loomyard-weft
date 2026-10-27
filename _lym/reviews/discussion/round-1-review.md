---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md, Decision 3 (with Decision 5 and the Acceptance criterion Each exit path of Decision 3 writes its line)
    summary: Decision 3 lists only the exits from Run onward, so a re-exec onto a new binary whose placeDobby refuses or whose hubDobbyDeps wiring fails ends with its envelope in dobby.out alone, silently in dobby.log, against the Goal.
    fix: In Decision 3, add to the list of logged ends the exits runDobby takes before Run, a hubDobbyDeps wiring failure and a placeDobby refusal, and state where each line goes. A wiring failure has its Geometry, so it writes its line to Geometry.LogPath directly, without the Deps.Log seam, and when the state record holds this process's pid it also posts the failed-re-exec notice naming the error. A placeDobby refusal has no Geometry, so state that its error envelope in dobby.out is its only trace, as the bound for that one case. Extend the Acceptance criterion so a wiring failure after a re-exec writes its line to dobby.log.
  - id: F2
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md, Decision 10 (Bound) and Decision 11
    summary: Decision 10's bound says the gate slots serialize the check, but slots is 2; a second lym adopt that records an owner spawns a second check, which truncates suite.log and rewrites check.json while the first still writes and posts. Between a spawn and the child's first write, beam status also shows the previous verdict.
    fix: "In Decision 10, state that the check verb takes a non-blocking lock beside check.json in fabricengine.AdoptScratchDir before it touches anything. The verb also opens suite.log itself, truncated, and writes a fresh running record only once it holds that lock, so the spawner no longer truncates suite.log. A check that finds the lock held exits 0 without writing suite.log, check.json or any orch post. lym adopt names the running check in the envelope's suite, and spawns none, when check.json reads running and its process is alive by Decision 11's test. Replace the bound's claim that the gate slots serialize it with this rule, and state that the verb resolves the prime from the hub whatever its cwd. Add an Acceptance criterion: a second check started while one runs leaves the first's record and log untouched."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md, Decision 9 and Decision 10
    summary: lym adopt runs only the origin reconcile and gatecli's loadGate reconciles nothing, so the first adopt check after the deploy that brings this change can run before hubreconcile writes adopt, and it runs the template's tier 0 where finalize ran tiers 0, 1 and 2, which breaks Decision 9's claim that a hub's check keeps its tiers.
    fix: In Decision 10, state that lym adopt check calls hubreconcile.EnsureOrWarn over the hub before it loads the gate key, the same call the batten and loom arms make, so the adopt migration of Decision 9 has run whenever the check reads adopt. Add to the migration criterion that a check on a hub whose build stamp is stale runs finalize's tuned tiers.
  - id: F4
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md, Decision 11
    summary: The dead-check test reads the pid alone, but check.json lives in the never-tracked adopt dir and survives a reboot, so a check ended by a reboot whose pid a later process reuses shows running in lym beam status forever.
    fix: "In Decision 10, add the process's start ticks (proc.StartTime, as runproc keys its records) to check.json. In Decision 11, a running record shows ended without a verdict when its pid is not alive or the live process's start ticks differ from the record's. Extend the beam status criterion with a record whose pid is alive but whose start ticks differ."
  - id: F5
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md, Decision 2 (Rationale and Bound)
    summary: Telling a re-exec by the record's pid alone misclassifies two cases. A fresh dobby that reuses a dead dobby's pid after a reboot waits ten seconds and posts the re-exec notice. A re-exec that reads the record after a second dobby's recordStart wrote its own pid exits on the refused-fresh path, not the refused-re-exec path the bound names.
    fix: In Decision 2, make the re-exec test the record's pid equal to this process's pid and the record's process start ticks equal to this process's own, which exec keeps; the state record gains that field, written by recordStart. In the Bound, state that a re-exec that reads another dobby's pid in the record takes the refused-fresh path and posts nothing, since one dobby runs, and add that case to the Acceptance criterion on refused starts.
  - id: F6
    severity: LOW
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md, Decision 5 and Decision 6
    summary: Moving stderr to dobby.out takes out of dobby.log the logger episode lines that reach it today, the tick-failure episode and the unreadable-owner episode, and Decision 6's list keeps only failing passes, so a dobby whose ticks fail shows nothing but heartbeats.
    fix: In Decision 6, add to the logged changes the first failing tick of an episode with its error and the tick that ends it, and the first unreadable owner read of an episode with its error and the read that recovers, each from the episode the loop already keeps. Add both to the Acceptance criterion that lists the logged events.
  - id: F7
    severity: LOW
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md, Decision 5
    summary: Decision 3's refused start writes to dobby.log from a process that holds no lock, so it and the live dobby can both see the file past 1 MiB and both rename it, the second rename replacing the first's dobby.log.1 and losing a MiB of history.
    fix: In Decision 5, state that only the lock holder rotates, and that a process holding no lock appends its line without rotating.
  - id: F8
    severity: LOW
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md, Decision 7
    summary: Decision 7 stamps machine only when the told name is non-empty, and Decision 8 never counts an entry without one, but the record does not say what batten's wiring does when machinename.Name fails, so a landing could carry no machine and never be tidied anywhere.
    fix: In Decision 7, state that batten's wiring refuses the run with the error and its way forward when machinename.Name fails, and never tells the shed an empty name.
  - id: F9
    severity: LOW
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md, Decision 1
    summary: Registering after the graced lock leaves a freshly spawned dobby unregistered through hubDobbyDeps and the lock wait, so a pair teardown in that window reads no dobby record under the beam's slug and ends nothing, and the Decision states no bound for that window.
    fix: In Decision 1, state the bound. A fresh dobby is unregistered from its spawn until it holds the lock, which is at most the wiring time plus lock.ProbeGrace. A beam dobby spawned inside a teardown of its own beam escapes that teardown and exits at its first probe tick, which finds no worktree or another owner. Alternatively, move the wiring after the lock so the window is the lock wait alone, and say which.
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md, Acceptance criteria
    summary: Two stated behaviours have no criterion. Decision 6 says a re-exec logs no change for a state it did not see change, and Decision 10 says suite.log is truncated and the record rewritten fresh at each start.
    fix: Add two Acceptance criteria. After a re-exec, the first tick logs no state-change line for a run whose state is unchanged, and the start heartbeat names it. A check started over an existing check.json and suite.log writes a fresh record and an emptied log.
  - id: F11
    severity: LOW
    class: consistency
    origin: lens:board-fit
    location: _lym/discussion/decision-record.md, Scope, the first Out bullet
    summary: The bullet says the trace sink's placement under the hub's .lym/logs is the subject of test-logs-leak-into-bolt, but that entry's record covers a fabriccli test's sink redirect and Bolt's .lym exclude, and puts the logger's semantics out of its scope.
    fix: In the first Out bullet, drop the clause naming test-logs-leak-into-bolt and keep only the reason that the dobby's own events go to dobby.log (Decision 5).
  - id: F12
    severity: NIT
    class: consistency
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md, Decision 4
    summary: Decision 4 ties keeping State.Tidy to the re-exec reading its own pid, but recordStart rewrites only the pid and start time and tidyStep settles any State.Tidy whatever the pid, so every start keeps it.
    fix: In Decision 4, replace the clause a re-exec reads its own pid in the record and keeps State.Tidy with any start keeps State.Tidy, since a start rewrites only the record's pid and start time and the tidy step settles a tidy in flight whatever pid wrote it.
  - id: F13
    severity: NIT
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md, Decision 9
    summary: The migration writes adopt, once, but it runs at every reconcile that finds a tuned finalize and no adopt, so an operator who deletes adopt to fall back to the template gets finalize's value copied back.
    fix: In Decision 9, replace once with whenever the file holds no adopt, and add to the second Open risks line that a hub returns to the template's adopt by setting it through lym config gate, never by deleting it.
  - id: F14
    severity: NIT
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md, Decision 8 (Bound)
    summary: The bound rests on the state record still listing untidied landings after ownership moves away, and the fact holds in code, but the Decision states it without the evidence it rests on.
    fix: In Decision 8's Bound, add that only a finished tidy clears LandedSinceTidy, and that no start and no owned-elsewhere exit resets it.
  - id: F15
    severity: NIT
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md, Constraints, the last bullet
    summary: The last Constraints bullet, that selfexec refuses to exec under go test so only a test running the built lym can cover a re-exec, repeats the reason the PATTERN-test-speed bullet already gives for the tier1 test.
    fix: Delete the last Constraints bullet, and in the PATTERN-test-speed bullet replace needs a real exec with needs a real exec, which internal/selfexec refuses under go test.
---

### [MEDIUM] Exits before `Run` stay silent in `dobby.log`

**Location:** `_lym/discussion/decision-record.md`, Decision 3, with Decision 5.

**Issue:** `runDobby` returns `output.Err` before `Run` when `placeDobby` refuses or `hubDobbyDeps` fails (`internal/beamcli/cli.go:373-387`).
Its wiring can fail in `ResolveWorktree`, `PauseGeometry` or `noticeProbeInterval` (`internal/beamcli/dobbydeps.go:69-83`).
Today that envelope lands in `dobby.log` through stdout (`internal/dobbyengine/start.go:270-276`).
Decision 5 moves stdout to `dobby.out`, and Decision 3's list starts at `Run`.
A re-exec onto a freshly deployed binary is the first time new wiring code runs, so this is the incident's own shape, and the Goal "never ends silently" fails there.
Four lenses (interactions, walkthrough, existing-paths, bound-breaker) and the handler found it independently.

### [MEDIUM] Two concurrent adopt checks share one record and one log

**Location:** `_lym/discussion/decision-record.md`, Decision 10 Bound, Decision 11.

**Issue:** `lym adopt` spawns a check on every call that records an owner (`internal/adoptcli/adopt.go:323-330`), and `startSuite` truncates `suite.log` with `os.Create` before the spawn (`internal/adoptcli/deps.go:212`).
The template's `slots: 2` lets two gate runs go at once, so the bound "the gate slots serialize it" does not hold.
`lym adopt root` followed by `lym adopt <beam>` leaves two checks writing one `check.json`, one truncated `suite.log`, and two streams of tier lines and end notices.
Right after a spawn, before the child writes, `lym beam status` shows the previous check's verdict.
Interactions, bound-breaker and walkthrough found it, as did the handler.
This settles the focus directive's question on a second concurrent check.

### [MEDIUM] The first adopt check can read the template's `adopt` before the migration runs

**Location:** `_lym/discussion/decision-record.md`, Decisions 9 and 10.

**Issue:** `lym adopt`'s reconcile is `fabricengine.ReconcileOrigin` (`internal/adoptcli/deps.go:65`).
`hubreconcile.Ensure` and `EnsureOrWarn` run only from `lym beam start` (`internal/beamcli/cli.go:229`), the clone, and the loom, batten and preflight arms.
`gatecli`'s `loadGate` reconciles nothing (`internal/gatecli/run.go:274`).
So a check that runs before any start verb after the deploy reads the per-name template merge, `adopt: {tiers: [0]}`.
On this hub `finalize` runs `[0, 1, 2]` (`_board/_lym/config/gate.yaml`).
That check goes green on tier 0 alone, the tiers that find a machine-move break skipped.
I confirmed the reconcile callers by grep before keeping this finding.

### [LOW] A reused pid keeps a dead check reading running

**Location:** `_lym/discussion/decision-record.md`, Decision 11.

**Issue:** `check.json` sits in the never-tracked `_board/.lym/adopt` and survives a reboot.
A reboot can end a check while its record reads running.
If a later process takes the same pid, a pid-alone test reads the check as alive for good.
`runproc` already keys its records on pid plus start ticks (`internal/runproc/record.go:70-73,99`).
This settles the focus directive's question on a stale `check.json` pid.

### [LOW] The re-exec test by pid alone misclassifies two cases

**Location:** `_lym/discussion/decision-record.md`, Decision 2.

**Issue:** `recordStart` writes the pid on a fresh start (`internal/dobbyengine/loop.go:190-203`).
A second dobby that wins the lock and records first makes the re-exec'd image read a pid other than its own.
That image takes the refused-fresh path silently instead of the refused-re-exec path the bound names, and the criterion "posts the orch notice when it is a re-exec" misses it.
After a SIGKILL and a reboot, a fresh dobby can reuse the recorded pid, wait ten seconds and post a false re-exec notice.
The hub keeps one dobby in both cases, so the bound's outcome holds but its stated path does not.

### [LOW] Tick-failure and owner-read lines leave `dobby.log`

**Location:** `_lym/discussion/decision-record.md`, Decisions 5 and 6.

**Issue:** The loop's `d.failures.Fail` tick episode and `beamOwnedElsewhere`'s `d.ownerRead.Fail` episode log through `logger` (`internal/dobbyengine/loop.go`), and reach `dobby.log` today through stderr.
After Decision 5 they go to `dobby.out`, and Decision 6's list of changes names failing passes but not failing ticks.
A dobby stuck on an unwritable state record would then show only heartbeats where the operator reads.

### [LOW] Rotation has two writers

**Location:** `_lym/discussion/decision-record.md`, Decision 5.

**Issue:** Decision 3's refused start writes `dobby.log` from a process that holds no lock (`internal/dobbyengine/loop.go:118`), through the same rotating seam the live dobby uses.
Both processes can pass the size check and rename, and the second rename replaces the first's `dobby.log.1`.

### [LOW] An unreadable machine name is unbounded

**Location:** `_lym/discussion/decision-record.md`, Decision 7.

**Issue:** `machinename.Name` returns an error (`internal/machinename/machinename.go`).
Decision 7 stamps only a non-empty name, and Decision 8 never counts an entry without one.
The record does not say whether batten's wiring refuses the run or tells an empty name.
If it tells an empty name, the landing is never tidied on any machine.

### [LOW] A fresh dobby's unregistered window widens

**Location:** `_lym/discussion/decision-record.md`, Decision 1.

**Issue:** Pair teardown ends the dobby through its run-process record under the beam's slug (`internal/pairteardown/teardown.go:596-629`).
With registration behind the graced lock and after `hubDobbyDeps`, a fresh beam dobby has no record for longer than today, and a teardown in that window ends nothing.
A re-exec keeps the old image's record (same pid and start ticks), so only fresh starts are affected.
The Decision states what a loser can reach but nothing about this window for a winner.

### [LOW] Two behaviours lack a criterion

**Location:** `_lym/discussion/decision-record.md`, Acceptance criteria.

**Issue:** Decision 6's rule that a re-exec logs no change for a state it did not see change, and Decision 10's fresh record and truncated log at each start, have no matching criterion.
Every other stated behaviour and bound the focus directive names has one.
That includes the dead-pid record and `dobby.out` rotation at a spawn.
`dobby.out`'s growth between spawns is a stated bound, not a behaviour to test.

### [LOW] The Scope Out bullet points at the wrong entry

**Location:** `_lym/discussion/decision-record.md`, Scope, Out, first bullet.

**Issue:** `test-logs-leak-into-bolt`'s record covers a tier1 test redirecting the trace sink onto the live hub and `Bolt.Commit` seeding the `.lym/` exclude.
It puts the logger's own semantics out of scope, so it does not settle where a dobby trace file belongs.
Decision 5 alone justifies the exclusion.

### [NIT] Decision 4's reason does not match the code

**Location:** `_lym/discussion/decision-record.md`, Decision 4.

**Issue:** `recordStart` rewrites only `PID` and `StartTime` (`internal/dobbyengine/loop.go:190-204`), and `tidyStep` settles any `State.Tidy` whatever the pid (`internal/dobbyengine/tidy.go:146-150`).
The keep holds for every start, including Decision 2's second dobby, not only for a re-exec.

### [NIT] The migration is not once

**Location:** `_lym/discussion/decision-record.md`, Decision 9.

**Issue:** `MigrateConfig`'s held-key rule (`internal/gateslot/config.go`) makes the copy repeat at every reconcile that finds no `adopt`, so "once" overstates the bound.
On the focus directive's question, the Decision and the second Open risks line do say what the copy lets through: a hub's tuned `finalize` tiers now govern the adopt check.
That copy is bounded to a hub that tunes `finalize` and holds no `adopt`.
The remaining gaps are this wording and the ordering in the third MEDIUM finding.

### [NIT] Decision 8's bound states its fact without evidence

**Location:** `_lym/discussion/decision-record.md`, Decision 8, Bound.

**Issue:** The fact holds: only `settleTidy` clears `LandedSinceTidy` (`internal/dobbyengine/tidy.go:203`), and `recordStart`, `tick` and the owned-elsewhere exit never touch it.
On the focus directive's question, the fact is not left for Plan, but the Decision should carry the evidence it rests on.

### [NIT] A Constraints bullet repeats another

**Location:** `_lym/discussion/decision-record.md`, Constraints.

**Issue:** The last bullet gives the reason for the tier1 test again, a reason the PATTERN-test-speed bullet two lines up already implies.
In the relocation pass, I judged the other Constraints bullets, the Rationale paragraphs and the Open risks lines.
Each Constraints bullet names the PATTERN entry a Decision answers to, which the plan writer uses.
Each Rationale paragraph carries a reason the Decision rests on, and each Open risks line carries a risk the Decisions leave standing.
None belongs only in `support-log.md`.

## Rejected

- goal-scope's MEDIUM scope finding, that Decision 8 counts landings by the landing machine where the entry's Idea says the machine that owns: the record states the departure, its rationale and its bound (Decision 8, Scope Out). The rejected alternatives in the support log show the shared-mark route was weighed. The leftover is a tidy delay for landings untidied at a handover, a recorded tradeoff, not a missed goal point.
- bound-breaker's note that a re-exec loser's run-process record still exists: the record is the old image's, keyed on the same pid and start ticks, and runproc reads a dead pid's record as stale. Decision 1's claim that the loser writes none holds.
- The prober list in Decision 1's Rationale omits `lym beam stop` and pawl's `DobbyLive`. Every one of them goes through `dobbyengine.Live`, so the claim holds, and the rubric does not flag an incomplete enumeration.
