---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:149
    summary: D7 names lyx loom resume as a way to restart the ended Go driver chain, but resume spawns no driver for a running child and refuses it
    fix: In D7's Decision, in the sentence that ends "the operator restarts the chain with `lyx loom start` or `lyx loom resume`", name `lyx loom start` alone, run in the task worktree. Add that `lyx loom resume` refuses a child whose status reads running and holds no driver, because its Go spawn is for a halted run only.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:259
    summary: The restart after an owner pause is stated only for the batten owner-probe site; at the wake and lyx orch start sites End also ends batten, so the stated driver-dead notice has no sender and the actual mechanism is neither named nor pinned
    fix: In D7's Decision, split the restart sentence by site. For the owner-probe site keep the driver-dead notice from the surviving batten, then `lyx loom start`. For the hibernate wake and `lyx orch start` sites, where `End` ends batten with the driver chain, state that nothing restarts the chain until `lyx adopt` runs its step 5, `battencli.ResumeInWindow`, which starts a batten that spawns a running child whose spawn it has not confirmed again (battenshed doc.go, the unconfirmed-spawn rule), and that a run left owned elsewhere gets no restart. In the acceptance criterion at the line "A reconcile owned-elsewhere pause at wake or `lyx orch start`...", add what pins it, that a run ended at the wake or at `lyx orch start` and then adopted has its Go driver chain spawned again by the resumed batten with no second driver, and that the owner-probe case restarts through the notice and `lyx loom start`. The implementer should confirm the respawn against `ResumeInWindow` and record the evidence in the Facts of Constraints.
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:131
    summary: D6 exempts a push child from the coverage exit while its slug holds a parent lock record, but the record is read live at each call and batten releases it once Finalize is Done, so a push that outlives the record exits under the owner pause and no lift path respawns it
    fix: In D6's Decision, state when the parent lock record is read, which is at each network call through the gate's `HoldsParentLock` (`HeldBy` over the live records, `internal/tendercli/gate.go`), not at spawn. State the outcome for a marked child that meets a covering owner pause after the record was released at Finalize Done, which is that it puts its flags back and exits. Then either decide that the spawner marks the child as exempt for its whole life when it is spawned under the lock, with the bound that this covers only the pushes that Finalize spawned, or state that this push is deferred and name what respawns it. The lift spawn excludes a pair another machine owns, and `lyx resume <slug>` on a landed run lifts the owner pause without an owner write, so the exclusion applies exactly there; if the push is accepted as deferred, say that it waits for the next push of that worktree. In the acceptance criterion "a push spawned by a Finalize past its parent-side merge under an owner pause is not deferred", pin the same case, a push that checks after the record is released.
---

### [MEDIUM] D7 names `lyx loom resume` as a restart that refuses

**Location:** `_lyx/discussion/decision-record.md:149`
**Issue:** D7 says the operator restarts the ended chain "with `lyx loom start` or `lyx loom resume`".
In `internal/loomcli/resume.go`, a child whose status reads `running` and that has no live driver, run lock or driver lock skips the Go spawn, which is reserved for `status.State != StateRunning`, and returns `noLiveDriverMessage`: "the run has no live driver to wake, and resume starts none".
`start` does spawn: `runDriverSpawnAndWait` sets `mustSpawn` from the missing locks and strand, whatever the status.
The record's way forward is therefore half wrong.
The refusal itself names `lyx loom start`, so an operator is not stuck, but the plan writer would write the criterion and the batten notice text against a false premise.

### [MEDIUM] The restart after an owner pause is pinned for one of D7's three sites

**Location:** `_lyx/discussion/decision-record.md:259`
**Issue:** The focus question is what restarts the chain once the run reads owned here again.
D7 answers for the owner-probe site only: batten survives, `End` ends the driver chain, and batten's existing driver-dead notice (`noticeCondition`, suppressed while `w.pause.Covered`) fires after the owner pause lifts.
The wake and `lyx orch start` sites end "the run's owned processes" with no ancestor to spare, so `lyx batten run` is ended with the chain and nothing is left to send a notice.
The existing mechanism for them is outside the record: `lyx adopt`'s step 5 calls `battencli.ResumeInWindow`, and a fresh batten spawns a running child "whose spawn this batten process has not confirmed" again (`internal/battenshed/doc.go`), with the driver lock refusing a second driver.
The record neither names this nor states the absence of a restart for a run that stays owned elsewhere, and the acceptance criterion at line 259 pins only the probe path's notice.
The criterion also pins the notice and leaves the restart itself unpinned.

### [MEDIUM] A Finalize's push can outlive the parent lock record that exempts it

**Location:** `_lyx/discussion/decision-record.md:131`
**Issue:** The coverage reading of the marked push child is the presence of a parent lock record for its slug.
`internal/tendercli/gate.go` wires `HoldsParentLock` to `parentlock.Store.HeldBy`, a live listing, and `WaitUncovered` re-evaluates it on every poll, so the reading is taken at each call, not at spawn.
Batten keeps a record while the child runs and releases it when the child is in any other state (`internal/battenshed/innerrun_parentlock.go`), so a push spawned at the end of Finalize can check after Finalize is Done and the record is gone.
D6 then sends the child down its exit path under an owner slug entry: it puts back its flags and exits, and the push is deferred, against the criterion's "is not deferred".
The only respawn is `liftEntries`' spawn, which excludes a pair another machine owns.
`internal/adoptcli/doc.go` sends a landed run to `lyx resume <slug>` under its owner pause, a lift with no owner write, so that is the path where the exclusion applies.
Under a hub entry the child waits whatever the record reads, so that case holds.
The record states neither when the record is read, nor that the push can be deferred here, nor what respawns it.

## Rejected

- D4's order for a pair whose task worktree is gone is stated (refusal probe, then `End` for the slug, then the session end by exact name, with no quiet wait, run-lock wait, loop end or driver-lock re-check), it matches `EndSession` in `internal/pairteardown/teardown.go`, where the probe runs for a gone worktree and only the loop steps are guarded by `!gone`, and the acceptance criterion "A teardown of a pair whose task worktree is gone..." pins it. Not a defect.
- A process that survives SIGKILL is settled across D3, D4 and D8 and pinned by the criterion on `End` over such a process: `End` returns an error naming each survivor's pid and role beside the processes it ended, `RefuseWhenBusy` callers and `lyx reed down` refuse with nothing removed and the session live, batten's teardown and the reconcile's forced teardown go on and report it in `SessionResult`, the D7 sites log at Warn, and `lyx tender reap` reports it and exits non-zero. A survivor of a forced teardown has no pair left and surfaces as an orphan in `tender status`, where `reap` reports it again. Not a defect.
