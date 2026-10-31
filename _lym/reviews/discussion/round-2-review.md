---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D6, the On entry bullet and the Bound paragraph
    summary: the entry path ends the row Done on a current done flag without stopping a recorded session that is still live, so the agent can change the checkout after the check and keep working through Describe and Finalize
    fix: In D6's On entry bullet, before the done flag is judged, add that the row stops the recorded session when its strand is live, as the wake bullet does before its deciding read; keep the three outcomes as they are, and for the uncommitted-change outcome say the session is then ensured (resumed) before it is mailed. Extend the D6 acceptance bullet on a flag written before a halt with a case where the session was reopened while halted (by lym reed resume or an interrupt that left it live) and the resume ends the row Done with the session stopped.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D5, the fourth bullet (publish shell guard)
    summary: D5 promises the session a publish-guard steer text of its own and blocked forms, but the publish row has one fixed steer, no Forms, and a spec can only switch it on, and the promised claim that lym pushes only after sampler-done is not something the code guarantees
    fix: Rewrite D5's fourth bullet to say the session's spec sets DenyPublish, so it gets the existing publish row with its existing steer, which already names commit and telling the operator as the way forward; drop "with steer text of its own", "blocked forms" and the claim that lym pushes the branch only after sampler-done ends the row. Move the sampler-done way forward to D8's stencil bullets, where it already is. If a sampler-specific steer is wanted instead, state in D5 that it is a new row with its own switch in internal/shellguard's table that every engine renders (PATTERN-shell-guard-table), and make its text claim only that lym pushes the task branch itself.
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D4, second paragraph (so the admitted strand is always the live one)
    summary: D4 says the admitted strand is always the live one, but D6's Bound lets lym reed resume or the stop chain reopen the session while halted, which the row does not re-record
    fix: In D4, replace "so the admitted strand is always the live one" with a statement scoped to the row's own starts and resumes, and add one sentence that a session reopened outside the row while the run is halted is admitted only if it keeps the recorded strand name, otherwise its decision add is refused until lym loom resume re-records it.
  - id: F4
    severity: NIT
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D4, Bound paragraph
    summary: the Bound says each entry is committed in its own diff as the caller rule's package doc says, but the doc says in the call's one committed diff, and a --from batch commits several entries in one diff
    fix: In D4's Bound, replace "committed in its own diff" with "committed in the call's one diff".
---

### [MEDIUM] D6's entry path ends Done without stopping a live session

**Location:** `_lym/discussion/decision-record.md` D6, the "On entry, before it ensures a session" bullet, read with the Bound paragraph.

**Issue:** The wake bullet stops the session before its checkout and record check; the entry bullet reads the flag "before it ensures a session" and ends Done "without starting or resuming a session", but never stops a session that is already live.
D6's own Bound says the stop chain or `lym reed resume` may reopen the session while the run is halted, and the verb is accepted while halted.
So: halted at the row, session reopened, `sampler-done` run on a clean checkout, `lym loom resume` re-enters, the entry check passes and the row ends Done with the agent still live.
The agent can then commit or dirty the checkout after the check, while Describe and Finalize run, which breaks the Bound's "Describe and Finalize always start from a committed tree" and the D6 acceptance bullet "the session is stopped".
PR-Review's `endSession` in `internal/landingshed/prsession.go`, the model D5 follows, also leaves the session running when the call's context is cancelled, a second way a live strand meets the entry check.
All four lenses and the handler found this independently.
This answers the focus's second question: the wake path leaves no window; the entry path does.

### [MEDIUM] D5's publish-guard steer has no mechanism behind it

**Location:** `_lym/discussion/decision-record.md` D5, fourth bullet.

**Issue:** D5 gives the session "the publish shell guard's blocked forms ... with steer text of its own".
In `internal/shellguard/rows.go` the publish row's `Steer` is the one constant `steerPublishDeny`, its `Forms` is empty, and `internal/shuttleengine/spec.go` offers only `DenyPublish bool` to switch the row on; no path carries a per-session steer.
A sampler-specific text therefore needs a new row in the guard table, which `PATTERN-shell-guard-table` governs and D5 never names.
The text D5 prescribes also claims lym pushes the branch "only after `sampler-done` ends the row"; the publish row's own comment says `lym fabric push`, which the guard does not block, and a run-record commit's detached both-sides push reach origin from the session too, so the steer would assert an ordering the code does not hold.
The existing steer ("lym pushes the task branch after it gates your commits ... commit your change and tell the operator") already fits a sampler session.
This answers the focus's third question: the D5 bound names a steer text the code does not provide; D4's readers hold (see Rejected).

### [LOW] D4's "always the live one" does not cover a session reopened outside the row

**Location:** `_lym/discussion/decision-record.md` D4, second paragraph.

**Issue:** Only the row's own ensure re-records the strand.
D6's Bound admits a reopen by `lym reed resume` or the stop chain while halted, which the row does not re-record, so either that strand keeps the recorded name and the sentence's reasoning is not what makes it hold, or it does not and its `decision add --by operator` is refused while the stale strand stays admitted.
The record states neither outcome.

### [NIT] D4's Bound misquotes the caller rule's package doc

**Location:** `_lym/discussion/decision-record.md` D4, Bound paragraph.

**Issue:** `internal/loomcli/decision.go`'s package doc says entries are committed "in the call's one committed diff"; a `--from` batch commits several entries together, so "its own diff" is not what the doc says.

## Rejected

- existing-paths F1 and walkthrough 4, `decision add` pushes the code branch mid-session: `decision add` commits through `fabricengine.CommitAnchoredPaths`, documented as the no-push commit shape (`internal/fabricengine/commitweftpaths.go`), as the interactions and bound-breaker lenses confirmed; the true part, that other paths reach origin, is folded into F2.
- walkthrough 2, D9 sends the agent through `docs/sandbox-howto.md`'s `go build ./...` steps: D9 names the sandbox hub as the place a trial runs, not the runbook's suite procedure, so the agent is never told to run that step.
- Focus question 1, D6 stamp orderings: held. A halt and resume leave the stamp; a goto away and back raises it (`internal/shedengine/goto.go` writes a goto entry); a relaunch onto the same row does not; no row of the recipe routes `on_stuck` or `on_reroute` onto Sampler-Session; a goto refuses a running status, so a flag can always end its visit at the next wake or resume. No sequence ends the row on a flag from an earlier visit, and no current flag is left unable to end the row.
- Focus question 4, D7 and the watchdog facts: held. `agentActivityFor` in `internal/loomcli/looparm.go` reads exactly the verify marker, parent lock wait, drift lock, non-held wait markers and live agent activity the Constraints list; the template's `step_idle_timeout_min` is 60 and counts trace writes; D7's list is a new reading, and its bound holds since a goto cannot touch a running step and a pause holds without ending it.
