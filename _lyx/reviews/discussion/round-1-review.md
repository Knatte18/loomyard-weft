---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:94
    summary: D7 treats D5's foreign-box error as nothing typed, but after a paste-submit the sent text can sit in the box beside the operator's text, so D8 and D9's resume retry can type the resume line into the driver twice
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:100
    summary: D7 rests on every caller already retrying, but shuttle Start's skill load and prompt delivery tear the run down as died on ErrSessionBusy, and D6 puts both behind the new activity hold and foreign-box failure
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:73
    summary: D5 accepts a bare collapsed-paste placeholder before a guarded Enter but states no bound; an operator's own paste after a swallowed send gets submitted, and only R3 mentions it
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:99
    summary: D7's bound covers cycle phases only; an idle-phase notice held by the operator-activity hold retries every tick, never counts as a failure and never reaches State.Stuck
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:75
    summary: D5's confirmation names landed, exact match and other content, but not a capture with no readable box after the paste-submit, such as a slash menu opened by a pasted command
  - id: F6
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:97
    summary: D7 leaves open whether the foreign-box error wraps ErrSubmissionNotLanded, but D8 must tell it apart from a never-appeared failure that resumeParkedDriver deliberately never retries
  - id: F7
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:48
    summary: D2 says paste-buffer -p sends bracketed paste as fact, but tmux adds the bracket codes only when the pane's application has turned on bracketed-paste mode; no assumption or criterion pins that Claude Code does
  - id: F8
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:54
    summary: D3's unknown-readiness fallback does not say whether D4's unreadable-box hold applies there; D4 names judgeReadiness only, which the unknown path returns from before reading the box
  - id: F9
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:164
    summary: Acceptance criteria miss D3's fail-open on an activity-query error and D8's lyx loom resume refusal
  - id: F10
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:141
    summary: The spawn-observability constraint says one tmux invocation per write, but D3 adds a list-clients query to every reading, including every idle-wait poll
  - id: F11
    severity: NIT
    class: design
    location: _lyx/discussion/decision-record.md:111
    summary: D8 and D9 carry the operator's draft text into the refusal, batten's report and possibly the orch notice; the record does not say whether the notice typed into the orch pane quotes it
---

### [BLOCKING] A foreign-box error is not "nothing typed"

**Location:** `_lyx/discussion/decision-record.md:94` (D7), with D5 at line 77, D8 at line 104 and D9 at line 111.

**Issue:** D5 raises the foreign-box error after the paste-submit has run, when the box holds anything other than exactly the sent text or the placeholder.
That includes the sent text plus the operator's characters: after an Enter that Claude Code turned into a newline (R1), or in D5's no-Enter variant, where the operator's keys can land between the paste and the guarded Enter.
In both cases the sent text stays in the operator's box, and D5 forbids clearing it.
D7 still tells the orch watcher to treat the error as "nothing typed" and retry, and D8 and D9 make batten retry the resume at every `noticeProbe`.
Once the operator sends or clears their draft, the leftover text goes out with the draft, and the retry then types the same line again.
`resumeParkedDriver` (`internal/loomcli/driverresume.go:26-30`) refuses to retry a send that may have delivered, because "re-sending could type the line into the driver twice".
D9 overrides that guard and states no bound on the duplicate.
The same duplicate applies to every watcher step D7 lists, such as a notice, `/clear` or `/compact`.

**Fix:** Have D5 split the foreign-box outcome: a box that does not contain the sent text means nothing of ours landed and the write is retryable; a box that contains it beside other text means our text is in the operator's draft.
For the second case, state what each caller does so nothing is delivered twice.
For example, the watcher and batten wait until the box reads empty and then check for a turn start past the send's offset before re-sending, or the case surfaces as a non-retried error naming the pane.
Name the leftover-text outcome in Open risks.

### [MEDIUM] D7's "every caller already retries" is false for shuttle Start

**Location:** `_lyx/discussion/decision-record.md:100` (D7 Rationale), with D6 at line 84.

**Issue:** D6 puts the start prompt and the skill-load message behind D3's guard, including the new operator-activity hold, and behind D5's foreign-box failure.
`loadSkillsThenPrompt` (`internal/shuttleengine/run.go:562-604`) turns any skill-load or prompt failure other than a late-landed `ErrSubmissionNotLanded` into `abandonStartup(OutcomeDied, …)`, which kills the pane together with any draft the operator typed into it.
`lyx shuttle send` (`internal/shuttlecli/send.go:38`) also returns the error to its caller with no retry.
D7 lists neither, so the record does not say whether a held or foreign-box start delivery dies, waits or is retried.

**Fix:** Add the start delivery to D7 and decide its response: keep it as a died start, or wait or retry.
If it stays a died start, state that bound and add an acceptance criterion.
Correct the rationale's claim that every caller already has a queue or a retry loop.

### [MEDIUM] D5's placeholder acceptance has no stated bound

**Location:** `_lyx/discussion/decision-record.md:73` (D5).

**Issue:** Today `boxHoldsSentText` never treats a placeholder as the sent text (`internal/shuttleengine/run.go:1262-1263`).
D5 now sends a guarded Enter whenever the box holds exactly the placeholder.
That opens a path around the Enter guard: when our paste is swallowed or lands late and the operator then pastes their own long text, lyx submits the operator's paste.
D3's activity check runs once, before the write, and never before a guarded Enter.
The only mention is R3 under Open risks, and it calls the safeguard "high likelihood, not certainty" without saying what bounds it.
The rubric requires the bound in the Decision that opens the path.

**Fix:** Add a Bound line to D5 that names what can now be submitted and what limits it.
Tighten that limit where cheap: re-run the operator-activity check right before every guarded Enter, and accept the placeholder only within the send's own `submit_confirm_timeout_s` window.

### [MEDIUM] D7's bound leaves idle-phase notices retrying unseen

**Location:** `_lyx/discussion/decision-record.md:99` (D7 Bound).

**Issue:** The bound relies on `HandoffTimeout` marking a cycle stuck.
`deliverNotice` (`internal/orchengine/watcher.go:563-585`) runs in `PhaseIdle`, where no cycle is running.
It gates on the watcher's own `probeIdle`, which does not read operator activity.
So an operator active in the orch pane passes the probe, and each tick's `Send` then waits out the 60 s `send_ready_timeout_s` and fails `ErrSessionBusy`.
Under D7 that failure no longer counts toward `maxConsecutiveTickErrors`, and nothing writes `State.Stuck`.
The tick-failure count is the guard being downgraded, and for this path the replacement bound does not apply.

**Fix:** State the bound for non-cycle steps as well.
For example, a held notice records its hold reason in `State.Stuck` the way `logHeld` does, or the watcher's probe reads operator activity so the hold happens before the send's 60 s wait.

### [LOW] D5 omits the unreadable-box outcome after the paste-submit

**Location:** `_lyx/discussion/decision-record.md:75`.

**Issue:** Confirmation covers a turn start, an empty box, an exact match and "any other box content".
A capture that shows no box at all after the write is not covered.
R1 names one way to get there: a pasted slash command that opens the slash menu.
The plan writer cannot tell whether that case counts as landed, foreign or still pending.

**Fix:** Name the unreadable-box outcome in D5 and its action: keep reading within `submit_confirm_timeout_s`, and send no Enter while no box is readable.

### [LOW] The foreign-box error's identity is left open, but D8 depends on it

**Location:** `_lyx/discussion/decision-record.md:97`, with D8 at line 104.

**Issue:** D7 lets the plan wrap the foreign-box error as either `ErrSubmissionNotLanded` or `ErrSessionBusy`.
D8 refuses as `driver_busy` on the foreign-box error but must keep failing, without the retryable kind, on a never-appeared `ErrSubmissionNotLanded`.
`resumeParkedDriver` does not retry that failure on purpose.
If the error only wraps `ErrSubmissionNotLanded`, D8 cannot tell the two apart.

**Fix:** Decide that the foreign-box error can be identified on its own, as its own sentinel or type that also wraps the error gate and seat retries need.
Keep in step with F1's split.

### [LOW] Bracketed paste is stated as fact

**Location:** `_lyx/discussion/decision-record.md:48`.

**Issue:** `paste-buffer -p` adds the bracket codes only when the application in the pane has turned on bracketed-paste mode.
D2's rationale, R1 and the live check in the acceptance criteria all assume Claude Code turns it on, but no assumption names that.
Without bracketed paste there is no paste end before the Enter, and the collapsed-paste placeholder D5 accepts presumably never appears.

**Fix:** Add an Auto-mode assumption that Claude Code enables bracketed-paste mode.
Have the `llm`-tier check confirm it, for example by checking that a long paste collapses into the placeholder.

### [LOW] The unknown-readiness fallback's box bound is unstated

**Location:** `_lyx/discussion/decision-record.md:54` (D3), with D4 at line 66.

**Issue:** On an unknown reading, `sessionReadiness` returns before reading the box (`internal/shuttleengine/readiness.go:95-97`), and `busyReading` falls back to `IdleSession`.
For Claude, `IdleSession` already requires an on-screen empty box (`claudeengine/session.go:57-70`), so in practice the unreadable-box hold also applies on that path.
Neither D3 nor D4 says so.
D4 scopes the hold to `judgeReadiness`, and D3 lists "the input box reads empty" without saying what a missing box means.

**Fix:** State in D3 that a missing box fails the empty-box condition on both paths for a box-reading engine, or that the fallback's idle reading is the bound there.

### [LOW] Acceptance criteria miss two Decisions

**Location:** `_lyx/discussion/decision-record.md:164`.

**Issue:** No criterion covers D3's fail-open: an activity query that errors logs at Warn and does not hold the write.
That fail-open is a downgraded guard, and its behavior should be pinned.
The `driver_busy` criterion names only `lyx loom start`, though D8 also covers `lyx loom resume`.

**Fix:** Add a criterion for the fail-open, and name `lyx loom resume` in the `driver_busy` criterion.

### [LOW] The spawn-observability constraint undercounts tmux invocations

**Location:** `_lyx/discussion/decision-record.md:141`.

**Issue:** The constraint says no new OS process is spawned "beyond the one tmux invocation per write".
D3 also adds a `list-clients` activity query to every pre-write reading, and the idle wait repeats that reading on each poll.

**Fix:** Name both new invocations, the activity query per reading and the paste-submit per write, and state that both are logged.

### [NIT] Draft text crossing into the orch pane

**Location:** `_lyx/discussion/decision-record.md:111` (D9), with D8 at line 104.

**Issue:** D8's refusal carries the draft text, and D9's report names it.
The record does not say whether the one orch notice repeats it.
If it does, a fragment of the operator's draft from another pane gets typed into the orch session as part of a prompt.

**Fix:** State that the orch notice names the slug, the stall and the remedy, and does not quote the draft.

## Settled without a finding

- D3's fail-open on an activity-query error states what it lets through and what bounds it, in the Decision itself (line 60).
- D4's limit to box-reading engines is stated in D4.
- D6's dropped Escape: with D3's empty-box precondition, no autocomplete menu can be open over an empty box, and D4 holds a picker that hides the box. The rationale states why prompt suggestion no longer needs the Escape.
- D9 bounds the retry's rate (`noticeProbe`) and names what ends the stall. The missing deadline is a deliberate choice, recorded in R5.
- Scope against Decisions: every In item has a Decision, and no Decision touches an Out item.
- Relocation: no Decision or Open risk carries deliberation narrative that belongs in the support log.
- Names the record cites exist: `StartNotParkedKind` (`internal/shedrun/paths.go:38`), `ErrChildNotParked` and the `noticeProbe` retry (`internal/battenshed/innerrun.go:806-808`).
