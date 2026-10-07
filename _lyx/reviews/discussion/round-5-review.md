---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:118
    summary: D7's dead-strand refusal names lyx batten run as the way to bring the driver back, but D6's revive and reed resume skip a live strand, so it cannot revive one reed marked retiring
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:112
    summary: D7's awaiting refusal names approve or reject and assumes a watching batten run, while the awaiting hint kept by D6 names lyx loom start; the record does not say which verb fits which awaiting sub-state
  - id: F3
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:123
    summary: D7's rationale says lyx loom start spawns a driver instead of waking a live one, which contradicts D7's own decision text and the code; start wakes a parked live driver and spawns only when none is live
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:90
    summary: D6, D7 and the Acceptance rows name lyx loom start --no-attach as a way forward, but the flag is hidden and ignored since start never attaches
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:58
    summary: D3 calls the parameter a deadline and a bound while it is a 60-minute duration, never names the constant, and the expiry shares the term with the context deadline
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:118
    summary: The record says the run lock for two different locks, the child loom run's lock and batten's own run lock
---

### [MEDIUM] D7's `lyx batten run` way forward does not hold for a retiring driver strand

**Location:** `_lyx/discussion/decision-record.md`, D6 bullet 2 and D7's dead-strand outcome (lines 87 and 118), Acceptance `lyx loom resume` row (line 181).
**Issue:** D7 files a driver strand reed marked retiring under "not live" and sends it down the dead-strand refusal, which names `lyx batten run <slug>` as the way that "brings the driver back (D6)".
D6's revive fires only on a driver strand "that is not live", and it works through reed resume.
`resolveDriverStrandAction` reports a retiring strand as live (`strand.Live && strand.Retiring`), and `planResumeLaunches` skips every live strand, so a revive over a retiring strand relaunches nothing.
The batten run would warn that the revive left the driver not live and name `lyx loom start`, one hop later than the refusal could have said.
The record does state that a retiring strand counts as not live (the driver-liveness question of this round's focus is settled on that point), but D6 and D7 disagree about what that classification reaches.
**Fix:** in D7's dead-strand outcome, split the retiring case from the dead case: a retiring strand's refusal names `lyx loom start` in the task worktree, which removes the retiring strand and spawns a fresh driver, and only a dead strand names both ways.
Make D6's revive condition say "dead (not live and not retiring)" and update the Acceptance row to match.

### [MEDIUM] The awaiting hint and D7's awaiting refusal name different verbs, and D7 assumes a watching batten

**Location:** `_lyx/discussion/decision-record.md`, D6 last bullet (line 92), D7 awaiting outcome (line 112), Acceptance `batten run` and `lyx loom resume` rows.
**Issue:** `battenshed`'s awaiting hints split three ways.
The plain hand-off and the decide-again hint name `lyx loom approve` or `lyx loom reject`.
The hint for a decision already acted on whose driver has not re-stepped names `lyx loom start` "if the driver has ended".
D7 refuses an awaiting run with approve or reject only, "after which batten resumes the child on the recorded decision".
D6 justifies keeping the hints with "since D7 refuses an awaiting run", which explains why they do not name `lyx loom resume` but not why one of them names `lyx loom start`.
So an operator in the acted-on, driver-ended sub-state reads `lyx loom start` from the batten log and approve or reject from `lyx loom resume`.
D7's refusal also holds only while a batten run watches the child: with none, a recorded decision resumes nothing, and `lyx loom approve`'s own envelope points at `lyx loom start`.
**Fix:** state in D7 that the awaiting refusal names approve or reject, and for a run no batten run watches, `lyx batten run <slug>` to resume the child on the recorded decision.
State in D6 that the acted-on, driver-ended hint keeps `lyx loom start` as its driver-ended fallback, and that the awaiting refusal does not read batten's decision marker, so does not name it.
Replace the "since D7 refuses an awaiting run" justification with that.

### [LOW] D7's rationale misdescribes `lyx loom start`

**Location:** `_lyx/discussion/decision-record.md:123`.
**Issue:** The rationale reads "the orch's `lyx loom start` spawns a driver instead of waking a live one".
D7's own decision text says the verb shares the branch `lyx loom start` takes for a parked driver, and `runDriverSpawnAndWait` spawns only when no driver strand is live and no run lock is held; a parked live driver gets the resume line.
The premise is false, and a plan writer who reads only the record is told start cannot wake a driver.
What the new verb adds is that it can never spawn: a dead driver goes back through batten's revive (D6), keeping its own session, rather than being replaced by a fresh one.
**Fix:** rewrite the rationale to say that: the orch needs a verb that wakes a parked driver and cannot spawn a fresh one over a dead driver's session.

### [LOW] The record names `--no-attach`, a hidden and ignored flag

**Location:** `_lyx/discussion/decision-record.md`, D6 (line 90), D7 (lines 118–119), Acceptance rows (lines 179 and 181).
**Issue:** `lyx loom start` never attaches for any caller; its `--no-attach` flag is hidden, "accepted and ignored; kept for existing callers".
The refusals and halt reason the record specifies would pin that flag into new user-facing text and spec rows.
**Fix:** name plain `lyx loom start` in the task worktree.
Where Acceptance relies on "never `lyx loom start` alone" to tell the halt reason apart, say instead that the reason names `lyx loom resume` and names `lyx loom start` only after a failed revive or for a child with no driver strand.

### [LOW] D3 uses one word for a duration, a bound and an expiry, and leaves the constant unnamed

**Location:** `_lyx/discussion/decision-record.md`, Goal (line 6), Scope (line 15), D3 (lines 52–63), Constraints (line 150), Auto-mode assumptions (line 156).
**Issue:** The parameter is a 60-minute duration a test shortens, yet it is called a "deadline", which is a point in time.
The Goal calls it a "bound", and D3 uses "bound" again for `WaitDelay`'s pipe-read limit.
The expiry is also called "deadline", the word for the context deadline `verifyrun` sees.
Verify must derive its own context to tell its expiry from a caller's cancel, which the record leaves implicit.
D3 says production callers pass "the one named constant" that `verifytree` declares but gives no name, so the plan writer picks it.
**Fix:** use "timeout" for the duration and its expiry throughout: a `time.Duration` parameter of `verifytree.Verify`, derived into its own context so a caller's cancel stays distinguishable, with the production value the one constant `verifytree.Timeout` of 60 minutes.
Keep "bound" for `WaitDelay` only, and change the Goal's wording to "a verify gate cannot hang without a timeout".

### [LOW] "The run lock" names two locks

**Location:** `_lyx/discussion/decision-record.md`, D6 last paragraph (line 98), D7 (lines 104, 113–114, 118).
**Issue:** D6 and most of D7 mean the child loom run's lock, taken by a driver.
The parenthetical "`lyx batten run` is refused while one holds the run lock" means batten's own run lock, a different file under the prime.
A reader cannot tell which one the refusal condition reads.
**Fix:** say "the child's run lock" for the loom run's lock and "batten's own run lock" in the parenthetical.

## Settled without a finding

- D6's venue (focus 1): the record claims that `lyx batten status` does not show the halted reason, and the code agrees.
  A halted child persists as an exempt Stuck with batten's state `running` and an empty error, `battenStatusExtras` adds `stuck_reason` only for state `blocked`, and `Activity.Wait` is empty outside blocked, failed and awaiting.
  The record names no surface that does not exist, so the venue is not a defect.
- D7's liveness read of a retiring strand (focus 4): the record states the treatment, matching `resolveDriverStrandAction`; only its consequence is the F1 finding.
- D8's docs scope: `docs/overview.md`'s verb list sits in its module table, and the batten paragraph's claim about the reason on `lyx batten status` is false today, so both edits belong in the commit.
  The overview's other `lyx loom start` mentions remain true, since start still resumes; no further docs finding.
