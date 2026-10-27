---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md, Decision 3 second bullet of the exits before Run, Open risks first bullet, Acceptance criteria
    summary: The premise that a placeDobby refusal has no Geometry and no log path is false for every refusal after lymcwd.ResolveWorktree succeeds, so a re-exec'd image refused there ends with no dobby.log line and no post, and no Acceptance criterion covers any placeDobby refusal trace
    fix: In Decision 3, split placeDobby's refusals in two. A refusal after lymcwd.ResolveWorktree succeeded (RequireDrivableWorktree, PrimeName, ReadOrigin, neither prime nor beam, the prime resolve, DobbyGeometry) still holds the resolved location, so runDobby derives the worktree's own .lym/beam directory from it through the same path rule hubgeom.DobbyGeometry uses, appends the refusal line with its error to that dobby.log without rotating, runs Decision 2's re-exec test on that directory's state.json, and on a re-exec posts the failed-re-exec notice through orchcli.NotifyFromScoped over that location naming the refusal's error and its way forward. Only a filepath.Abs or ResolveWorktree failure keeps dobby.out as its only trace; state that narrower bound in Decision 3 and in the Open risks bullet. Add an Acceptance criterion that a re-exec'd dobby refused by placeDobby after its worktree resolves writes its line to dobby.log and posts one notice, and that a refusal with no resolved worktree leaves its envelope in dobby.out and nothing in dobby.log.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md, Decisions 2 and 4
    summary: The pid-plus-start-ticks re-exec test governs only the lock wait, while recordStart still decides start or re-exec on the pid alone, the tick path refills a pid-less record with no ticks, and a ticks-less record is never back-filled, so a pid-reusing fresh dobby is recorded as a re-exec and its later real re-execs read as fresh starts with only lock.ProbeGrace
    fix: In Decision 2, state that the one re-exec test (this pid and this process's start ticks, or this pid and no ticks) is the test recordStart uses too, so a start that fails it owes a full pass and rewrites pid, start time and start ticks; that every writer of State.PID, including tick's refill of a record found without a pid (internal/dobbyengine/loop.go tick), writes the start ticks beside it; and that a re-exec onto a record holding this pid and no start ticks writes this process's start ticks into it, leaving every other field as Decision 4 says. Add Acceptance criteria that a fresh dobby whose pid matches the record's but whose ticks differ rewrites the record's start ticks and owes a full pass, and that a re-exec onto a ticks-less record leaves the record holding its ticks.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md, Decision 5 and the Goal
    summary: A dobby spawned by a binary from before this change holds dobby.log as its fd 1 and 2 and keeps them across every re-exec, so raw output keeps reaching dobby.log and then its renamed inode, and dobby.out never receives it, since a dobby that lives by re-exec is never respawned
    fix: In Decision 5, add that runDobby, once placeDobby returns its Geometry and before anything else writes, reopens its own stdout and stderr onto Geometry's dobby.out for append, on the platforms where selfexec execs, so a dobby whose fds were inherited from an older spawn moves them off dobby.log at its first start on this binary; or, if the plan writer is to keep the fds, state the bound that such a dobby's raw output reaches dobby.log until its next spawn. Add an Acceptance criterion that a dobby started with its stdout and stderr on dobby.log writes its later envelope and stderr lines to dobby.out after a start.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md, Decision 1 worktree-gone exit, Decision 3 line list, Acceptance criterion on a removed beam worktree
    summary: A beam dobby's dobby.log lies inside the beam worktree that was removed, so the worktree-gone exit line Decision 1 and its Acceptance criterion require has nowhere to land, and a per-line append either fails or recreates part of the removed tree
    fix: In Decision 1, name where the worktree-gone exit line goes, the prime's dobby.log naming the beam's slug, appended without rotating, since this process does not hold the prime's lock, and state that the Deps.Log seam never creates a missing log directory. Change the Acceptance criterion to assert the exit line in the prime's dobby.log and that no directory is recreated at the removed worktree's path.
  - id: F5
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md, Decision 3 post list
    summary: A fatal error out of Run right after a re-exec, from its state read, lock acquisition or recordStart, gets a dobby.log line but no orch post, though it is a failed re-exec the entry's Expected says is posted, and Decision 3 posts the parallel wiring failure after a re-exec
    fix: In Decision 3, add a fatal error out of Run, when the start passed Decision 2's re-exec test, to the cases that post one notice to the nearest live orch, naming the error and lym beam start once its cause is fixed, through the Deps.Notice seam Run already holds; a fatal error on a fresh start keeps its line only. Extend the Acceptance criterion on Decision 3's exit paths with a fatal Run error after a re-exec that writes its line and posts the notice.
  - id: F6
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md, Decision 10 and the Acceptance criteria on a second check and on a fresh record and an emptied log
    summary: The record moves suite.log's opening into the check but never says where the detached check process's own stdout and stderr go, so today's wiring onto suite.log appends a losing check's envelope and logger lines to the running check's log, while a null device would lose a check's panic text
    fix: In Decision 10, state that lym adopt spawns the check with its own stdout and stderr on a file of their own beside check.json, such as check.out, opened for append and never suite.log, so the gate's output alone reaches suite.log, a losing check writes its envelope there and leaves suite.log untouched, and a panic of the check is on disk; name the file in the PATTERN-durable-vs-ephemeral-state constraint and in Decision 11's ended-without-a-verdict line beside suite.log.
  - id: F7
    severity: LOW
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md, Decision 9 and Decision 10, the way forward of a check that fails to spawn
    summary: When the check fails to spawn, no hubreconcile.Ensure ran and gatecli's loadGate reconciles nothing, so the named rerun lym gate test --gate adopt reads the template's adopt, tier 0, on a hub whose build stamp is stale and goes green on less than finalize's tuned tiers
    fix: In Decision 9 or 10, state that lym adopt's warning for a check that did not start names lym fabric reconcile from the prime before lym gate test --gate adopt, so the migration has written adopt before the hand run reads it.
  - id: F8
    severity: LOW
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md, Decision 8
    summary: A state record from before this change already holds other machines' landings in LandedSinceTidy, which only settleTidy clears, so the first tidy after the deploy on a machine like Neve still tidies the foreign landings part 3 reports
    fix: In Decision 8, add that the landing count drops from LandedSinceTidy every slug the machine-filtered Landings no longer returns, before deciding a tidy is due, and add an Acceptance criterion that a record holding another machine's landing in LandedSinceTidy spawns no tidy for it.
  - id: F9
    severity: LOW
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md, Decision 1 worktree-gone exit and Decision 4
    summary: The new worktree-gone exit can end a dobby with State.Tidy in flight, and nothing stops the tidy agent or settles the record, the open-strand symptom of the incident
    fix: In Decision 1, state that the worktree-gone exit stops a tidy agent in flight through TidyShuttle.StopStrand before it returns, logging that stop, or state the bound that its strand stays open until removed by hand.
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md, Decision 5, rotation of dobby.out at a spawn
    summary: SpawnDetached rotates dobby.out holding no lock, contradicting Decision 5's own rule that only the lock holder rotates, so two concurrent spawners can lose a .1 and a spawn renames the file a running dobby holds as fd 1 and 2
    fix: In Decision 5, state that the spawn rotates dobby.out only when Live reads the dobby lock free, and accept the remaining race of two concurrent spawners as a stated bound, or move the rotation into the process that took the lock, before F3's reopen of its fds onto dobby.out.
  - id: F11
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md, Decision 2 bound
    summary: In the interleave where the re-exec'd process reads the record before the second dobby's recordStart, it exits through the refused-re-exec path and posts a notice naming lym beam start while a dobby runs, a false alarm the bound presents as harmless
    fix: In Decision 2, state that a refused re-exec reads the state record again when its wait ends and posts no notice when the record then holds another pid, taking the refused-fresh path, and add that case to the refused-re-exec Acceptance criterion.
  - id: F12
    severity: NIT
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md, Decision 10 bound
    summary: The bound says the second lym adopt's envelope names a check that never ran, but either spawned check can win the lock, so either envelope can
    fix: In Decision 10's bound, write that either envelope's pid can name the check that lost the lock, while both name the same log and record.
  - id: F13
    severity: NIT
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md, Decision 6 rationale
    summary: The rationale says a dead dobby shows within ten minutes, but the heartbeat is written by the first tick ten or more minutes after the last, with ticks up to a minute apart, so the gap reaches about eleven minutes
    fix: In Decision 6's rationale, write that a dead dobby shows in its log within eleven minutes, the interval plus the loop's one-minute idle ceiling.
---

### [MEDIUM] placeDobby refusal premise is false for most refusals, leaving a re-exec'd image's refusal silent

**Location:** `_lym/discussion/decision-record.md`, Decision 3, second bullet of the exits before `Run`; Open risks; Acceptance criteria.

**Issue:** Decision 3 bounds every `placeDobby` refusal to an envelope in `dobby.out`, on the premise that it "has no Geometry and no log path".
In `internal/beamcli/cli.go:407-433`, every refusal after `lymcwd.ResolveWorktree` returns holds a resolved location.
The worktree's `.lym/beam` directory, with `dobby.log` and `state.json`, comes from that location by the rule `internal/hubgeom/dobbygeom.go:47` applies, and the location is the worktree `orchcli.NotifyFromScoped` takes.
A deploy whose new image refuses the prime at `RequireDrivableWorktree` or `PrimeName` after a re-exec therefore ends the dobby with no `dobby.log` line and no post.
That is the shape of #880 that the entry's Expected section forbids.
For focus 1: the wiring failure's trace is stated and covered by an Acceptance criterion; the `placeDobby` refusal's is not, and no criterion asserts any trace for it.
Also raised by lens bound-breaker (F1) and lens goal-scope (F1).

### [MEDIUM] the start-ticks re-exec test does not reach recordStart or every pid writer

**Location:** `_lym/discussion/decision-record.md`, Decisions 2 and 4.

**Issue:** `recordStart` (`internal/dobbyengine/loop.go:190-203`) returns early on `record.PID == os.Getpid()`.
`tick` (`loop.go:275-278`) refills a record found without a pid using pid and start time only.
Decision 2 adds the ticks to the lock wait alone.
A fresh dobby that reuses a dead dobby's pid after a reboot gets the short wait, but `recordStart` then records it as a re-exec: no full pass and no new ticks.
Its first real re-exec then fails the ticks test and waits only `lock.ProbeGrace`, which brings back the incident's failure mode.
A ticks-less legacy record or one refilled by `tick` is never back-filled, so the pid-reuse guard never arrives for a dobby that only re-execs.
Lenses existing-paths (F1), interactions (F4) and bound-breaker (F2) raised parts of this; it is merged here.

### [MEDIUM] inherited fds keep raw output on dobby.log after the first re-exec onto this change

**Location:** `_lym/discussion/decision-record.md`, Decision 5 and the Goal.

**Issue:** Today `SpawnDetached` points the dobby's fd 1 and 2 at `dobby.log`, and `syscall.Exec` keeps them.
Under Decision 1 such a dobby is never respawned, so its envelope, stderr `logger` lines and a panic keep reaching `dobby.log`.
After a rotation they reach the renamed inode in `dobby.log.1`, and after the next one an unlinked file.
"No raw process output reaches the file" and the stated bound both fail on every hub that runs a dobby when this ships.
Also raised by lens walkthrough (F3) and lens bound-breaker (F3).

### [MEDIUM] the worktree-gone exit line has nowhere to land

**Location:** `_lym/discussion/decision-record.md`, Decision 1, Decision 3's line list and the removed-worktree Acceptance criterion.

**Issue:** `Geometry.LogPath` lies under `<HubPath>/<worktree>/<anchorRel>/.lym/beam` (`internal/hubgeom/dobbygeom.go:26,47`).
Once the beam worktree is removed, the per-line append of Decision 5 fails, or recreates part of the removed tree if it creates directories.
The record names no other destination, so the Acceptance criterion asserts a write the geometry cannot take.
Also raised by lens walkthrough (F1).

### [MEDIUM] a fatal Run error after a re-exec is not posted

**Location:** `_lym/discussion/decision-record.md`, Decision 3.

**Issue:** `Run`'s only error returns come before the loop: the lock directory, the lock acquisition and `recordStart` (`internal/dobbyengine/loop.go` `Run`); the loop itself returns only nil.
Each one can follow a re-exec onto a new binary.
Decision 3 posts a wiring failure after a re-exec but not these, so a re-exec that dies in `Run` is logged and never posted, against the entry's Expected section.
Also raised by lens goal-scope (F1).

### [MEDIUM] the adopt check process's own stdout and stderr are unstated

**Location:** `_lym/discussion/decision-record.md`, Decision 10 and the two Acceptance criteria on `suite.log`.

**Issue:** Today `startSuite` sets `command.Stdout` and `command.Stderr` to `suite.log` (`internal/adoptcli/deps.go:220-223`).
Decision 10 moves the opening of `suite.log` into the verb, but it is silent on the child's own fds.
Kept as today, a losing check's envelope and `logger` lines append to the running check's log, against "leaves the first's record and log untouched".
Sent to a null device, a check's panic leaves no text, and Decision 11 shows only "ended without a verdict".
For focus 2: the lock keeps `check.json` and `suite.log` consistent between two checks, and the record states what `lym beam status` shows before the child's first write; this fd gap is what remains.
Also raised by lens interactions (F3).

### [LOW] the spawn-failure way forward reads an unmigrated adopt key

**Location:** `_lym/discussion/decision-record.md`, Decisions 9 and 10.

**Issue:** `loadGate` (`internal/gatecli/run.go:274-301`) runs no reconcile.
When the check fails to spawn, no `hubreconcile.Ensure` has run.
On a stale stamp, `lym gate test --gate adopt` therefore reads the template's `{tiers: [0], scope: all}` (`internal/gateslot/template.yaml`) instead of the hub's tuned `finalize` `[0, 1, 2]` (`_board/_lym/config/gate.yaml:46`).
For focus 3: the check path is stated (Decision 10's `Ensure` before the load) and covered by the stale-stamp Acceptance criterion; only this hand path is open.
Raised by the handler too.

### [LOW] foreign landings already queued in LandedSinceTidy are tidied once more

**Location:** `_lym/discussion/decision-record.md`, Decision 8.

**Issue:** `countLandings` (`internal/dobbyengine/tidy.go:101-131`) appends to `LandedSinceTidy`, and only `settleTidy` clears it.
A record that counted another machine's landings before this change keeps them, so the first tidy after the deploy covers them again.
Also raised by lens bound-breaker (F4).

### [LOW] the worktree-gone exit leaves a tidy agent in flight

**Location:** `_lym/discussion/decision-record.md`, Decision 1 and Decision 4.

**Issue:** Decision 4 covers `State.Tidy` only across a re-exec.
The new exit ends the dobby for good, and nothing stops the tidy agent's strand: the open strand the incident's Seen section reports.

### [LOW] the dobby.out rotation contradicts Decision 5's own lock rule

**Location:** `_lym/discussion/decision-record.md`, Decision 5.

**Issue:** Decision 5 lets only the lock holder rotate "so two renames never race", then has the lockless spawner rotate `dobby.out`.
Two spawners, such as `lym beam start` and an `EnsureLive` spawn in Decision 2's exec gap, can lose a `.1`.
A spawn while a dobby runs renames the file that dobby holds as its fds.

### [LOW] a refused re-exec can post a false alarm while a dobby runs

**Location:** `_lym/discussion/decision-record.md`, Decision 2 bound.

**Issue:** In the first interleave of the bound, the second dobby is live and holds the lock.
The re-exec'd process still posts a refused-re-exec notice naming `lym beam start`.

### [NIT] either adopt envelope can name the check that never ran

**Location:** `_lym/discussion/decision-record.md`, Decision 10 bound.

**Issue:** Lock order between the two spawned checks is not spawn order, so the first `lym adopt`'s envelope can name the losing pid.

### [NIT] the heartbeat bound is eleven minutes, not ten

**Location:** `_lym/discussion/decision-record.md`, Decision 6 rationale.

**Issue:** The heartbeat is written on the first tick at or after ten minutes, and ticks can be one minute apart (`idleCeiling`, `internal/dobbyengine/loop.go:23`).

## Rejected

- lens goal-scope F2 (the zero-tidy case narrows part 3): the record already names it under Scope Out and in Decision 8's bound, and the entry's stated aim, that an adopt never tidies a landing again, is met; restating it as a departure adds content with no requirement behind it.

No constraint violation was found: the PATTERN entries the record names (leaf packages, cwd resolution, durable versus ephemeral state, tolerant records, spawn observability, pane-text injection, gate slot, config layers and sparse base, test speed) are respected by the decisions as written.
