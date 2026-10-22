---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:78
    summary: D4's two session-end orders put runproc.End ahead of EndSession's refusal probe, so a removal the probe refuses has already ended the paused run's batten, driver and loop
    fix: >-
      In D4, insert the refusal probe (Topology.RemoveRefusal, EndSession step 2) into both orders ahead of End, so they read "D5's run-lock wait, the refusal probe, End, the driver-lock re-check, the loop end, the loop end's own run-lock wait, the session end" and "the quiet wait as today, the refusal probe, End, the driver-lock re-check, the loop end, its run-lock wait, the session end", and add the sentence "A removal the refusal probe refuses ends no owned process." Add to the acceptance criteria a bullet that `lyx fabric remove` on a paused run whose removal is refused (a dirty pair without --force) ends no owned process and leaves the session live.
  - id: F2
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:138
    summary: D7 ends every owned process of a slug an owned-elsewhere verdict pauses, including a Finalize past its parent-side merge that keepsLockForFinalize and PATTERN-owner-machine let finish its landing, so the landing is killed half done
    fix: >-
      In D7, after the sentence that lists the sites, add "A run whose Finalize is past its parent-side merge, read as `keepsLockForFinalize` reads it (current producer Finalize, running or failed transient with no re-step, and the parent holding the task's work), is not ended at any of these sites: it keeps its parent lock and finishes its landing, as `PATTERN-owner-machine` allows, and its processes are ended by the pair's removal." Extend D7's Bound to "only for a slug an owned-elsewhere verdict pauses at that site, never a Finalize past its parent-side merge." Add the Fact that `ownedElsewhere` keeps the parent lock for such a Finalize and that a parent-lock holder passes a slug pause entry at the gate (`internal/battenshed/innerrun_owner.go` `keepsLockForFinalize`, `internal/pauserecord/hold.go` `SlugEntryExempt`), and extend the owned-elsewhere acceptance criterion with a Finalize past its parent-side merge that is not ended at the probe, the wake or `lyx orch start`.
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:85
    summary: reed down's not-running reading omits a live loom driver strand, which holds no driver lock, so a running LLM-driven run reads not running and is ended, against D4's Bound
    fix: >-
      In D4's reed down readings, change "a run not running, which is no driver lock held, the loop lock free and batten not running" to "a run not running, which is no driver lock held, the loop lock free, no live driver strand as the quiet rule reads it, and batten not running", and add to the `lyx reed down` acceptance criterion that a run with a live loom driver strand and no pause is refused with nothing ended.
  - id: F4
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:164
    summary: D4's reed down refusal keeps alive a running unpaused pair's session and pane processes that today's reed down kills, while D9 states reed's session end is unchanged and its Bound does not name that change
    fix: >-
      In D4, after "otherwise it refuses with the way forward `lyx tender pause <slug>`, then `lyx reed down` again, with nothing ended", add "Today `reed down` in a pair kills the session of a running run, whose strands batten then revives; this refusal withdraws that." In D9, change the Bound to "Bound: no change to what reed's session end kills once it runs; `lyx reed down` in a pair no longer runs it for a run that reads running and unpaused (D4), so that session and its pane processes stay live until a pause."
---

### [BLOCKING] D4's orders end owned processes before the refusal probe

**Location:** `_lyx/discussion/decision-record.md:78`

**Issue:** `Teardown.EndSession` runs the quiet wait, then `t.refusal` (`Topology.RemoveRefusal`), and only then the loop end (`internal/pairteardown/teardown.go:166`), and its doc says "a refused removal leaves the session and its strands live".
D4 states both orders as complete sequences ("Its order ... is:") and neither names the refusal probe; End sits directly after the quiet wait or D5's run-lock wait.
A plan writer following the record ends batten, the Go driver, the loop and the pair's pushes and then refuses the removal for dirtiness, leaving a paused run with no processes and a batten record reading `running`, which only the out-of-scope `batten-cancel-verb` settles.
D5's "nothing ended" refusals show the record intends refusals to be non-destructive, so the probe's placement is a missing decision, not a detail for the plan.

### [BLOCKING] D7 ends a Finalize past its parent-side merge

**Location:** `_lyx/discussion/decision-record.md:138`

**Issue:** `innerRunProducer.ownedElsewhere` releases the child's parent lock only when `keepsLockForFinalize` is false: a Finalize past its parent-side merge keeps the lock and finishes its landing (`internal/battenshed/innerrun_owner.go:107`, `:158`).
Finalize's own owner checks skip the owner halt once the parent holds the task's work (`internal/landingshed/finalize_owner.go` `ownerChecks`), and `pauserecord.Network` exempts a parent-lock holder from a slug pause entry (`SlugEntryExempt`, `internal/pauserecord/hold.go:271`), so that Finalize passes the owner pause.
`PATTERN-owner-machine` names the same exception: "bar a Finalize past its parent-side merge".
D7 calls `runproc.End` at every owned-elsewhere pause with no such exception; at batten's probe that ends the driver chain and the Finalize step in it, and the wake's `holdRuns` and `lyx orch start`'s `pauseOwnedRuns` pause such a run as well.
The landing is cut between the parent-side merge and its push, the case the existing code goes out of its way to let finish.

### [MEDIUM] reed down reads a running LLM-driven run as not running

**Location:** `_lyx/discussion/decision-record.md:85`

**Issue:** D4 defines "a run not running" for `reed down` as no driver lock held, the loop lock free and batten not running.
Only the Go driver, `lyx loom drive`, holds the driver lock (`internal/loomcli/doc.go`, `internal/sheddriver/doc.go` `DriveVerb`); a loom driver runs as a reed strand, which `pairteardown`'s quiet rule checks separately ("its driver strand is absent, dead, retiring or parked").
A running, unpaused LLM-driven run with no batten and no step in flight meets all three readings, so `reed down` ends it, contradicting D4's Bound "a running unpaused run is never ended by `fabric remove` (D5) or `reed down`".

### [MEDIUM] D9 does not name the processes reed down now leaves alive

**Location:** `_lyx/discussion/decision-record.md:164`

**Issue:** Today `lyx reed down` calls `Engine.Down` unconditionally (`internal/reedcli/up.go` `downCmd`), which reaps every live pane's descendant closure and late session members (`reapPaneChildren`, `internal/reedengine/lifecycle.go:1150`).
D4 makes `reed down` in a pair refuse a run that reads running and unpaused "with nothing ended", so that session's agents and their pane processes, killed by today's verb, now survive it.
That is the one case where a process killed today now survives: D3's End before the session end changes nothing the reap reaches, since a process reparented by End stays in its pane's session and `sessionMemberPIDs` still reaps it.
D9 says "reed's end of a session is unchanged" and its Bound "no change to what reed's session end kills"; neither it nor D4 states that the operator verb stops reaching the session end for a running run, so the change in observable `reed down` behaviour is recorded nowhere as a change.

## Rejected

- D5's waiting-marker reading (focus 2): D5 now introduces the marker itself, written before `WaitUncovered` and removed before the in-flight marker, and the Fact at line 209 matches `pauserecord.Network` and `awaitRunLockNamingHolder` as read; the reading rests on state the decision creates. Not a defect.
- D3/D4/D5 order against `StartTied`'s parent-death SIGKILL (focus 3): with the checks skipped, the run-lock wait runs before End, and with them not skipped the quiet wait already requires the run lock and loop lock free; D5's mid-merge guarantee and the declared `PATTERN-pair-teardown` text agree with that order. The refusal-probe gap in the same orders is F1.
- Remote git children under `ConfigureGroupKill` (focus 4): D3 reads every record's descendants before the first signal and signals a descendant's own group, and End completes inside the session end before `RemovePair`'s remote step, so no recorded push outlives `remove --remote`; an unregistered agent's push is a pane descendant reaped by the session end, which also precedes `RemovePair`. Not a defect.
- reed down's readings (focus 5): listed in D4 itself, no reconcile and no network call, no `stopped` reading; the not-running reading's gap is F3.
- D8's orphan rule and shield (focus 6): the shield counts only a live process registered with the batten role, so a stale `running` record holds nothing, and an orphan requires no registered linked worktree for the slug, with unregistered processes inside a registered worktree excluded. Every batten owned-elsewhere path, spawn refusals included, goes through `ownedElsewhere`, so D7's site list is complete. Not a defect apart from F2.
