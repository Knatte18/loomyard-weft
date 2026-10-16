---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md (D6, Acceptance criteria)
    summary: D6's 60 s bound ignores the sweep's reset to the 30 s base on new mail and on a failed listing, so a post or a failed listing inside the confirm interval pushes the release past 60 s
    fix: In D6, state what a new post, reply, forward, return or inject and a failed listing do to a pending confirm, either keeping the confirm deadline (a reset never lengthens a pending confirm) or narrowing the Rationale's 60 s claim to a pool with no new mail and successful listings; make the Acceptance criterion's removed-holder line name the same condition, and add a criterion row for a post arriving inside the confirm interval.
  - id: F2
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md (D6)
    summary: D6 says the confirm interval replaces the backoff wait, but the sweep that first misses a removed holder always sees a changed strand-name set and sets the 30 s base, and D6 does not say the confirm interval wins over that base reset
    fix: Reword D6's first bullet to say that after a sweep that misses a holder for the first time, the next sweep runs after the 10 s confirm interval in place of whatever wait that sweep would otherwise set, the changed-set base reset included.
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md (D4, Acceptance criteria)
    summary: D4 rests on a false premise; a daemon that loses the start race exits once the winner holds the hub lock, which the winner takes before it folds the journal and binds the socket, so the one last connect can fail where today's backoff connects, and the Bound's claim that the signal only shortens the wait misses that a shortened wait can turn a success into an error
    fix: In D4, make the exit signal carry whether the spawned daemon exited because another daemon holds the lock (ErrDaemonRunning, by exit status or equivalent), and on that exit keep the existing backoff instead of making one last connect; restate the Bound to say the signal ends the wait early only on an exit other than a lost start race; reword the acceptance line about a daemon that exits because another daemon already serves to cover a winner that holds the lock but does not yet answer.
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md (D9)
    summary: D9's new shuttleengine seam runs claude's whole startup-gate classification, which covers the bypass-permissions gate as well as the folder-trust gate, and is an exported production seam any package can call, yet its Bound names neither the second gate nor who may call it
    fix: In D9's Bound, state that the provider's classification covers every one-time startup gate it knows, for claude the folder-trust gate and the bypass-permissions risk gate, and either restrict the seam to the folder-trust gate or state that accepting the bypass gate is let through and bounded by the strand's own launch flags; also state who may call the seam (test helpers only, or any caller) and that the trust-list side effect follows every call, not only the llm smoke test's.
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md (D1)
    summary: D1's cut test is not valid JSON while the fold skips a line that fails to parse as a record, so a last line that is valid JSON but no record is skipped by the fold and kept by the repair, against D1's own Bound that the skipped line is always the cut line
    fix: In D1, replace when that line is not valid JSON with when that line does not parse as a record, the same test the fold's skip rule applies, and add to the shared-definition sentence that the repair and the fold share the parse test as well as the last-line definition.
  - id: F6
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md (Scope)
    summary: Scope's same-commit doc list omits internal/creelcli/doc.go, which describes the client's give-up path D4 changes and the refusal output D2 changes, and internal/shuttleengine/doc.go, which documents the engine seams D7 and D9 extend
    fix: Add internal/creelcli/doc.go (D2's stderr fallback, D4's exit signal) and internal/shuttleengine/doc.go (D7's IdleProbe field, D9's gate seam) to Scope's Docs in the same commit bullet.
---

### [MEDIUM] D6's 60 s bound fails under new mail or a failed listing

**Location:** `_lyx/discussion/decision-record.md`, D6 and the Acceptance criteria line on removed holders.

**Issue:** `sweepLoop` in `internal/creelengine/sweep.go` restarts its timer at `sweepBackoffBase` (30 s) whenever `sweepWake` fires with a changed `mailSequence`, and `Scheduler.wake` fires `sweepWake` on every post, reply, forward, return and inject.
A failed listing also sets the wait to the base.
So a removed holder whose first miss lands at up to 50 s after the removal, followed by a post 5 s into the 10 s confirm interval, gets its second sweep 30 s later: about 85 s, not 60 s.
D6's Rationale claims 60 s "for every holder the sweep covers", and the acceptance line says "whatever the sweep's backoff had grown to", but neither states the quiet-pool and successful-listing conditions the arithmetic needs.
The start point of the acceptance line, the removal, does match the Decision's arithmetic (ceiling after the last sweep, then the confirm interval) on a quiet pool.

### [MEDIUM] D6 does not say the confirm interval overrides the changed-set base reset

**Location:** `_lyx/discussion/decision-record.md`, D6 first bullet.

**Issue:** removing a strand removes its name from the listing, so the sweep that first misses it always reports `changed` and today sets the 30 s base, not the doubled backoff.
D6 says the confirm interval runs "instead of the backoff wait", which a plan writer can read as replacing only the doubling branch, leaving the first-miss case at 30 s and the bound at 80 s.

### [MEDIUM] D4's last connect can fail against a winner that holds the lock but does not yet listen

**Location:** `_lyx/discussion/decision-record.md`, D4 and its acceptance lines.

**Issue:** `Serve` in `internal/creelengine/server.go` takes the hub lock with `lock.TryAcquireWriteLock`, returns `ErrDaemonRunning` at once when it is held, and only after that folds the journal through `OpenStore` and binds the socket.
A spawned daemon that loses the race therefore exits while the winner may still be folding, so D4's premise that it "exits while that other daemon answers" is false.
D4's single last connect then fails and the client returns the daemon-log error, where today's backoff would have connected.
The Bound "the signal only shortens the wait" states no exposure for this case.

### [MEDIUM] D9's gate seam bound omits the bypass-permissions gate and its callers

**Location:** `_lyx/discussion/decision-record.md`, D9 Bound.

**Issue:** `startupGateNeedles` and `gateAcceptNeedles` in `internal/shuttleengine/claudeengine/startup.go` classify two gates, the folder-trust gate and the bypass-permissions gate whose accepting option reads "Yes, I accept".
A seam that "runs the provider's own startup-gate classification and accept sequence on a live strand" accepts both.
D9's Bound limits the seam to the provider's own sequence on a named strand when a gate reads, but does not say the bypass-permissions risk acceptance is among what it lets through, nor whether a production caller other than the llm smoke test may reach it.
The trust-list side effect is bounded only for "an llm run the operator starts by hand", while the seam itself is engine-neutral production code.

### [LOW] D1's cut test differs from the fold's skip test

**Location:** `_lyx/discussion/decision-record.md`, D1.

**Issue:** `foldJournal` in `internal/creelengine/fold.go` skips the newest file's last non-blank line when `json.Unmarshal` into `record` fails.
D1 cuts that line only when it "is not valid JSON", the `json.Valid` test `repairTail` uses today.
A torn crash prefix is never valid JSON, so the crash case is covered, but a last line such as `{"id":"x"}` is skipped by the fold, kept by the repair, and becomes a middle line on the next append, failing the next start, which D1's Bound says cannot happen.
The blank-trailing-lines and single-torn-line cases do stay inside the Bound: truncating to the end of the previous line drops the trailing blanks too, and a lone torn line truncates to empty.

### [LOW] Scope's doc list misses two package docs

**Location:** `_lyx/discussion/decision-record.md`, Scope.

**Issue:** `internal/creelcli/doc.go` describes the spawn and "a client that gives up connecting quotes the started daemon's lines", which D4 changes, and CLAUDE.md's docs rule covers D2's new observable stderr line.
`internal/shuttleengine/doc.go` documents the engine seams; D7 adds an `IdleProbe` field and D9 adds a gate seam.
Scope's "Docs in the same commit" bullet names only `internal/creelengine/doc.go`, the PATTERN background and the sandbox suite.

## Focus items judged not a defect

- D6 and Open risks, guard exposure: the Bound's "two successful listings at least 10 s apart" and the Open risks entry state the same exposure, the drop from at least 30 s to at least 10 s; the base reset only lengthens the gap, so the 10 s floor holds.
- D3 and D8: D8 only shortens the worker's wait and changes neither `probeBackoffCeiling` nor the probe's pane read, so D3's 16 s freshness bound stays correct; D3 states what the tell rule repeats (A, B, A three times) and what it drops (a stale stall), and adds a report, not a pass through a guard.
- D2, D5 and D7 add an output line, an event entry and a stall; none lets through anything a guard refused, so their lack or presence of a Bound is not a finding.
  D2's SIGPIPE clause matches Go's behaviour on a broken fd 1, and the read acknowledgement in `internal/creelengine/client.go` is sent only after the handler returns nil, so the mail stays unread as D2 says.
- Turn 3 of the support log: its probe and sweep timings are diagnosis; D8's TTL criterion and D6's numbers carry every requirement the plan writer needs, so nothing is lost there.
- Scope against Decisions: every in-scope module item traces to a Decision, and no Decision changes this task's own board entry.
