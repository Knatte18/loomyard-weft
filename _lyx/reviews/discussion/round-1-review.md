---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D1
    summary: D1 reads coverage before sendVerified's idle wait, so a pause that lands while the send waits for idle still gets the line typed into a covered session, which breaks the declared PATTERN-pause-holds-prompts
    fix: In D1, state that the coverage read sits after awaitIdleSession returns and immediately before typeVerified (a read before the idle wait is allowed as an early exit but not as the only check), and add an acceptance criterion that a pause added while a send waits for an idle session types nothing and returns ErrPaused.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D6
    summary: D6 says the orch watcher records the orch with GUID the strand name, but the watcher writes the orch strand's reed GUID (State.Strand), so a claim in D6's stated shape is invisible to settledWithoutInterrupt, Owner and ownsUnnoticedOrch, and the watcher sends its own Esc and never queues the orch's resume line
    fix: In D6, replace (GUID the strand name, Orch true) with (GUID the orch strand's reed GUID, the value orchengine State.Strand holds, Orch true), and add an acceptance criterion that an orch claimed by D7 under a hub entry gets no second Esc from the watcher and gets its resume line after the resume.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D1
    summary: Telling the orch runner the hub geometry through SetPause also arms Runner.Start's waitWhileCovered, so lyx orch start now blocks with no deadline while a hub entry is live, and D1 says nothing about this
    fix: In D1, either state that lyx orch start waits for the resume under a hub entry, and why that is acceptable, or say that the orch runner gets the record path for the D1 send check only and that its run start keeps starting under a hub pause as it does today. Add the chosen behaviour as an acceptance criterion.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D4
    summary: D4 reads the hold only while the command runs, so a Verify entered under a covering pause still writes its marker and starts its command (taking or queueing for a gate slot), and is only stopped once the grace end has passed, which misses the Goal of holding everything lyx would start
    fix: In D4, add that before starting its command, after the dirty and skip checks, Verify waits through the second hold function while a pause covers the scope, writing the paused marker during that wait. Add an acceptance criterion that a Verify entered under a covering pause starts no command until the resume.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D4
    summary: SIGKILL to the group after its wait delay does not follow from verifyrun today, because exec's WaitDelay kills only the shell and fires only while Wait has not returned, and with a file log Wait returns as soon as sh dies on SIGTERM, so a lyx gate test still alive is never escalated
    fix: In D4, state that the stop sends SIGTERM to the process group and then, after the wait delay, sends SIGKILL to the same group whether or not the shell has already exited, through a timer of verifyrun's own and not through exec.Cmd.WaitDelay. Add to the stop acceptance criterion that a group member which ignores SIGTERM is killed after the delay.
  - id: F6
    severity: MEDIUM
    class: consistency
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D4 and Scope
    summary: D4 changes verifyrun.Run's cancel, which also ends batten's deploy and Command rows and the card Verify rerun on their timeouts, while Scope Out calls those no verify site and the record never says they now get SIGTERM first
    fix: In D4, state that the SIGTERM-then-SIGKILL cancel applies to every verifyrun.Run caller, naming batten's deploy and Command rows and cardverify's timeouts as also getting it, or else that only Verify's pause stop uses SIGTERM first while timeout cancels keep the immediate group SIGKILL. Bring the Scope Out bullet into line with whichever is chosen.
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D8
    summary: When ResolveStrandGUID fails, D8 refuses and records nothing, yet the refusal still tells Master to rerun after the resume line arrives, and tender resume sends lines only to recorded sessions, so Master ends its turn and stays idle with no way forward
    fix: In D8, give the failed-lookup branch its own refusal text that tells Master the resume line will not come and to run the same verb again once lyx tender status shows no pause, and state that bound. Add an acceptance criterion that reaches this branch.
  - id: F8
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D1
    summary: D1 states no outcome for a failed reed lookup of the target's name, and its fallback for a name that does not parse uses the runner's geometry slug, which for runners told the hub geometry lets a slug entry's sessions through, so neither fail-open says what it lets through
    fix: In D1, state what a failed name lookup does (read uncovered with one Warn per change of error, or refuse), and add a Bound sentence saying that both the lookup failure and the unparsable-name fallback can let a line into a session that only a slug entry covers when the runner's slug is empty.
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D1 and PATTERN changes
    summary: Runner.SendConditioned, which tender revive uses, types through typeVerified and never passes sendVerified, so D1's premise that every shuttle line reaches a pane through sendVerified is false; behaviour holds only because revive reads coverage itself
    fix: In D1, replace every shuttle line with every shuttle line but SendConditioned's, and add one sentence saying that SendConditioned's only caller, tender revive, reads coverage itself under the record lock and sends nothing to a covered session.
  - id: F10
    severity: LOW
    class: design
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D7
    summary: D7's re-read runs under creel's pane lock, which tender resume never takes, so a resume can land between the re-read and the key, and if its send fails busy the strand is interrupted with no resume line
    fix: In D7, state that window as an accepted residual of the check-then-key order, the same one tender's pause has, and add it to Open risks. Do not claim that the pane lock excludes a resume.
  - id: F11
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D3 and Open risks
    summary: An Expired probe that fires under cover calls finalize, which re-runs the gate closures, verify included, under the pause, against D3's rationale; Open risks names only a liveness or deadline finalize, and the deadline cannot fire while covered
    fix: In D3, state that an Expired probe firing under cover finalizes the run and that the final evaluation then sits in D4's held Verify. Change the Open risk to name an Expired or liveness finalize, and drop the deadline.
  - id: F12
    severity: LOW
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D4
    summary: A paused Verify keeps a live marker, and loomcli agentActivityFor and battencli's activity reading treat any live marker as activity now, so a held verify resets the loop's step idle timeout and suppresses batten's quiet notice for the whole pause, and D4 names only loom status as a reader
    fix: In D4, state how the paused marker reads to the activity readers in loomcli looparm.go and battencli wire.go, either as activity now (and why that is wanted under a pause) or as no activity.
  - id: F13
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md Notes for the plan writer
    summary: The note that cancelling ends a held Verify has no path at the loomshed and webster verify gates, which pass context.Background(), so there a held Verify ends only on resume or on the death of its process
    fix: In the plan-writer note, say that at the loomshed and webster verify gates a held Verify ends only on the resume or on the end of its process, because those sites pass a background context.
  - id: F14
    severity: LOW
    class: consistency
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D2
    summary: D2's start bullet lists a resume prompt as part of run start, but the deferred-prompt send (DeliverPendingPrompt) is reached from burler's ResumeFix after the start's own wait, so the bullet's framing does not describe the caller that receives the refusal
    fix: In D2, give DeliverPendingPrompt its own bullet with its outcome (burler's ResumeFix waits while covered and then delivers, or returns the refusal), and limit the start bullet to the prompt line and the skill-load line.
  - id: F15
    severity: LOW
    class: consistency
    origin: handler
    location: _lyx/discussion/decision-record.md Acceptance criteria
    summary: Several stated bounds have no criterion that reaches them, among them D4's no stop before the grace end and a caller cancel during the hold, D7's refusals that stay and its no-running-turn record, and D8's record-batch, await-batch and status staying open and its unreadable record refusing nothing
    fix: Under Acceptance criteria, add one criterion each. A covering entry resumed before its grace end lets a running verify finish and record its result. A caller cancel during D4's hold returns the cancel error. Under a pause, lyx creel interrupt still refuses on the privilege rule, another sender's hold and an unreadable record. A strand with no running turn stays recorded. record-batch, await-batch and status run under a pause. An unreadable record lets begin-batch run with one Warn.
---

### [BLOCKING] D1's coverage read comes before the idle wait

**Location:** `_lyx/discussion/decision-record.md` D1

**Issue:** `sendVerified` (`internal/shuttleengine/run.go:1457`) calls `awaitIdleSession` and then `typeVerified`, and D1 places its read "first".
A tender pause waits out its grace until covered sessions go idle (`internal/tenderengine/doc.go`, Pause).
That is the same window in which a gate re-prompt's `sendWithin` waits for the writer to go idle.
A session that goes idle inside the grace is listed `already_idle` and never recorded, and the re-prompt then lands in a covered session.
This is the incident's own shape, and the declared `PATTERN-pause-holds-prompts` text forbids it.

### [MEDIUM] D6 misstates the orch watcher's session shape

**Location:** `_lyx/discussion/decision-record.md` D6

**Issue:** The watcher's `State.Strand` is the strand's GUID (`internal/orchengine/state.go:79`), and `recordOrchInEntry` writes `GUID: st.Strand`.
`settledWithoutInterrupt`, `Owner`, `ownsUnnoticedOrch` and `ClearResumeNoticed` all match on that GUID (`internal/orchengine/pause.go`).
A D7 claim written as D6 says would make the watcher record the orch again and send its own Esc.
After the resume, the orch would get no resume line, since tender skips `Orch` sessions.
All four lenses that touched D6 found this.

### [MEDIUM] SetPause on the orch runner holds lyx orch start

**Location:** `_lyx/discussion/decision-record.md` D1

**Issue:** `Runner.SetPause` also arms `waitWhileCovered` in run start (`internal/shuttleengine/run.go:94-120`), which waits with no deadline.
`lyx orch start` reaches `Runner.Start` through the orch runner (`internal/orchcli/start.go`, `internal/orchcli/cli.go`).
Today the orch can start under a hub pause, and `settledWithoutInterrupt` has a branch for an orch that started under the entry.
D1 states only the typing effect of the new geometry.

### [MEDIUM] D4 does not hold a Verify that starts under a pause

**Location:** `_lyx/discussion/decision-record.md` D4

**Issue:** `Verify` goes dirty check, skip check, marker, `verifyrun.Run` (`internal/verifytree/verify.go`), and D4 adds the watch only while the command runs.
D3 keeps a shuttle gate from calling Verify under cover, but landingshed's Merge-In, Finalize and Publish and `lyx webster verify` are outside D3.
Each of them can start a fresh `lyx gate test --wait` under a live pause, and that run is only stopped once the grace end has passed.

### [MEDIUM] The SIGKILL escalation does not follow from the wait delay

**Location:** `_lyx/discussion/decision-record.md` D4

**Issue:** `verifyrun.run` sets `cmd.WaitDelay`, and the group kill lives in `cmd.Cancel` (`internal/proc/procgroup_linux.go`).
After a cancel, exec's wait delay kills `cmd.Process`, which is the shell alone, and only while `Wait` has not returned.
Verify passes an `*os.File` log, so `Wait` returns when `sh` dies on the group SIGTERM.
Nothing then escalates against a `lyx gate test` that is still alive.

### [MEDIUM] D4's cancel change reaches callers Scope calls out

**Location:** `_lyx/discussion/decision-record.md` D4 and Scope

**Issue:** `verifyrun.Run` is also called by `battencli/land.go` `runDeploy` and `runRowCommand`, and by `cardverify/runner.go` `execute`, and each cancels on its timeout.
D4 changes the cancel for all of them.
Scope Out says these "run through `verifyrun` directly and are no verify site", and the record never states that their timeout kill changes.

### [MEDIUM] D8's failed GUID lookup strands Master

**Location:** `_lyx/discussion/decision-record.md` D8

**Issue:** `liftEntries` sends resume lines only to `entry.Sessions` (`internal/tenderengine/resume.go`).
D8's failed lookup records nothing and still refuses with the message to rerun "after the resume line arrives".
Master ends its turn, and no line wakes it after the resume.

### [LOW] D1's fail-open states are unbounded

**Location:** `_lyx/discussion/decision-record.md` D1

**Issue:** The unparsable-name fallback uses the runner's geometry slug.
For runners told the hub geometry that slug is empty, and `Entry.covers` lets only a hub entry cover an empty slug (`internal/pauserecord/record.go`).
D1 also says nothing about a reed lookup of the target's name that fails.

### [LOW] SendConditioned bypasses sendVerified

**Location:** `_lyx/discussion/decision-record.md` D1 and PATTERN changes

**Issue:** `Runner.SendConditioned` calls `typeVerified` directly (`internal/shuttleengine/conditionedsend.go:50`).
Tender revive is safe only because it reads coverage itself, inside its `pauserecord.Update`.

### [LOW] D7's re-read does not exclude a tender resume

**Location:** `_lyx/discussion/decision-record.md` D7

**Issue:** The pane lock in `interruptTarget` is creel's own, and `liftEntries` takes only the record lock.
A resume between the re-read and the key whose send fails busy leaves an interrupted strand with no line.

### [LOW] An Expired finalize runs the gate under cover

**Location:** `_lyx/discussion/decision-record.md` D3 and Open risks

**Issue:** The off-boundary branch calls `probeExpiry`, clears the memo and calls `finalize(OutcomeDone)`, which re-evaluates the closures.
The deadline check is skipped while covered (`internal/shuttleengine/wait.go:401`), so the Open risk names the wrong finalize.

### [LOW] Activity readers misread a paused Verify

**Location:** `_lyx/discussion/decision-record.md` D4

**Issue:** `agentActivityFor` (`internal/loomcli/looparm.go:121`) and battencli's activity reading (`internal/battencli/wire.go:582`) return activity now for any live marker.
D4 names only loom status as a reader of the new state.

### [LOW] The plan-writer note's cancel path is missing at two sites

**Location:** `_lyx/discussion/decision-record.md` Notes for the plan writer

**Issue:** `internal/loomshed/verifygate.go:161` and `internal/websterengine/verifygate.go:217` call Verify with `context.Background()`.

### [LOW] The deferred prompt is not a run-start send

**Location:** `_lyx/discussion/decision-record.md` D2

**Issue:** `Run.DeliverPendingPrompt` is reached from `burlerengine` `ResumeFix` (`internal/burlerengine/fix.go:232`), after the run start's own wait.

### [LOW] Acceptance criteria miss the negative side of several bounds

**Location:** `_lyx/discussion/decision-record.md` Acceptance criteria

**Issue:** D4's rationale rests on a short pause letting a nearly done verify finish, but no criterion reaches a stop that must not happen.
Several of D7's and D8's stated bounds have no criterion either.

## Rejected

- Incomplete enumeration of `sendVerified` callers in D2, beyond the ones filed as F14: the rubric places call-site enumeration with Plan-Write's own lookups.
  The session-cycle sends (`ClearSession`, compact, reload, color, load-skills) all arrive through creel delivery, which D2's creel bullet covers, and the start color is display-only and logged.
- A start loop when the target slug differs from the runner slug: every runner that starts runs shares its target's worktree (batten's runners take the task location), so the two slugs match.
- D7's no-running-turn record and the resume-busy extra line: D7 and Open risks already state these as accepted costs.
- A support-log requirement missing from the record: every lens-review finding marked fixed appears in the record's Decisions, Open risks or notes, and the turn-1 facts the design rests on are restated in D4 and D8.
