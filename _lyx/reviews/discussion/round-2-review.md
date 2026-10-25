---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D3 and D4 (owed full pass, failing-pass retries), D7
    summary: A run another machine took over is refused by every full pass's origin reconcile, so dobby retries it forever as a failing pass and InnerRun's owner pause, lock release and owner notice never run.
    fix: In D3 and D4, state what dobby does when a full pass is refused as owned elsewhere. Give the resume's owned-elsewhere refusal (resumeRefusal's EntryOwnedElsewhere arm) an exit code of its own, as the --continue-pass refusal has, and have dobby read it as owned elsewhere. Dobby then stops passing that run, clears its owed full pass, and runs the existing owned-elsewhere handling once, through tendercli's PauseOwnedElsewhere or the same pause and parent-lock release InnerRun's ownedElsewhere makes, sending the owner notice naming lyx adopt instead of the failing-pass notice. It drives the run again only when a later full pass, after an adopt back or a lift of that owner pause, exits 0. Add the matching acceptance criterion and drop D3's claim that InnerRun's owner check covers a takeover found by a full pass.
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D4 (spawn hold returns Yield)
    summary: With ownedBy persisted, the spawn hold now yields before the once-per-probe owner re-check that alone ends an owner hold, so a run read owned elsewhere stays held on every pass, even after it is adopted back.
    fix: In D4's spawn-hold paragraph, state that before yielding on a standing owner the hold runs the owner check when a notice probe is due (the persisted lastProbe), and that an owned-here verdict clears ownedBy and goes on to the spawn in the same pass. Add an acceptance criterion that a run held on a standing owner and then adopted back is respawned by the next probe pass.
  - id: F3
    severity: BLOCKING
    class: design
    origin: handler
    location: _lyx/discussion/decision-record.md D4 (continue pass) and PATTERN changes
    summary: "A continue pass resumes an existing run record while skipping the run-scope ReconcileOrigin, but PATTERN-owner-machine gates every resume on that origin verdict, and the record declares no change to that entry."
    fix: "Either add a PATTERN-owner-machine change to PATTERN changes, its text saying that a continue pass dobby schedules into a run it already drives is not a resume and needs no origin verdict, while the first pass after a dobby start or a wake and every hand pass is one; or drop the reconcile skip from --continue-pass. Either way, D4's continue-pass paragraph names the PATTERN entry."
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D4 (one check of its child per Call; memory-record inventory)
    summary: D4 does not say which round a pass's one check is. The persisted owner triggers, lockStep's idle release and refreshWatched run only in check(), never in wait()'s first round, and the owner trigger's process-local ownerSleep is in neither the inventory nor the replacement.
    fix: In D4, state that a pass's one check runs check()'s full body, coverage, foldOwnerTrigger, checkChild, ownerStep, refreshWatched and lockStep on a due probe, noticeStep, the arm's step and the pause-flag read, with probe-due judged from the persisted lastProbe. Add to the inventory's last bullet that the owner trigger's sleep watch (ownerSleep) is also replaced, by dobby's recorded sleeps and the owed full pass.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D4 (respawn cap) and D9 (api-error)
    summary: The new cross-pass counters, the respawn count with the child status write it resets on, the per-episode api-error revive-sent mark and the count of revived episodes in a row, are given no home, so each pass starts them fresh and the caps never trip.
    fix: Add a bullet to D4's memory-record inventory naming these fields, the respawn count and the child status file's modification time at the last respawn, the api-error episode whose revive line was sent, and the count of revived api-error episodes in a row with the status modification time it last saw, each with the rule it keeps.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D4 (owed full pass cleared only when a full pass exits 0)
    summary: A full pass with origin unreachable resumes on the local Bolt and exits 0, which clears the owed full pass after a wake with no origin verdict, so a takeover while the network was down is never judged afterwards.
    fix: In D4, state that the full pass reports an unreachable-origin resume (refuseResumeUnreachable's path) to dobby, by an exit code or a line in its state output, and that dobby keeps the full pass owed until a full pass reaches origin.
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D2 (lyx adopt ends with lyx beam start) and the matching acceptance criterion
    summary: lyx adopt of one member task of a beam another machine owns moves the task here while its parent beam stays owned there. lyx beam start then starts no dobby for that beam here, and the other machine's dobby drives only local runs, so the adopted run has no dobby.
    fix: "In D2 or D3, decide who drives an adopted member task whose beam another machine owns: either lyx adopt of such a member refuses, naming lyx adopt of the beam, or the dobby of this machine's nearest owned ancestor drives runs whose parent beam is owned elsewhere. Make the acceptance criterion name that case."
  - id: F8
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D5 (creating writes; no code path but an explicit release)
    summary: A note upserted to kind task, which the board's task guard counts as making a task, keeps its empty status, so dobby starts it unreleased. An upsert with status null and a merge's nil set_status also empty a status without the release D5 names.
    fix: In D5's list of writes that give draft, add an upsert or batch upsert that turns an existing note into a task. State that an upsert or merge that explicitly sets an empty status is a release, like lyx board set-status with no status.
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D4 (Bound - a pass stays in flight only through those waits)
    summary: D4's list of in-pass waits omits Land's deploy run (up to 30 minutes) and InnerRun's synchronous lyx loom start bootstrap on a spawn, so the bound is stated false.
    fix: Add both waits to D4's paragraph of in-pass waits, each with its bound, the deploy timeout and the loom start bootstrap.
  - id: F10
    severity: LOW
    class: design
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D10 (a restart neither misses nor double counts one)
    summary: lyx batten cleanup --apply removes a done run's records directory, history included, so a landing made while dobby is down and cleaned up before it restarts is never counted.
    fix: "In D10, either state the miss as an accepted bound (a landing whose run records cleanup removed before dobby read them is not counted), or say how the count also reads landings from the board's done entries since the last tidy."
---

## Focus verdicts

- Wake and ownership: the first pass after a wake does run the origin reconcile (D4's owed full pass), but its refusal of a run another machine took over is mishandled (F1), an unreachable origin clears the owed pass with no verdict (F6), and the continue pass's skip of the reconcile departs from `PATTERN-owner-machine` with no declared change (F3). D3's claim that InnerRun's owner check covers a takeover is false under the chosen mechanism, because no pass reaches InnerRun once the reconcile refuses.
- Pass state across processes: the inventory covers today's producer and `childWait` fields, but not which round of checks a pass runs or the owner trigger's sleep watch (F4), nor the new counters D4 and D9 add (F5). The Constraint about surviving restarts is accurate as worded once those are persisted.
- Removal and cancel: `settleClaim`'s `claimClear` arm now writes `draft`. An owned-elsewhere or ended `lyx fabric remove` is `Local` and writes no board status (`internal/pairteardown/teardown.go:369`). Cancel's `LeaveClaim` removal leaves the claim to cancel. The only paths that leave a task statusless are in F8. `lyx reed down` sparing `RoleDobby` is bounded in D1: the prime returns early (`internal/reedcli/up.go:87`), and teardown and cancel still end the dobby. That part is not a defect.
- Escape hatches and pass bounds: `--continue-pass` is bounded by the status and the state-record check, and a caller from outside that meets them does only what dobby would do. That part is not a defect, apart from its PATTERN fit (F3). A pass does stay in flight through the Go driver's pause hold and Land's waits, as D12 now states, but D4's list of waits is incomplete (F9).

### [BLOCKING] A takeover found by a full pass loops as a failing pass

**Location:** `_lyx/discussion/decision-record.md` D3, D4, D7
**Issue:**
A full pass runs the resume's origin reconcile, and `resumeRefusal` returns `ownedElsewhereRefusal` for an owned-elsewhere entry before the driver steps anything (`internal/battencli/resumereconcile.go:78-81`, `internal/battencli/owner.go:192-211`).
The pass exits non-zero with a plain error.
D4 keeps an owed full pass until a full pass exits 0 and retries a failing pass on every probe tick, with "the retries go on".
So dobby re-runs the refused pass every `notice_probe_s` forever, and D13 counts the run as in flight, so the timer never backs off.
D7 posts a failing-pass notice instead of the owner notice.
InnerRun's `ownedElsewhere` (`internal/battenshed/innerrun_owner.go:101-115`) never runs: no `PauseOwned`, no parent-lock release, no notice naming `lyx adopt`.
`lyx orch start`'s `pauseOwnedRuns` (`internal/orchcli/start.go:283-300`) pauses such runs only when an orch starts, and D4 still passes a covered `running` run.
D3's "drives only runs this machine owns" has no mechanism behind it.
All four lenses raised this.

### [BLOCKING] The spawn hold yields before the owner re-check that ends it

**Location:** `_lyx/discussion/decision-record.md` D4, the spawn-hold paragraph
**Issue:**
`holdSpawnWhileCovered` loops while `ownedBy` is set, and only its in-loop `ownerStep`, once per notice probe after a `Sleep`, can clear `ownedBy` (`internal/battenshed/innerrun_pause.go:47-71`).
D4 makes the hold return `Yield` instead of sleeping, and persists `ownedBy`.
Every later pass that must spawn (`internal/battenshed/innerrun.go:400-417`) enters the hold and yields at once, never reaching that re-check or `check()`'s owner trigger.
A run read as owned elsewhere and then adopted back, or whose owner pause lifted, stays held on every pass with no way forward.
That breaks the persisted triggers' stated purpose.

### [BLOCKING] Continue passes skip the resume's origin verdict without a declared PATTERN change

**Location:** `_lyx/discussion/decision-record.md` D4, the continue-pass paragraph, and `## PATTERN changes`
**Issue:**
Batten resumes a found status file only after `ownerPreflight`, the run-scope `ReconcileOrigin` (`internal/battencli/arm.go:639`, `internal/battencli/owner.go:188-211`).
`PATTERN-owner-machine` says "the owner verdict that gates a resume, a wake or a landing" is "judged against origin inside `ReconcileOrigin`".
A continue pass is a new process resuming that record, and `--continue-pass` skips the reconcile.
The record argues the skip from dobby's continuity, but neither cites the entry nor declares a change to it.
A departure from a listed constraint is blocking.

### [MEDIUM] A pass's one check is unnamed, and the owner fold and lock step may never run

**Location:** `_lyx/discussion/decision-record.md` D4, "one check of its child per Call" and the memory-record inventory
**Issue:**
`wait()`'s first round runs only `coverage`, `noticeStep` and `runArm` (`internal/battenshed/innerrun.go:669-678`).
`foldOwnerTrigger`, `ownerStep`, `refreshWatched` and `lockStep` run only in `check()`, after the first `Sleep` (`innerrun.go:700-714`).
If a pass's one check is the first round, the persisted owner triggers never fire, D12's pause-lift trigger is lost, and an idle owner's parent lock is never released, against D4's own inventory and its acceptance criterion.
`foldOwnerTrigger` also reads `ownerSleep` against the process-local awake clock (`innerrun.go:264-266`), and the inventory neither persists nor replaces that watch.

### [MEDIUM] The new counters of the respawn cap and the api-error rule have no home across passes

**Location:** `_lyx/discussion/decision-record.md` D4 (respawn cap), D9 (api-error)
**Issue:**
D4's three-respawn cap needs a count and the child status write it resets on.
D9 needs the revive line sent once per episode and a count of revived episodes in a row.
None of these exists today, and the inventory lists only today's fields, so each pass, a new process, starts them fresh and neither cap trips.

### [MEDIUM] An unreachable origin clears an owed full pass with no verdict

**Location:** `_lyx/discussion/decision-record.md` D4, the owed full pass
**Issue:**
With origin unreachable, `reconcileResume` Warns and resumes on the local Bolt through `refuseResumeUnreachable` (`internal/battencli/resumereconcile.go:52-55`), so the full pass exits 0.
D4 clears the owed full pass on that exit, so a wake while the network is down never judges a takeover against origin.
InnerRun's retry trigger cannot catch it either, because it needs `ownedBy` already set.

### [MEDIUM] An adopted member task of a beam owned elsewhere has no dobby

**Location:** `_lyx/discussion/decision-record.md` D2, and the acceptance criterion on `lyx adopt`
**Issue:**
`lyx adopt` takes single task slugs (`internal/adoptcli/adopt.go:214-229`).
Adopting one member task of a beam another machine owns moves the task to this machine, while its parent beam stays owned there.
D2's `lyx beam start` starts no dobby for a beam this machine does not own.
D3's dobby drives only runs on its own machine.
So no dobby drives the adopted run, against D2's "a dobby runs wherever the adopted runs' parent now is".

### [MEDIUM] A note upserted to kind task becomes a statusless task, and other writes clear a status

**Location:** `_lyx/discussion/decision-record.md` D5
**Issue:**
`UpsertTask` patches an existing note through `ApplyPatch` and keeps its status (`internal/boardengine/store.go:514-515`).
The board's own guard counts "a note upserted to kind task" as making a task (`internal/boardengine/taskguard.go:19`), yet D5's draft list names only creating writes and `promote`.
Dobby would start such a task unreleased.
The upsert payload's `status` key and a merge's nil `set_status` (`store.go:436-448`, `:1074-1077`) also empty a status outside the release D5 names, so "no code path but an explicit release leaves a task statusless" is false as worded.

### [LOW] D4's list of in-pass waits is incomplete

**Location:** `_lyx/discussion/decision-record.md` D4, "Bound: a pass stays in flight only through those waits"
**Issue:**
Land's deploy runs for up to `deployTimeout`, 30 minutes (`internal/battenshed/land.go:33`, `:334`).
InnerRun's spawn runs `lyx loom start --no-attach` synchronously (`internal/battencli/wire.go:1205-1213`).
Both keep a pass in flight, and neither is in the list.

### [LOW] Cleanup can remove a landing before dobby counts it

**Location:** `_lyx/discussion/decision-record.md` D10
**Issue:**
`lyx batten cleanup --apply` removes a done run's records directory, history included (`internal/battencli/cleanup.go:217`, `:276-277`).
A landing made while dobby is down and cleaned up before it restarts is never counted, against D10's "a restart neither misses nor double counts one".

## Rejected

- bound-breaker, D12, a pause written between InnerRun's coverage read and the driver's step into Land: Land is the next step of the same pass, so the window equals a pause landing a moment after Land starts, which the record never claims to stop. Not a defect.
- bound-breaker, respawn-cap reset on a status write by `lyx loom start` (unverified by the lens): a respawned driver that writes the child status file has made progress, which is what D4's reset rule counts. Not a defect on the evidence.
- existing-paths, `ownerSleep` reads as surviving: superseded by F4, which records the gap in the inventory's wording.
