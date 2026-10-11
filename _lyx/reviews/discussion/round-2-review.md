---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:123
    summary: D8 claims D5's in-draft record dedupes a rerun of the resume line, but each resume attempt builds a different line, so a same-text record never matches it
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:77
    summary: D5's not-landed outcome after a no-box window may leave the sent text in the box, yet only the in-draft failure gets a pending-send record, and the watcher and seat notices retry not-landed sends
  - id: F3
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:87
    summary: D5's placeholder acceptance is not tied to a sent text that can collapse, and its activity bound covers only the last operator_quiet_s of a submit window six times as long
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:110
    summary: D7 and R7 say a start dies only after a hold waits out send_ready_timeout_s, but a foreign-box failure after the paste kills the pane and the operator's draft at once
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:113
    summary: D7 records a held idle-phase notice as a StuckByHold reason, which the watcher's own passing idle probe clears on the next tick before the send holds again, so the reason flaps rather than staying until a send passes
---

### [MEDIUM] D8's dedupe claim rests on a resume line that changes every attempt

**Location:** `_lyx/discussion/decision-record.md:123` (D8), with D5's record at line 82.

**Issue:** D5's pending-send record dedupes "the next send of the same text to that strand".
D8 relies on it: "D5's in-draft record covers a rerun that reaches the send first."
The resume line is not stable across attempts.
`resumeParkedDriver` (`internal/loomcli/driverresume.go`) builds a fresh `reportPath` on every call through `driverReportPath(c.location, runID, time.Now, newDriverReportRand())`, and `driverResumeLine` (`internal/loomcli/driverresumeline.go`) embeds that path in the text.
Each batten retry is a new `lyx loom start` process, so every rerun carries a different timestamp and random suffix, the record never matches, and a rerun that reaches the send before the driver's re-step removes the park marker types a second resume line.
The premise is false for the one caller the focus names, and a plan writer following D5 literally builds a text-keyed record that cannot serve D8.

**Fix:** Key the pending-send record on something D8's caller can hold stable, or make D8 state how it qualifies: either the resume branch reuses the pending record's text (and report path) on a rerun, or the record is keyed per strand regardless of text with a caller-supplied identity, or D8 drops the claim and states the residual double-resume window as an open risk.

### [MEDIUM] Not-landed after a no-box window is a may-have-landed outcome without a record

**Location:** `_lyx/discussion/decision-record.md:77` (D5), with D7 at line 107 onward.

**Issue:** D5 fails a send as `ErrSubmissionNotLanded` when the box stays unreadable through the confirm window after the paste-submit, and names the cause itself: "a menu a pasted slash command opened, a redraw".
In that outcome our text was pasted and may still sit in the box under the menu, or be submitted later when the operator closes the menu with Enter.
Only the in-draft failure gets D5's pending-send record.
The callers still retry this outcome: `noticeSender.deliver` in `internal/seatengine/notice.go` retries `ErrSubmissionNotLanded` like `ErrSessionBusy`, and the orch watcher's `deliverNotice` (`internal/orchengine/watcher.go`) leaves the notice file in place on any send error, so the next tick types it again.
D4 holds the retry while no box shows, and a box holding our text holds it as non-empty, but once the operator submits that box, the box reads empty and the retry types the line a second time; for `/compact` or a notice that is a duplicate turn.
So the focus question, whether every may-have-landed outcome is kept apart from nothing-landed, is answered no for this outcome.
`resumeParkedDriver` and `Start` are safe here: the first refuses to retry it (D8) and the second checks for a late turn start.

**Fix:** Have D5 write the pending-send record for every outcome after the paste-submit that is not confirmed landed, not only for the in-draft failure, so the next same-text send dedupes by a turn start past the offset (and, for a slash command, by whatever confirmation D6 uses); or state that not-landed is never retried by the watcher and seat notices and route it as D8 does.

### [LOW] Placeholder acceptance is broader than the text it stands in for

**Location:** `_lyx/discussion/decision-record.md:87` (D5's bound).

**Issue:** D5 sends a guarded Enter into a box holding exactly Claude Code's collapsed-paste placeholder, whatever was sent.
A short sent text never collapses, so for it a placeholder cannot be ours, yet the decision accepts it.
`claudeengine`'s `pastePlaceholderShape` (`internal/shuttleengine/claudeengine/session.go`) already reads `[Pasted text #<n> +<m> lines]`, so the line count is available to match against the sent text.
The bound also says the fresh activity check holds an Enter after "a paste through an attached client within `operator_quiet_s`" (default 5 s), while the window it runs in, `submit_confirm_timeout_s`, defaults to 30 s (`defaultSubmitConfirmTimeoutS` in `internal/shuttleengine/send.go`); an operator paste between 5 and 30 s old is not covered, and the decision does not say so.
The rubric's attack-surface question wants what the new acceptance lets through stated with its bound; the stated bound overstates it.

**Fix:** Accept the placeholder only for a sent text the engine would collapse, and when the placeholder carries a line count, only when it matches the sent text's; and state in the bound that the activity check covers the last `operator_quiet_s` of the window, not the whole window.

### [LOW] A foreign-box failure at start dies without the wait D7 and R7 describe

**Location:** `_lyx/discussion/decision-record.md:110` (D7), `_lyx/discussion/decision-record.md:184` (R7).

**Issue:** D7 says `Start` keeps "a held or foreign-box failure" a died start and explains it with "the hold first waits out `send_ready_timeout_s`".
That wait applies to the pre-write hold only.
D5's foreign-box failures come after the paste-submit and return at once, and `loadSkillsThenPrompt` (`internal/shuttleengine/run.go`) turns any non-not-landed delivery error into `abandonStartup(OutcomeDied, …)`, which kills the pane.
An operator who types into a fresh pane in the gap or the confirm window loses the draft immediately, not after `send_ready_timeout_s`.
R7 names only the held-past-timeout case.
The focus asked whether the record states what a held or foreign-box start delivery does: it states the outcome (died start), but misstates its timing for the foreign-box case.

**Fix:** Split D7's Start bullet into the held case (waits, then dies) and the foreign-box case (dies at once, killing the pane with the operator's draft), and widen R7 to cover the second.

### [LOW] A send hold recorded as StuckByHold is cleared by the next passing probe

**Location:** `_lyx/discussion/decision-record.md:113` (D7's bound).

**Issue:** D7 records an idle-phase notice's send hold "in `State.Stuck` as a probe hold does (`StuckByHold`), and a send that passes clears it".
`deliverNotice` calls `probeIdle` before every send, and `probeIdle` (`internal/orchengine/watcher.go`) clears and saves any Stuck with `StuckByHold` set when its probe passes.
The watcher's probe does not read operator activity, so with the operator typing or a draft that the probe reads idle, every tick clears the reason, then the send holds and writes it back: two state writes per tick and a reason that flaps in `lyx orch status`, rather than one held until a send passes.
The surfacing the focus asks about holds, but not in the form D7 states.

**Fix:** State that a send-hold reason is recorded so the probe's pass does not clear it (its own marker beside `StuckByHold`, or cleared only by a passing send), or accept the flap and say so.

## Focus verdicts not raised as findings

- Focus 2 (held or foreign-box start delivery): D7's fourth bullet and R7 state that both end as a died start, so the record does state what happens; only the timing gap in F4 remains.
- Focus 3 (placeholder bound): D5's bound does state a fresh activity check before each guarded Enter, not only D3's one check before the write; the residual is F3.
- Focus 4 (repeatedly held idle-phase notice): D7 surfaces it through `State.Stuck` and leaves the retry unbounded, which fits a notice waiting on the operator's own draft in the orch pane; not a defect beyond F5.
