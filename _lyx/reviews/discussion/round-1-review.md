---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md#D10
    summary: The clearing and resuming phases have no exit bound, resuming has no exit condition at all, and the startup-ready wait cannot tell whether /clear landed
  - id: F2
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md#D10
    summary: A non-idle persisted phase left by stop or strand death carries into the fresh session that start launches, and the record does not decide what happens
  - id: F3
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md#D6
    summary: D6 picks the newest file in the handoffs directory, which can be a partial or aborted handoff that D11 left behind
  - id: F4
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md#D7
    summary: Between /clear and the first post-resume turn end the newest Stop event still names the pre-clear transcript, so a usage reading taken then is stale and above threshold
  - id: F5
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md#D9
    summary: The idle rule cannot see a turn in progress, because the events file has no turn-start signal and an input box that is present and empty also describes a working session
  - id: F6
    severity: LOW
    location: _lyx/discussion/decision-record.md#D10
    summary: D10 and Open risks call the handoff focus a focus line, which reads as a second input line against D13's single-line rule
  - id: F7
    severity: LOW
    location: _lyx/discussion/decision-record.md#open-risks
    summary: Open risks does not state that the margin between the threshold and auto-compaction is consumed by operator turns before idle qualifies plus the handoff turn
  - id: F8
    severity: LOW
    location: _lyx/discussion/decision-record.md#acceptance-criteria
    summary: The smoke test verifies SendMessage address stability across /clear but not background-task survival, the other unverified premise the cycle depends on
---

Evidence read beyond the two artifacts and the writer's stencil:
`internal/shuttleengine/engine.go` (the `EventKind` set),
`internal/shuttleengine/claudeengine/settings.go` (`buildSettings`: the events file is written by a `Stop` hook and by a `PreToolUse` hook on `AskUserQuestion`, nothing else),
and `internal/shuttleengine/claudeengine/startup.go` (`Startup` classifies ready on the input marker `❯` or the `shortcuts` footer).

### [MEDIUM] Clearing and resuming have no bound, resuming has no exit, and startup-ready does not prove `/clear` landed

**Location:** `decision-record.md` D10, D11

**Issue:** D11 bounds only `handoff-requested`, with a timeout and an abort.
D10's `clearing` waits for "the provider's startup-ready classification", and `resuming` has no stated exit to `idle` at all: the text says what to type, not what ends the phase.
The startup-ready wait is also not evidence of anything here.
`claudeengine.Startup` classifies ready on the `❯` input marker or the `shortcuts` footer, which a healthy pre-clear pane already shows, so the wait passes whether or not `/clear` took effect.
The same fact means a restarted watcher in `clearing` cannot tell from that classification whether `/clear` landed, and D10's "never re-injects a step whose effect is already observable" gives the plan writer no per-phase rule to implement.
Neither phase has a timeout, so a session that never settles leaves the watcher waiting indefinitely.

**Fix:** Decide each phase's exit, bound and restart rule in D10:
`clearing` needs no proof that `/clear` landed, since the handoff is complete and a repeated `/clear` loses nothing;
it types the choreography (again on restart), then moves to `resuming` once the input-box probe shows an idle session or a bounded wait expires, because a resume prompt in an uncleared session is harmless.
`resuming` ends at the first turn end newer than the resume injection, bounded by the handoff timeout, after which the watcher returns to `idle` and records an abort reason.
On restart in `resuming`, a turn end newer than the phase entry means go `idle`, an in-progress turn means keep waiting, and an idle pane means re-type the resume prompt.

### [MEDIUM] A stale non-idle phase carries into a fresh session

**Location:** `decision-record.md` D5, D10, D14

**Issue:** D10 persists the phase in `.lyx/orch/state.json` and says a restarted watcher resumes from it.
D14's `stop` removes the strand mid-cycle and the watcher "exits on its own", and D4 has the watcher record the strand's death, but neither says the phase is reset.
D5's dead-strand branch then launches a fresh session and spawns a watcher, which by D10 resumes the old phase into the new session: typing `/clear` into it from `clearing`, or a resume prompt from `resuming`, on top of D6's own start prompt.
The record does not decide this.

**Fix:** Bind the persisted phase to the strand it belongs to.
Phase resumption applies only when a watcher restarts against the same live strand, which is D5's live-strand, no-watcher branch.
A fresh launch, D5's dead-or-absent branch, resets the phase to `idle` and records the abandoned phase as the last abort reason.

### [MEDIUM] D6 can resume from a partial or aborted handoff

**Location:** `decision-record.md` D6, D11

**Issue:** D11's abort leaves whatever handoff file exists in place:
a file written for a turn that never ended, or one that is partial because the timeout struck mid-write.
D6's second source is "the newest file in the prime's `.lyx/orch/handoffs/`", which picks such a file on the next fresh launch whenever the aborted attempt was the latest.
The record has no rule separating completed handoffs from aborted ones.

**Fix:** Record the handoff path in orch state only when `handoff-requested` completes by D11's own test (file exists, is non-empty, turn ended), and make D6's second source that recorded path when the file still exists.
D14's "last handoff path" is then the same field.

### [MEDIUM] A pre-clear usage reading can trigger a second cycle right after resume

**Location:** `decision-record.md` D7, D8, D10

**Issue:** D7 locates the transcript from what the Stop hook records.
`/clear` produces no Stop event, so between `/clear` and the resume turn's end the newest Stop event still names the pre-clear transcript, whose last main-chain assistant message is the handoff turn, above threshold by construction.
If `resuming` hands back to `idle` before the resume turn ends (D10 does not say when it does, see F1), a usage check in that window reads the stale value and can start a second cycle immediately.
The record does not say which turn end a usage reading belongs to.

**Fix:** State in D7/D8 that a usage reading is taken from the transcript named by the turn end being evaluated, that only a turn end newer than the last injection is evaluated, and that no turn end from before `/clear` is ever evaluated after it.
Combined with F1's `resuming` exit, the first post-cycle reading then comes from the resume turn in the new transcript.

### [MEDIUM] The idle rule cannot see a turn in progress

**Location:** `decision-record.md` D9

**Issue:** The events file holds Stop events and `AskUserQuestion` asks only (`buildSettings`); nothing records a turn starting.
So after a turn end, an operator prompt or a background task's completion starts a new turn that adds no event until it ends, and D9's "no newer event for 30s" is satisfied while that turn runs.
The only remaining guard is the input-box probe, specified as "input box present and empty", but Claude Code draws the input box while a turn is running too, and `Startup` treats that `❯` marker as ready.
The watcher can therefore inject the handoff into a running turn, the case the grace period's rationale says it keeps out of.

**Fix:** Specify the probe as classifying an idle session: input box present and empty, and no turn in progress on screen.
Add a pane-capture fixture for a working session to the input-box probe's acceptance tests.

### [LOW] "Focus line" reads as a second line of input

**Location:** `decision-record.md` D10 step 1, Open risks first bullet

**Issue:** D13 requires every injected stencil to render to a single line, and D10 describes the handoff injection as `/scribe:handoff <path>` "with a focus line".
"Line" can be read as a separate line after the command, which the single-line send path cannot carry.

**Fix:** Say the focus instruction follows the path on the same line, and call it focus text in both places.

### [LOW] The compaction margin is not stated as a risk

**Location:** `decision-record.md` D8, Open risks

**Issue:** Usage is checked at a turn end, but injection waits for D9's grace and probe, and the operator's further turns plus the handoff turn itself all add tokens before `/clear`.
D8's rationale places 150000 below auto-compaction on a 200k window, but Open risks' last bullet covers only a threshold set above that point, not a default whose margin gets consumed.
The outcome is benign, since a compaction lowers usage and cycling resumes later, but the plan writer and operator should see it.

**Fix:** Extend the last Open risks bullet: on a 200k window the margin between the default and auto-compaction is consumed by operator turns before idle qualifies and by the handoff turn, so a busy session can compact first.

### [LOW] Background-task survival across `/clear` has no verification step

**Location:** `decision-record.md` Open risks, Acceptance criteria; `support-log.md` Q13

**Issue:** Open risk 3 gets a smoke-test verification and a recorded outcome; open risk 1, whether background tasks and their completion notifications survive `/clear`, gets neither.
The orchestrator's standing background watchers are D9's own premise, and the resumed session's handling of them depends on the answer: re-arming duplicates them if they survived, and skipping re-arming loses them if they did not.

**Fix:** Have the smoke test start a background task before the cycle and record, in the module doc, whether it and its completion notification survive `/clear`, and point risk 1 and Q13 at that step.
