---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md, Open risks second bullet and the last Constraints fact (watchdog)
    summary: The record leaves for later whether Beam-Await and PR-Review are already exempt from the step watchdog, though the code answers it now (they are not), and the Constraints fact gives the wrong mechanism (a held wait marker) for the PR-Review session.
    fix: Replace the last Constraints fact with one stating that, from the code, the step watchdog has no row exemption today. agentActivityFor in internal/loomcli/looparm.go counts only a non-held shuttle wait marker, a live verify, a parent lock wait, a drift lock and the transcript or events times of a live agent run. A session started through StartSession writes no wait marker, PR-Review's and Beam-Await's wakes write no trace line, and reviewWaitingFor only feeds the status display. So a Beam-Await step and a PR-Review step with no agent activity are killed after step_idle_timeout_min today, and D7's list ends that for both. Delete the second Open risks bullet.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md, D6 entry-stamp bullet and the D6 acceptance bullet on a flag written before a reroute or goto
    summary: The Done-count stamp does not move when the run leaves the row by a goto without a Done, so a flag written while halted at the row ends a later re-entry Done at once, against the acceptance bullet and the stamp's purpose.
    fix: In D6, make the stamp the count of the row's entries into Sampler-Session in the status history, not of its Done entries, so every re-entry raises it, and state that a halt and a resume do not count as an entry. Add an acceptance bullet in D6 saying a flag written while halted at the row, followed by a goto away and a return with no Done in between, is ignored after the return.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md, D6 bullet "A wake that reads a done flag carrying the current stamp"
    summary: D6 never orders the session stop against the checkout check. Checking before stopping lets the agent dirty the tree after the check, and stopping before checking leaves the dirty branch mailing a stopped session.
    fix: In D6's done-flag wake bullet, state the order. The wake first stops the session, as PR-Review's wake does before its deciding read, then checks the checkout and the record. On an uncommitted change it clears the flag, resumes the session, and then mails it the reason. Add an acceptance bullet in D6 saying a flag read with a dirty checkout leaves the session live and mailed.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md, D6 Bound sentence on a flag accepted while halted, against D5's session start
    summary: D6 promises a resume after sampler-done while halted ends the row Done at once, but D5 ensures a session on entry. A dead strand whose recorded session cannot be resumed then halts Stuck again on every resume, and the flag never ends the row.
    fix: Add to D6 that on entry the row reads the done flag before it ensures a session. A flag carrying the current stamp, with a clean checkout and an intact record, ends the row Done without starting or resuming a session. Extend the D6 acceptance bullet on a flag written before a halt to cover a run whose recorded session can no longer be resumed.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md, D4 Bound
    summary: One leg of D4's bound on the unverified --by operator label is false. The Describe stencil reads the added entries as amended scope but never lists them in the description, and on an empty landing no one reads the description before it lands.
    fix: In D4's Bound, delete the words and Describe shows every added entry. Add that each entry is append-only, labelled, dated and committed in its own diff, as the caller rule's package doc says, so the operator can read every entry in the record's history.
  - id: F6
    severity: MEDIUM
    class: consistency
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md, Acceptance criteria bullet D1, D2, D3 with code_review false
    summary: Only Code-Bouncer can be recorded Skipped, because a skipped row routes by on_done to Describe and the two Burler rows are reached only on on_stuck edges, so the bullet asserts history entries that never occur.
    fix: In that acceptance bullet, replace the three Code-Review rows are recorded Skipped and spawn nothing with Code-Bouncer is recorded Skipped, the run goes on to Describe, and no Code-Review row spawns anything.
  - id: F7
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md, D6 entry-stamp bullet and the D6 acceptance bullet naming a reroute
    summary: D6 says Finalize's reroute returns to Sampler-Session, but D1 keeps on_reroute Code-Bouncer, after Sampler-Session on the path, so no reroute re-enters the row.
    fix: In D6's stamp bullet, delete by Finalize's reroute or, so the return names only a goto. In the D6 acceptance bullet, change before a reroute or goto back to the row to before a goto back to the row.
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md, D3 sentence naming the landing's CodeReviewOff
    summary: D3 names CodeReviewOff as a perch-off reading on a sampler run, but only Parent-Align reads it, and D1 gives sampler no Parent-Align row.
    fix: In D3, delete and the landing's CodeReviewOff in internal/loomcli/landingdeps.go, so the sentence names Finalize's rerouteReviewed and codeBudgetZero as the readings that follow the perch rows' entry_gate.
  - id: F9
    severity: LOW
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md, D5 bullet on the publish shell guard
    summary: D5 gives the sampler session PR-Review's publish shell guard, whose steer text says lym pushes the task branch after gating the agent's commits, which nothing does at Sampler-Session.
    fix: In D5, state that the session gets the publish shell guard's blocked forms with steer text of its own, saying the run pushes and lands only after sampler-done, through Finalize. Alternatively, state that the shared steer text is reworded to hold for both rows.
  - id: F10
    severity: NIT
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md, D7 Bound
    summary: D7's bound says a wedged step at a listed row ends as batten's Run-Shed step already does, but internal/battencli/arm.go:514 says that one is ended only by the operator's pause or by ending the detached loop job.
    fix: In D7's Bound, delete as batten's Run-Shed step already is, leaving the bound's own statement.
---

### [MEDIUM] The watchdog question is left open, though the code answers it

**Location:** `decision-record.md`, the Open risks second bullet and the last Constraints fact.
**Issue:** The Open risk says no one checked whether `Beam-Await` and `PR-Review` are already exempt from the step watchdog.
The code answers it:
- `agentActivityFor` (`internal/loomcli/looparm.go:132`) has no row exemption.
- A `StartSession` session writes no wait marker (`internal/shuttleengine/run.go:515`, `waitmark.go:239`).
- Neither wait logs on a quiet wake (`internal/landingshed/prreview.go:385-470`, `internal/pawlengine/beamawait.go`).
- `reviewWaitingFor` feeds only the status display (`internal/loomcli/reviewstatus.go:163`).

The rubric counts a fact left for later as a finding.
The Constraints fact also attributes the kill to a held wait marker, which this session never has.
Three lenses and the handler found this independently.

### [MEDIUM] The Done-count stamp does not move across a goto without a Done

**Location:** `decision-record.md`, D6 stamp bullet.
**Issue:** `goto` writes an `OutcomeGoto` entry, and Done entries never fold (`internal/shedengine/run.go` `appendOrFold`).
Suppose the run is halted at the row, the flag is written, the run goes to `Record-Write` by `goto` and later returns.
The stamp still matches, so the new visit ends Done before the operator works with it.
The interactions, existing-paths and bound-breaker lenses found this independently.

### [MEDIUM] The order of the session stop and the checkout check is unstated

**Location:** `decision-record.md`, D6 done-flag wake bullet.
**Issue:** Either order breaks D6.
Checking before stopping lets the live agent write after the check, and stopping before checking leaves the dirty branch mailing a stopped session.
PR-Review stops its session before the deciding read and resumes it when the review goes on (`internal/landingshed/prreview.go:446-466`).

### [MEDIUM] A flag accepted while halted cannot end a row whose session cannot be resumed

**Location:** `decision-record.md`, D6 Bound, against D5.
**Issue:** D5 ensures a session on entry, and a non-transient resume failure is Stuck with Halt (`internal/landingshed/prsession.go:63-87`).
D6 promises that the next resume ends the row Done at once, but nothing orders the flag read before the session start.

### [MEDIUM] The Describe leg of D4's bound does not hold

**Location:** `decision-record.md`, D4 Bound.
**Issue:** `landing-template-describe.md` reads the `Added after Discussion` entries as amended scope only and never lists them in the description, so they are not shown to anyone.
The handler confirmed this from the stencil's "What to read" section.

### [MEDIUM] The acceptance bullet claims three Skipped rows

**Location:** `decision-record.md`, Acceptance criteria, the D1, D2, D3 bullet.
**Issue:** `enter` routes a skipped row by its `on_done` (`internal/shedengine/entry.go`).
Code-Bouncer's `on_done` is Describe, so the two Burler rows are never entered and get no Skipped entry.

### [LOW] D6 names a reroute that never reaches Sampler-Session

**Location:** `decision-record.md`, the D6 stamp bullet and its acceptance bullet.
**Issue:** D1's `on_reroute: Code-Bouncer` comes after Sampler-Session on the path, so no reroute re-enters the row.

### [LOW] D3 names a reader that sampler never runs

**Location:** `decision-record.md`, D3.
**Issue:** `CodeReviewOff` is read only at `internal/landingshed/parentalign.go:435`, and sampler has no Parent-Align row.

### [LOW] The reused publish guard's steer text is false at Sampler-Session

**Location:** `decision-record.md`, D5.
**Issue:** The steer text in `internal/shellguard/rows.go` describes PR-Review's gate-and-push flow, which does not exist at this row.

### [NIT] D7's bound miscites Run-Shed

**Location:** `decision-record.md`, D7 Bound.
**Issue:** `internal/battencli/arm.go:514` ends a wedged Run-Shed step by a pause or by ending the loop job, not as D7 says.

## Rejected

- interactions F3 (a session reopened by `lym reed resume` runs under a new strand, so D4 refuses it): reed keeps a strand's record through a resume (`internal/reedengine/doc.go:235`), so the reopened session keeps the recorded strand and D4's admitted strand stays the live one.

The handler also checked the focus items that turned up no finding:
- D2's profile values and defaults agree with D3 and the acceptance bullets.
- D1 and D3's perch gate against `on_reroute` halts Finalize when the perch is off.
- D6's covered-pause and dead-strand branches each have an acceptance criterion.
- D5, D6 and D9 hold no deliberation that belongs in the support log.
