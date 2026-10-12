---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: _lyx/discussion/decision-record.md (Decisions, "Added after Discussion (parent, 2026-10-12)")
    summary: The parent-added rewind-picker decision contradicts Scope Out, D6, D7's PATTERN rule, an Acceptance criterion and the support log's rejected alternative and Q4
    fix: Make the parent decision authoritative and reconcile the rest with it. Number it D8 under Decisions. In Scope, delete the Out bullet "Closing or detecting an open rewind picker or other dialog at resume" and add an In bullet naming picker detection and the closing Esc. Rewrite D6 so resume no longer "sends no Esc" and no longer tells a picker from a draft "without detecting the picker"; it detects the picker per D8 before the guarded send. Replace the Acceptance criterion ending "a pane with the picker open yields the guard's reason" with D8's outcome (picker closed with one Esc, input box re-checked, reported as a failure, nothing typed). Amend D7's PATTERN entry wording so the picker-closing Esc is a named exception with its own guard (see F2). In support-log.md, move the rejected alternative "Close a picker at resume with one Esc, or detect it from the pane" out of Rejected alternatives, record the parent override in the Interview, and mark Q4 as overridden by the parent with the new answer.
  - id: F2
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md (Decisions, "Added after Discussion (parent, 2026-10-12)")
    summary: The parent-added decision adds a new Esc path into a pane with no running turn but states no mechanism, seam, bound or acceptance criterion for it
    fix: In the reconciled D8, state (a) picker detection is a provider capability in claudeengine read from a pane capture, reached provider-neutrally like D3's running-turn reading; (b) which code path sends the closing Esc, given D3/D7 forbid an interrupt key outside the guarded interrupt and that guard refuses a pane with no running turn, so name a separate guarded close that sends exactly one Esc only into a pane the capability reads as the picker; (c) its bound, namely it sends at most one Esc per detection, only when the capture shows the picker, never selects an entry, and what happens when the re-check still shows no input box (report the failure and send nothing more); (d) for resume, whether the resume line is sent after a successful close or withheld; (e) which surfaces run the check (pause, resume, shuttle send, shuttle interrupt) and what each reports. Replace "Tests cover the picker being opened as a failure" with Acceptance criteria naming each behavior, namely picker detected, one Esc sent, input box re-checked, failure reported, nothing typed, and the re-check-fails case.
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md (D7, Bound; Acceptance criterion on Runner.Inject)
    summary: D7's Inject refusal matches only an input list equal to the interrupt sequence, so a two-Escape list, the exact rewind opener, or an Escape with a SettleMS still passes the guard
    fix: In D7, change the Inject refusal from equality with the engine's interrupt sequence to any input list containing a key of that sequence, sends nothing and returns the error naming lyx shuttle interrupt. Drop the dialog-dismissal carve-out from D7's Bound, since Runner.Inject has no production caller today, and state the new bound, namely Inject can deliver every key but the provider's interrupt key. Update the Inject Acceptance criterion to match, with rows for a lone Escape, two Escapes, and Escape among other keys, all refused.
  - id: F4
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md (D1; D5 no_running_turn; Open risks third bullet; Acceptance criterion on no_running_turn)
    summary: A no_running_turn session whose raw state still reads busy/turn is refused at resume by the pre-send Busy check, contradicting D5 and the criterion that it gets the resume line
    fix: In D1, drop resume's pre-send check from the uses of the raw-state predicate, and state that resume removes its separate Busy pre-check and lets the guarded send's readiness, which applies the turn-start hold, decide; D6 then reports a running turn by the guard's own reason. Keep the Acceptance criterion that a no_running_turn session gets the resume line, and add the released-hold case to it.
  - id: F5
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md (D3, Bound; Open risks second bullet)
    summary: D3's bound omits that the running-turn needle matches anywhere in the capture, so a transcript line quoting the hint lets an Esc through onto an idle prompt
    fix: Add to D3's Bound that a capture quoting the running-turn hint in its transcript reads as a running turn, so the guard lets an Esc through onto an idle prompt, at most one per settle time, so two such Escs land at least the settle time apart and open no picker. Alternatively decide that the new capability matches the hint only on the status line below the input box, and say so in D3.
---

### [BLOCKING] Parent-added picker decision contradicts the rest of the record

**Location:** `_lyx/discussion/decision-record.md`, Decisions, "Added after Discussion (parent, 2026-10-12)".
**Issue:** The added Decision says lyx detects an open rewind picker, closes it with one Esc, re-checks the input box and reports a failure, on pause, resume, shuttle send and interrupt.
The rest of the record still says the opposite.
Scope Out excludes "Closing or detecting an open rewind picker or other dialog at resume".
D6 says "Resume keeps the guarded paste-submit and sends no Esc" and tells a picker from a draft "without detecting the picker".
D7's PATTERN rule sends an interrupt key only through the guarded interrupt, which refuses a pane showing no running turn, so the picker-closing Esc breaks it.
The Acceptance criterion for a pre-change entry expects the guard's no-input-box reason from an open picker, not a close.
In `support-log.md`, Rejected alternatives still rejects "Close a picker at resume with one Esc, or detect it from the pane", and Q4 still answers "No".
`Plan-Write` reads only the decision record, so it gets two opposite instructions.

### [BLOCKING] Picker-closing Esc has no mechanism, seam or bound

**Location:** `_lyx/discussion/decision-record.md`, Decisions, "Added after Discussion (parent, 2026-10-12)".
**Issue:** The Decision adds an Esc into a pane with no running turn, which is new routing around D3's guard.
It doesn't say how the picker is detected, where that provider-specific reading lives under `PATTERN-shuttle-provider-seam`, or which code path sends the Esc.
It also doesn't say what bounds that path, what happens when the re-check still shows no input box, or whether resume then types its line.
The rubric's attack-surface question requires the Decision to say what the new edge can let through and what bounds it.
"Tests cover the picker being opened as a failure" isn't an Acceptance criterion the plan writer can act on.

### [MEDIUM] Inject's interrupt refusal matches only the exact sequence

**Location:** `_lyx/discussion/decision-record.md`, D7 Bound and the `Runner.Inject` Acceptance criterion.
**Issue:** `claudeengine`'s `InterruptSequence` is `[]PaneInput{{Key: "Escape"}}` (`internal/shuttleengine/claudeengine/startup.go:223`), and `PaneInput` carries a `SettleMS` field (`internal/shuttleengine/engine.go:53`).
An equality refusal stops only a lone bare Escape.
`[{Key: "Escape"}, {Key: "Escape"}]` passes, and that is the double-Esc this task exists to prevent.
`{Key: "Escape", SettleMS: 1}` also passes, and so does Escape followed by any key, which still interrupts the turn.
D7's own rule, that "sending an interrupt key to an agent pane goes only through shuttle's guarded interrupt", is therefore not what the refusal enforces.
The parent decision's "every Esc lyx sends must be guaranteed not to reach an idle prompt or to double up" is not enforced either.
`Runner.Inject` has no production caller (`grep` finds only its definition at `internal/shuttleengine/run.go:996`), so the dialog-dismissal carve-out covers no current use.
Focus item 1: D7 does state Inject's fate for the interrupt key, and an Acceptance criterion pins it, but the bound it states is too narrow; this finding is the verdict.

### [MEDIUM] A no_running_turn session can be refused at resume

**Location:** `_lyx/discussion/decision-record.md`, D1, D5 `no_running_turn`, Open risks third bullet, and the `no_running_turn` Acceptance criterion.
**Issue:** D1 has resume's pre-send check use the raw-state predicate.
Today that check is `pair.Busy` at `internal/pauseengine/resume.go:135`, which returns "the session reads busy" before any send.
Open risks says a `busy`/`turn` state whose turn-start hold released reaches the guard and lands in `no_running_turn`.
Nothing in that case changes the raw folded state before resume.
At resume the same session still reads a running turn, and the pre-check refuses it with no send.
D5 and the Acceptance criterion promise the opposite, that it "gets the resume line on resume".
The guarded send's readiness (`sessionReadiness` in `internal/shuttleengine/readiness.go`) applies the hold and would let the send through.

### [LOW] D3's bound omits the needle's whole-capture false positive

**Location:** `_lyx/discussion/decision-record.md`, D3 Bound and Open risks second bullet.
**Issue:** `claudeengine` documents that "a transcript line quoting \"esc to interrupt\" anywhere in the capture classifies as not idle, because the running-turn hint is matched over the whole capture" (`internal/shuttleengine/claudeengine/session.go:66`).
D3 builds the guard's capability from that needle.
An idle pane whose transcript quotes the hint therefore reads as a running turn, and the guard lets an Esc through onto the idle prompt.
D3's Bound names only the capture-to-key race, and Open risks names only the opposite failure, a reworded hint withholding every Esc.
The settle keeps two such Escs apart, so this is a missing bound, not a picker path.

## Focus directives settled without a finding

- Focus 2 (`still_busy` vs `interrupted`): consistent.
  D4 decides the second Esc from the state after the first settle, and decides the final list from one pane reading after the second settle.
  D5's `interrupted` and `still_busy` and the second-Esc Acceptance criterion all use those same two readings.
  D1 lists only where the state predicate applies and claims nothing about the final classification.
- Focus 3 (orch watcher ordering): D4 now has `handleHubEntry` consult the remembered first-Esc time before `settledWithoutInterrupt`, whose `entry.Sessions` GUID match (`internal/orchengine/pause.go:113`) would otherwise settle the entry.
  It states the bound for a restarted watcher, which settles the entry with no second Esc.
  Not a defect.
- Focus 4 (quoted reason): D6 and the criterion quote "no input box is on screen; a menu or picker may be open", which matches `readinessReasonNoInputBox` at `internal/shuttleengine/readiness.go:16` exactly.
  Not a defect.
