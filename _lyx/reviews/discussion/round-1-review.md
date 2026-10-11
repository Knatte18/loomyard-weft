---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:49
    summary: D4 equates a nil cause with never-ready, but a pane that dies during the skill-load turn also tears down with a nil cause after ready, and D4's rule would append a stale ready-screen tail; the cause-present rationale also overstates that every cause is a busy-send carrying a pane tail
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:39
    summary: D2's background line defines empty but does not state what D1 lets through, so the background's opening promise that lyx text never merges with an operator draft stays unqualified while a lone-% draft can now be merged
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:60
    summary: D5's premise that a bounded loop puts awaitStartup outside PATTERN-daemon-wakeups skips the background's own line that only a bounded wait ending within a minute is outside the rule, while startup_timeout_s ships at 90 s and each probe captures a pane with no backoff
  - id: F4
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:62
    summary: D6's gap leaves unstated whether liveness_every_n_polls is floored to 1 as Wait floors it, whether a probe exactly one gap after the last dismissal may dismiss, and whether a failed or empty play counts as a played dismissal; the claim that Wait's probe never meets the throttle depends on the first two
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:31
    summary: D1's bound says the operator-activity hold runs on every reading, but it runs on the pre-write guard and the readiness branch of the idle probe; the idle probe's pane-reading fallback calls IdleSession with no operator-activity hold
  - id: F6
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:77
    summary: D7 says the only case the cap governs is an advisor stuck mid-turn, but advisorWork.outstanding also holds true for a live advisor whose up line the sender never landed
---

### [MEDIUM] D4's nil-cause rule covers more than the never-ready case it names

**Location:** `_lyx/discussion/decision-record.md:49`, D4, and its Acceptance criterion "A never-ready start's error ends with the last lines of its last startup capture".

**Issue:** D4 keys the pane tail on "`abandonStartup` runs with no cause", glossed as "the provider never reached `StartupReady`".
The code has a nil-cause teardown after ready, too: `loadSkillsThenPrompt` in `internal/shuttleengine/run.go` calls `run.abandonStartup(OutcomeDied, nil)` when the pane dies during the skill-load turn.
`run.lastStartupCapture` then holds the ready screen captured before any skill was sent, so D4's rule as written appends that screen's last lines to a died-mid-load error, and it is the screen from before the death.
The second sentence's premise is overbroad as well.
Cause-carrying teardowns include the three reed-status failures in `awaitStartup` (`wait.go`), the skill-loader lookup error, the prompt-offset persist error and a non-busy prompt-delivery error, and none of these carries a pane tail.
"A busy-send cause already carries the pane's last lines" holds for the busy-send cause only.
The decision text and the acceptance criterion disagree on scope, and the plan writer has to guess which one to build.

**Fix:** State the trigger as the provider never reaching ready (the `awaitStartup` not-ready exits), not as a nil cause, or say outright that the died-during-skill-load teardown gets no tail, or gets a fresh capture.
Reword the cause-present sentence to cover the busy-send cause only, and either state that other causes get no tail on purpose or extend the tail to them.

### [MEDIUM] D2's background line omits what D1 lets through

**Location:** `_lyx/discussion/decision-record.md:39`, D2.

**Issue:** `pattern/PATTERN-pane-text-injection.md` opens with "Text lyx types into an agent pane never merges with, submits or wipes an operator's draft".
D1's bound concedes that "a real operator draft of exactly `%` typed outside reed's quiet window would read as empty and a paste would merge with it".
D2's line says only that Claude's reading treats a lone `%` as empty.
A reader of the background still sees the unqualified never-merges promise and learns about the exception only from the classifier.
That is the outcome D2's own Rationale says it prevents: "a narrowed guard is stated in the invariant's background, not left for a reader to find in the classifier".
The focus question, whether D1's bound is consistent with D2's invariant wording, settles here: D1 states the let-through, but D2 does not carry it into the invariant.

**Fix:** Have D2's added line also state the let-through: a draft consisting of exactly a lone `%`, typed outside `operator_quiet_s`, reads as empty and can be merged with.

### [MEDIUM] D5 misreads PATTERN-daemon-wakeups' scope

**Location:** `_lyx/discussion/decision-record.md:60`, D5 Rationale, and the `PATTERN-daemon-wakeups` line under Constraints.

**Issue:** D5 argues "the loop is bounded by `startup_timeout_s` and its tick cap, so `PATTERN-daemon-wakeups` does not apply".
The background (`pattern/PATTERN-daemon-wakeups.md`, "What counts") says "A bounded wait that ends within a minute is outside the rule".
`startup_timeout_s` ships at 90 in `internal/shuttleengine/template.yaml`, so after D5 `awaitStartup` can run for 90 s and spawn one reed status and one pane capture every second, with no backoff.
The rule's backoff clause targets exactly "a wake that spawns a process or captures a pane".
Whether a 90 s bounded wait falls inside the rule is open in the background: its definition of an unbounded wait, "no deadline of its own", excludes `awaitStartup`.
Even so, the Rationale rests on a premise the background does not grant.
This is filed MEDIUM rather than BLOCKING because no listed constraint is clearly breached.

**Fix:** Restate the premise against the background's actual wording.
For example: `awaitStartup` has its own deadline, so it is not an unbounded wait under the background's definition, even though its window can exceed a minute.
Alternatively, cap or back off the one-second cadence past the first minute.
Either way, the Constraints line should say which reading the plan follows.

### [LOW] D6's dismissal gap leaves three config- and boundary-dependent edges unstated

**Location:** `_lyx/discussion/decision-record.md:62`–`69`, D6.

**Issue:** The gap is written as `pollInterval(cfg) × liveness_every_n_polls`, but `Wait` and `awaitStartup` floor a non-positive multiple to 1.
D6 does not say whether its gap uses the floored multiple, so a configured 0 gives a zero gap that differs from Wait's real liveness spacing.
The premise "Wait's own startup probe already runs at the dismissal gap, so the throttle never holds a dismissal there" holds only if the gap uses the floored multiple.
It also needs a probe exactly one gap after the last dismissal to be allowed to dismiss.
Under an injected clock, Wait's liveness ticks land exactly `interval × N` apart, so a strict "within the gap" comparison would hold every second dismissal in Wait.
D6 also does not say whether a `playInputs` failure or an empty `TrustDismissSequence` counts as a played dismissal.
`checkLivenessTick` logs a failed play as non-fatal and goes on.
If a failed play starts the gap, a transient reed error costs a full gap before the next try; if it does not, the bound needs to say so.
The rest of the bound is complete: a probe inside the gap that sees a gate plays nothing and still classifies the startup window.
That probe skips only the key presses, and every negative answer still goes through `classifyStartupWindow`.
The Acceptance criterion covers a gate that never clears ("ends the start at the startup deadline as died").

**Fix:** State in D6 that the gap uses the multiple floored to 1, as Wait floors it.
State that a probe at or past one gap may dismiss, and whether only a successful, non-empty play starts the gap.

### [LOW] D1's bound overstates where the operator-activity hold runs

**Location:** `_lyx/discussion/decision-record.md:31`, D1 Bound.

**Issue:** "The operator-activity hold still runs on every reading" is broader than the code.
The hold runs on the pre-write guard (`send.go`) and on the readiness branch of the session idle probe (`session.go`).
The idle probe's fallback branch, taken when no run is found for the strand, returns `cycler.IdleSession(capture)` with no `operatorActivityHold`.
After D1, a lone-`%` reading there is idle with no operator check.
The bound's conclusion stands for writes behind the pre-write guard, but the sentence as written promises more than holds.

**Fix:** Narrow the clause to "the operator-activity hold still runs before every write behind the pre-write guard", or name the fallback idle probe as a reading the hold does not cover.

### [LOW] D7 names one cap-governed case where the predicate has two

**Location:** `_lyx/discussion/decision-record.md:77`, D7 Rationale.

**Issue:** D7 concludes that "the only case the cap governs is an advisor stuck mid-turn".
`advisorWork.outstanding` (`internal/seatengine/work.go`) also returns true for a live advisor whose up notice the sender has not landed (`!sender.wasSent(advisorUpNotice(run.name))`).
That reading persists for as long as delivery fails, with no mid-turn reading involved, so the cap also bounds it.
The 10-minute value still follows from the 9 measured turns (longest about 209 s, so about three times the longest), and the stuck-mid-turn case is bounded by the cap itself.
The premise sentence is the only defect.

**Fix:** Say that the cap governs any reading `outstanding` keeps true: an advisor stuck mid-turn, or one whose up line never lands.

### Relocation check

No finding.
The Rationale paragraphs carry the premises the plan writer needs to judge the bounds, and the Open risks and Auto-mode assumptions are short and each carries a constraint.
