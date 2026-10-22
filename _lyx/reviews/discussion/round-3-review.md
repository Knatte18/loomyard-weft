---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:145
    summary: D7 says batten's dead-strand revival restarts the driver chain once the run reads owned here again, but that revival fires only for a halted or awaiting child with a dead driver strand, restarts no process, and the chain End ends has no strand
    fix: In D7 replace the sentence "When the probe reads the run owned here again, batten treats a child whose driver chain this `End` ended as halted with a dead driver strand, so its dead-strand revival restarts the chain as after any dead driver." with "Batten adds no restart; a running child whose Go driver `End` ended holds neither driver lock nor strand, so batten sends its existing driver-dead notice, and the operator restarts the chain with `lyx loom start` or `lyx loom resume` once the run is owned here again." Then in the acceptance criterion that opens "A reconcile owned-elsewhere pause at wake", replace "which keeps running and revives the chain when the run reads owned here again" with "which keeps running and, once the run reads owned here again, sends its driver-dead notice for a Go driver `End` ended". If the author instead wants an automatic restart, D7 must name the new batten rule that gives it, for example clearing the child's spawn-confirmation marker after an `End` that ended a driver-chain process so the unconfirmed-running-child respawn runs, and the acceptance criterion must pin that rule.
  - id: F2
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:127
    summary: D6 never says whether a marked push child reads coverage with the parent-lock holder's slug-entry pass, so under D7 a Finalize that finishes at an owner pause may have its status push deferred and never respawned
    fix: In D6, after the sentence "A detached push or board-sync child, marked by a hidden flag its spawner sets ..., checks its coverage before it marks a pending flag or takes its lock, and exits at once when covered." add "The child reads coverage as the gate does, so a child of a process whose slug holds a parent lock record passes a slug entry and exits only for a hub entry." Add to the acceptance criterion that opens "A detached push or board sync under a covering pause" a clause that a push spawned by a Finalize past its parent-side merge under an owner pause is not deferred. If the author instead wants such a push deferred, D6's lift spawn must name the owed push of a pair owned elsewhere whose Finalize landed, since the lift now excludes every pair owned elsewhere.
  - id: F3
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:77
    summary: D4 states both session-end orders only for a pair whose task worktree is present, and says nothing about whether End runs when the worktree is gone, where the driver-lock re-check and loop end have no worktree to read
    fix: In D4, after the two order sentences, add "A pair whose task worktree is gone skips the quiet wait, the driver-lock re-check and the loop end as today, still runs the refusal probe, and calls `End` for the slug before the session end by exact name." If the author decides `End` does not run there, add instead that the survivors of a gone pair are `lyx tender reap`'s (D8) and name that in D4. Add one acceptance criterion pinning the chosen behavior for a gone worktree.
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:98
    summary: D4's Bound says a running unpaused run is never ended by fabric remove, but D5 skips the driver checks and ends the owned processes on an ended or owned_elsewhere verdict whatever the local pause reads
    fix: In D4's Bound replace "a running unpaused run is never ended by `fabric remove` (D5) or `reed down`" with "a running unpaused run is never ended by `reed down`, nor by `fabric remove` bar a run the reconcile reads ended or owned elsewhere (D5)".
  - id: F5
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:60
    summary: D3 returns End only once each process is confirmed gone after SIGKILL, with no bound and no stated outcome for a process that survives it, and every end verb and teardown waits on End
    fix: In D3 after "sends SIGKILL to what is left" replace "and returns once each is confirmed gone" with "waits one further engine-constant bound, and returns an error naming the pid and role of each process still alive". State in D4 that such an error refuses the removal with nothing removed, and in D8 that `lyx tender reap` reports the survivors and exits non-zero. Add one `End` acceptance row for a process that survives SIGKILL.
---

### [BLOCKING] D7 names a revival mechanism that does not restart the chain

**Location:** `_lyx/discussion/decision-record.md:145`, acceptance criterion at line 252.
**Issue:** D7 relies on "dead-strand revival" to restart the driver chain after `End` ended it and the run reads owned here again.
Batten's documented revival (`internal/battenshed/doc.go`, the halt paragraph) applies to a halted or awaiting child whose reed state holds a dead driver strand, and it revives the pair's strands through reed resume while the run stays halted and nothing is resumed.
What `End` ends in a Go-driven run is the Go driver, the step loop and pushes, none of which is a reed strand, and an LLM driver strand is not registered under the owner record so `End` does not touch it.
A running child whose driver `End` ended has neither driver lock nor strand, and its spawn is already confirmed by this batten process (`SpawnConfirmedFile`, `innerrun.go`), so the respawn of an unconfirmed running child does not fire; the documented outcome is the driver-dead notice, which is a signal only.
The acceptance criterion therefore pins a restart no existing or decided mechanism produces, and a plan writer has to invent one.

### [MEDIUM] D6's coverage check and D7's Finalize exception are not reconciled

**Location:** `_lyx/discussion/decision-record.md:127`, with D7 at line 142.
**Issue:** D7 lets a Finalize past its parent-side merge finish its landing at an owner pause, which works at the gate because a process whose slug holds a parent lock record passes a slug entry (`HoldsParentLock` in `internal/tendercli/gate.go`, `SlugEntryExempt` in `internal/pauserecord/hold.go`).
Each commit the Finalize makes spawns a detached status push, the child D6 now makes exit at once when "covered", and D6 does not say which reading of coverage the child uses.
If it uses plain slug coverage, the Finalize's Done push is deferred, and the lift spawn excludes every pair owned elsewhere, so nothing respawns it from this machine; the acceptance criterion that the Finalize "finishes its landing" is then not met for its records.
The record has to say which reading the child takes.

### [MEDIUM] D4's orders omit a pair whose task worktree is gone

**Location:** `_lyx/discussion/decision-record.md:77`.
**Issue:** `EndSession` has a separate branch for a gone task worktree: no quiet wait, no loop end, no run-lock wait, and the session ended by exact name (`internal/pairteardown/teardown.go` `EndSession`, `doc.go` Quiet rule).
Both orders D4 states name the driver-lock re-check and the loop end, which read the task worktree and cannot run there.
Whether `End` runs for a gone pair decides whether a push registered under its slug is ended by the removal or left to `lyx tender reap`; the record leaves both open.

### [LOW] D4's Bound overstates what D5 allows

**Location:** `_lyx/discussion/decision-record.md:98`.
**Issue:** The Bound reads "a running unpaused run is never ended by `fabric remove` (D5)", while D5 skips the driver checks and ends the owned processes on an `ended` or `owned_elsewhere` verdict, a state a run reads locally as running and unpaused until batten's owner probe or the wake pauses it.
The two paragraphs disagree on what `fabric remove` may end.

### [LOW] D3's End has no bound after SIGKILL and no outcome for a survivor

**Location:** `_lyx/discussion/decision-record.md:60`.
**Issue:** D3 bounds the SIGTERM grace at five seconds and then "returns once each is confirmed gone".
A process that does not die on SIGKILL, such as one in uninterruptible sleep, holds `End`, and with it a teardown, a `reed down` and `lyx tender reap`, with no deadline and no stated result.
The record's other waits carry a bound and a way forward.

## Rejected

- Focus 1, whether every order puts the probe ahead of `End`: both orders D4 states put the refusal probe before `End`, matching the `EndSession` order in `teardown.go`, and the acceptance criterion for a refused removal such as a dirty pair without `--force` pins that no owned process is ended and the session stays live. Not a defect.
- Focus 3, whether a running unpaused LLM-driven run can read as not running: D4's reed-down reading requires no live driver strand as the quiet rule reads it, which counts a driver with a park marker as not live; the marker is written only when the run halted at a hand-back and removed by the resume (`driverresume.go`), and the acceptance criterion pins that a live loom driver strand makes `reed down` refuse. The Go driver is read through its driver lock. Not a defect.
- Focus 4, whether the record states that a running unpaused pair's session and pane processes survive `reed down`: D4 states the refusal with nothing ended, D9's Bound states the session and its pane processes stay live until one of D4's readings holds, and the two agree. Not a defect.
- Focus 2, whether D7 lets such a Finalize finish at the batten probe, the wake and `lyx orch start`: the Decision, the Bound and the acceptance criterion all exclude it from `End` at all three sites, and a later removal waits on the run lock. The gaps that remain are F1 and F2.
