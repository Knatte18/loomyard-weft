---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D9
    summary: D9 rests on a false premise; reed's pair-session end already snapshots pane descendants before kill-session and force-kills survivors, so D9's only real effect is an unbounded narrowing that spares lyx and registered processes
    fix: In D9, record that Engine.Down and EndSessionByName (via reapSessionVia) already snapshot each live pane's descendant closure before kill-session and SIGKILL survivors after reapExitTimeout, plus late session members, for every session the prime's included (internal/reedengine/lifecycle.go reapPaneChildren, overlay.go reapSessionVia). Then either drop D9 and say the existing reap covers the agent-shell case, or restate D9 as a change to that reap, naming only what it changes (for example a start-time match), and in its Bound state what it newly spares and why that is safe. In particular, settle whether an unregistered lyx process in the pane closure, such as an agent's foreground lyx gate test, may now outlive the session; today it is killed. Fix the Rationale (#796's shells were not at a session end) and the support-log Turn 3 line saying the prime's panes were newly excluded, since today they are reaped too.
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D5
    summary: D5 ends a run-lock holder only when it is a step waiting at the network gate, but no existing state records a process waiting at the gate, and the record leaves that reading for Plan to find out
    fix: In D5, state how a holder waiting at the gate is read. The research answer is that pauserecord.Network writes its NetOp marker only after WaitUncovered returns (internal/pauserecord/hold.go Network), ListNetOps lists only calls past the gate, and awaitRunLockNamingHolder names only the newest unfinished step's pid (internal/pairteardown/loop.go). So either add a decision that the gate writes a waiting marker (pid, start time, slug) before its wait and removes or turns it into the in-flight marker after, with D5 reading that marker for the run-lock holder's pid; or drop the exception and state what remove does with a parked step holding the run lock. Add the chosen reading to Constraints as a fact with its source.
  - id: F3
    severity: BLOCKING
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D4, D5
    summary: D4 runs runproc.End before the loop end, which SIGKILLs a step mid-merge through its parent-death signal before any run-lock wait, contradicting D5's promise that a merge under the parent lock is waited for through the run lock
    fix: State in D4 and D5 the exact order of the session end. If D5's guarantee is kept, add that a run-lock wait, bounded by runLockWait and naming its holder, runs before runproc.End and before the loop end whenever the driver checks are skipped. Today's run-lock wait (EndSession step 4) runs after killLoop, and End's SIGTERM of the loop kills its StartTied step child at once (internal/proc/tied_linux.go Pdeathsig SIGKILL; internal/pairteardown/teardown.go EndSession). Replace "the run-lock wait still runs first, bounded as today" with that placement, and align the PATTERN-pair-teardown listed text and D3's line that the step child keeps its SIGKILL path through the loop end; it now dies at End.
  - id: F4
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D3, D4
    summary: End signals a push or step's group, but its remote git child runs in its own process group with no parent-death signal, so an ended push or step leaves git push and ssh running and able to push after the removal
    fix: In D3, record the fact that gitexec starts remote git subcommands with proc.ConfigureGroupKill (Setpgid, kill only on context cancel, no Pdeathsig; internal/gitexec/gitexec.go, internal/proc/procgroup_linux.go), so a remote git child is outside the group End signals. Then decide how End reaches it, for example by reading descendants by parent pid for leaders as well as non-leaders before the first signal, and say so in D3's Decision and Bound. Also state what happens to a git push already past the gate when remove --remote has deleted the remote branch. Drop or narrow the Rationale line that SIGTERM leaves no index.lock, which holds only for git in the signalled group.
  - id: F5
    severity: BLOCKING
    class: design
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D4
    summary: "lyx reed down ends processes under the same readings as D5, but D5's verdicts come from a networked reconcile only remove is exempt from at the gate, so reed down under a hub pause waits with no deadline; stopped and not running are undefined for it"
    fix: In D4, list reed down's readings explicitly, without pointing at D5. State whether reed down runs the run-scope origin reconcile. If it does, give it D5's gate exemption for its own calls, with the same Bound, and state its stopped behaviour. If it does not, name the local readings it uses, such as a live pause entry covering the slug, local board entry done, Finalize Done with batten not running, local owner. Define not running as a concrete reading, for example no driver lock held, loop lock free and batten record not running. Update the reed down acceptance criterion to match.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D7
    summary: D7 ends processes only at the hibernate wake and lyx orch start, but a running batten's owner probe also pauses a run owned elsewhere, the path a live run takes when another machine adopts it, so its driver chain stays parked
    fix: In D7, add batten's owner probe (internal/battenshed/innerrun_owner.go ownedElsewhere, after PauseOwned) as a site that calls runproc.End for the slug. It runs inside lyx batten run, so End spares batten and its ancestors. State whether batten then exits, or which verb ends it. Widen the Bound to every site where an owned-elsewhere verdict pauses the run, and add the batten case to the D7 acceptance criterion.
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D4, D5, D8
    summary: a batten killed by End leaves its hub-level run record reading running, which then shields every survivor of that slug from D8's orphan rule and reap, and makes remove's claim settle keep the claim with a way forward naming a dead batten
    fix: In D4, D5 or D8, state the consequence and decide it. Either D8's shield reads running only when the batten pid recorded in the owner record is alive, or registered under the slug, rather than the record's status alone; or the record names the residue as a bounded risk until batten-cancel-verb lands, with the operator's way forward. Also state what settleBoardClaim (internal/pairteardown/claim.go) does for a running batten record whose process End ended, so the claim warning does not name lyx batten pause for a dead batten.
  - id: F8
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D8
    summary: D8's unregistered-orphan rule tests only a deleted working directory inside the hub, so a lyx process of a live pair whose cwd was removed reads as an orphan and is reaped, contradicting the Bound that reap never ends a process whose pair exists
    fix: In D8, narrow the unregistered rule so the deleted directory must be a removed task worktree, or lie outside every registered linked task worktree, rather than any deleted directory inside the hub. Keep the Bound's promise and add a row to the D8 acceptance criterion for a lyx process in a deleted directory inside a live pair, which is not listed and not reaped.
  - id: F9
    severity: MEDIUM
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D4
    summary: D4's driver-lock re-check refuses the teardown with ErrDriverBusy, but its Bound says batten's own teardown passes no refusal, and the reconcile's forced or wake-held teardown also passes no RefuseWhenBusy; which callers the re-check binds is unstated
    fix: In D4, state that the driver-lock re-check after End refuses only for callers passing RefuseWhenBusy, which are fabric remove and the reconcile's local removal, and say what batten's Worktree-Teardown and the forced reconcile teardown do with a driver lock still held after End, for example end it as today and report DriverWasLive. Align the acceptance criterion that a teardown with the lock held refuses.
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md PATTERN changes, D5
    summary: the listed PATTERN-pair-teardown text skips driver checks only for a paused, ended or foreign pair, but D5 also skips them for a local board entry done or a Finalize Done with batten not running
    fix: Either widen the listed PATTERN-pair-teardown text to name the local done or Finalize Done reading, or state in D5 that ended in the PATTERN text covers those local readings, so the binding text and D5 match.
  - id: F11
    severity: LOW
    class: decision
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D10
    summary: the sixty-minute fork bound rests on covering the longest fork seen, with no evidence recorded
    fix: In D10's Rationale, cite the evidence, the 45-minute Webster fork of 2026-10-14 from the board entry's pause-background-work note, and state the margin sixty minutes leaves over it.
  - id: F12
    severity: LOW
    class: consistency
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D8
    summary: reap ends unregistered orphans by pid with no signal named, and proc.KillPID is SIGKILL, which can leave index.lock, the outcome D3's SIGTERM grace exists to avoid
    fix: In D8, state that reap ends an unregistered orphan with D3's sequence, SIGTERM then up to five seconds then SIGKILL, after the start-time match, or name SIGKILL and add the index.lock risk to Open risks.
  - id: F13
    severity: LOW
    class: consistency
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md Open risks
    summary: the risk that an unawaited shell of a gated run lives until the pair's session end is off, since a Done run removes its strand and strand removal already reaps the pane's process tree
    fix: Rewrite the D11 Open risks line to say the shell lives until the run's strand is removed at its Done finalize, or the pair's session end when the pane is kept, and survives only when it left the pane's process tree.
  - id: F14
    severity: LOW
    class: decision
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D3, D4, Open risks
    summary: on Windows End ends no recorded pid, so a parked Go driver keeps its lock and every fabric remove of a paused run with a live driver refuses there; the record does not state this
    fix: Add to Open risks, or to D4, that on Windows fabric remove of a paused run whose Go driver is live refuses with ErrDriverBusy after End, and name the operator's way forward there.
  - id: F15
    severity: LOW
    class: consistency
    origin: handler
    location: _lyx/discussion/support-log.md Rejected alternatives
    summary: support-log decision references use pre-renumbering numbers; the process-table fallback cites D7 where it is D8, and the transcript alternative cites D10's arrival rule where it is D11
    fix: In support-log.md Rejected alternatives, change (D7) on the process-table scan line to (D8) and D10's arrival rule to D11's arrival rule. relevant.md carries the same drift (the Windows bound of D3 and D7, hibernateengine D9, shuttleengine D10), so renumber those lines to D8, D10 and D11 as well.
  - id: F16
    severity: NIT
    class: consistency
    origin: handler
    location: _lyx/discussion/decision-record.md Constraints
    summary: the fact line says tenderengine.Lift is called by tender resume, but tender resume calls Resume, which shares liftEntries with Lift
    fix: Reword the Constraints fact to say that tender resume (Resume) and Lift share liftEntries, which the hibernate wake and cancel, lyx adopt and the cleanup reconcile reach through Lift, so D6's spawn lives in liftEntries (internal/tenderengine/resume.go).
---

### [BLOCKING] D9 duplicates reed's existing pane reap and silently narrows it

**Location:** `_lyx/discussion/decision-record.md` D9
**Issue:** `Engine.Down` captures `paneProcessTreePIDsLocked()` before `kill-session` and calls `reapPaneChildren`, which waits up to `reapExitTimeout` and force-kills survivors and late session members (`internal/reedengine/lifecycle.go`).
`EndSessionByName` does the same through `reapSessionVia` (`internal/reedengine/overlay.go`).
D9's new parts are therefore only exclusions: lyx executables, registered processes, and "the prime's session left as today", which misstates today.
An agent's foreground unregistered `lyx gate test`, killed today, would survive under D9, and the Bound names no such loosening.
Raised by existing-paths, bound-breaker, walkthrough and the handler.

### [BLOCKING] D5's "holder waiting at the network gate" has no reading

**Location:** `_lyx/discussion/decision-record.md` D5
**Issue:** `pauserecord.Network` waits in `WaitUncovered` first and writes its marker only afterwards (`internal/pauserecord/hold.go`), so a process waiting at the gate leaves no trace, and `awaitRunLockNamingHolder` knows only a step's pid.
The incident's own case, a step parked at the gate holding the run lock, would fall into "any other holder refuses".
This is a fact the design rests on, left for Plan; raised by every lens and the handler.

### [BLOCKING] End before the loop end kills a mid-merge step before any run-lock wait

**Location:** `_lyx/discussion/decision-record.md` D4, D5
**Issue:** D4 places `runproc.End` before the loop end.
The loop has no SIGTERM handler, and its step child is started with `proc.StartTied`, so it gets a parent-death SIGKILL (`internal/proc/tied_linux.go`, `internal/shedverbs/loopchild.go`).
Today's run-lock wait runs after `killLoop` (`internal/pairteardown/teardown.go` `EndSession`), and the quiet wait's run-lock check is one of the skipped driver checks.
D5's guarantee and its acceptance criterion that remove waits for a step doing local work therefore have no wait to rest on.
Raised by all four lenses.

### [BLOCKING] Remote git children escape End

**Location:** `_lyx/discussion/decision-record.md` D3, D4
**Issue:** `gitexec` runs remote subcommands under `proc.ConfigureGroupKill`, a new process group with no parent-death signal (`internal/gitexec/gitexec.go`, `internal/proc/procgroup_linux.go`), which I confirmed.
Signalling a detached push's group, which `Detach`'s Setsid makes the push lead, leaves its `git push` running, reparented, up to its deadline.
A push past the gate can then recreate a remote branch `remove --remote` just deleted.
D3's "ends every live recorded process" and D4's "end all of it" do not hold for the network work the task targets.

### [BLOCKING] `reed down`'s readings are undefined and can hang at the gate

**Location:** `_lyx/discussion/decision-record.md` D4
**Issue:** D5's `ended`, `owned_elsewhere` and `stopped` are verdicts of the run-scope origin reconcile, a network read.
D5 exempts only the remove process from the gate, so a `reed down` that runs the reconcile under a hub pause waits with no deadline, the incident's hang on a new verb.
A `reed down` that reads only local state cannot produce those verdicts.
"Not running" is not defined.
Raised by interactions and walkthrough.

### [MEDIUM] D7 misses batten's owner-probe pause

**Location:** `_lyx/discussion/decision-record.md` D7
**Issue:** `innerRunProducer.ownedElsewhere` pauses the slug under the owner caller (`internal/battenshed/innerrun_owner.go`), which I confirmed.
That is the live-run adoption path, and D7 ends nothing there, so the driver chain on the machine the run left stays parked, the pattern D7 exists to close.

### [MEDIUM] A stale running batten record shields survivors and misleads the claim settle

**Location:** `_lyx/discussion/decision-record.md` D4, D5, D8
**Issue:** D4 says a killed batten leaves its record reading `running`.
That record outlives the pair, so D8's shield keeps every surviving process of the slug out of the orphan list and `reap`.
`settleBoardClaim` keeps the claim and names `lyx batten pause` for a batten that is already dead (`internal/pairteardown/claim.go`).
Raised by existing-paths and interactions.

### [MEDIUM] D8's deleted-directory rule can reap a live pair's process

**Location:** `_lyx/discussion/decision-record.md` D8
**Issue:** The rule matches any deleted cwd inside the hub.
A lyx process whose cwd, such as a finalized run directory, was removed inside a live pair reads as an orphan, which contradicts the Bound.

### [MEDIUM] Which callers the driver-lock re-check binds is unstated

**Location:** `_lyx/discussion/decision-record.md` D4
**Issue:** The Decision has "the teardown" refuse with `ErrDriverBusy` after `End`.
The Bound says batten's teardown passes no refusal, and the reconcile's forced and wake-held teardowns pass no `RefuseWhenBusy` (`internal/pairteardown/doc.go`).

### [LOW] PATTERN text and D5's skip readings differ

**Location:** `_lyx/discussion/decision-record.md` PATTERN changes, D5
**Issue:** The listed binding text names paused, ended or foreign pairs.
D5 also skips for a local done entry or for a Finalize Done with batten not running.

### [LOW] The sixty-minute bound's evidence is not recorded

**Location:** `_lyx/discussion/decision-record.md` D10
**Issue:** The record says "covers the longest fork seen" without naming the 45-minute fork the board entry records.

### [LOW] reap's signal for unregistered orphans is unstated

**Location:** `_lyx/discussion/decision-record.md` D8
**Issue:** Ending "by pid" with `proc.KillPID` is SIGKILL, which can leave `index.lock`.

### [LOW] The D11 Open risks line misplaces the shell's end

**Location:** `_lyx/discussion/decision-record.md` Open risks
**Issue:** A Done run removes its strand, and strand removal reaps the pane tree (`internal/shuttleengine/doc.go`), so the shell does not normally live until the pair's session end.

### [LOW] On Windows, removing a paused run always refuses

**Location:** `_lyx/discussion/decision-record.md` D3, D4, Open risks
**Issue:** On Windows `End` ends no recorded pid, so the Go driver's lock stays held, and D4's re-check refuses.
Open risks says only that the processes are listed and never ended.

### [LOW] Stale decision numbers in the support log

**Location:** `_lyx/discussion/support-log.md` Rejected alternatives
**Issue:** "(D7)" on the process-table fallback should be D8, and "D10's arrival rule" should be D11.
Every other D-number reference in both files matches the final numbering.

### [NIT] The Lift fact names the wrong function for tender resume

**Location:** `_lyx/discussion/decision-record.md` Constraints
**Issue:** `Resume` and `Lift` share `liftEntries` (`internal/tenderengine/resume.go`).
`lyx tender resume` does not call `Lift`.

## Rejected

- existing-paths' D10 finding that confirmation's state reading does not carry the fork or shell kind: the record already states, with its source, the fact the design rests on, that the Stop payload types each task (`claudeengine/background.go`). Carrying that kind into confirmation's reading is plumbing for Plan, and the rubric's enumeration exclusion covers it.
- interactions' note that D3's step-child wording is loose: folded into F3's fix, not a separate defect.

## Focus verdicts

- Relocation (D2, D3, D6 rationale history): not a defect. D2's and D6's departures from the board entry carry requirements the plan writer needs, and `e610f9250` and #741 are short source citations, not narrative.
- Board-entry edits (D2, D6, Auto-mode assumptions): not a defect. Each states a departure, or how an existing line is met, and none asks for an edit of the entry.
- D8's "no registered linked task worktree" matches the quiet rule's existing "gone" reading, and its deleted-directory reading matches the `(deleted)` cwd suffix the replaced-binary part already reads, so neither is a deferred question; D6's "Bolt ahead of its upstream" has `gitrepo.HasUnpushed`.
- The SIGTERM-handler fact and the LLM driver never entering `Wait` are recorded with evidence, and both held.
