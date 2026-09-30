---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    location: _lyx/plan/08-orch-cycle-machine.md (confirmation-rule bullets, restart-rule paragraph, test list)
    summary: Card 8's landed check reads any failing SessionIdle probe as proof an injection landed, so a resume Send that errors after a clearing timeout is never re-sent and the cleared session is stranded
---

# Plan-Review round 3

The plan was read in full (overview and cards 1–14) against `_lyx/discussion/decision-record.md`, with this round's focus on card 8's landed check for an unconfirmed injection.
Every Decision D1–D16 and every Constraint is carried by some card, and no card introduces unlicensed scope.
The verify section runs every targeted package, including under `-tags integration`, and only vets `smoke`.
Code read for this round: `Runner.Send` (`internal/shuttleengine/run.go:558`) runs `requireReadyAgentPane` before `sendVerified`, and `requireReadyAgentPane` (`run.go:617`) fails unless a capture classifies as `StartupReady`.
It also fails when all three capture attempts error.
Card 3's `IdleSession` fails on a running-turn hint, but also on a permission or select dialog, on a draft, and on any capture it cannot frame as an empty input box.

### [BLOCKING] A failing idle probe is read as proof the injection landed

**Location:** `_lyx/plan/08-orch-cycle-machine.md`.
The first bullet of the confirmation rule reads "a turn end past the entry offset, or a failing `SessionIdle` probe (a turn in progress), means the injection landed, so mark it confirmed and keep waiting".
Also affected: the restart-rule paragraph ("this same check, run on the first tick … whatever `PhaseInjected` says") and the test list.

**Issue:**
D10's restart rule counts "a turn end after the injection, or a turn in progress on the pane" as evidence of landing.
`SessionIdle` returning false is not that signal: card 3 makes it fail for a permission prompt, a select dialog, a draft and any unframable capture as well as for a running turn.
Card 8 then treats every one of those as a landed injection and sets `PhaseInjected` true.
Once the flag is true, no later tick runs the check or re-sends.

The case the focus names follows this path exactly.
Card 8 specifies that `clearing` moves to `resuming` on timeout even when the idle probe never passed ("the idle probe never passing after `/clear` → resume prompt after the timeout").
In that state the pane is by definition not readable as idle.
So `requireReadyAgentPane` in `Runner.Send` is likely to fail too, for example on a dialog that is not `StartupReady` or on capture errors.
The resume `Send` errors, and `PhaseInjected` stays false.
On the next tick the landed check probes, gets the same `false` it got through the whole `clearing` wait, and marks the resume prompt landed.
If the pane later reads idle with still no turn end, nothing re-sends, because the flag is already true.
`resuming` ends in a timeout abort, and the cleared session never received its resume prompt.
D10's rationale exists to prevent exactly that outcome ("strand a cleared session with no resume prompt"), and D11's guarantee cannot help: `/clear` has already run.

The same over-confirmation hits `handoff-requested` when a permission prompt sits on the pane at send time.
That case is safe, since D11 aborts with no `/clear`, but it throws away the re-send the confirmation rule was added to provide.

The restart path has a related gap.
The first tick of a restarted watcher runs the check "whatever `PhaseInjected` says", but a loaded flag that is already true survives a probe that fails for a non-turn reason.
So the watcher never reaches D10's "an idle pane with no such turn end means it did not, so re-type" on a later tick either.

The test list does not drive this path.
Its only unconfirmed-resume cases are "a `Send` error … then an idle pane with no turn end → one re-send" and "a `Send` error … whose text did land (a turn end follows) → no re-send".
Neither puts a failing probe between the error and the idle pane, so an implementation of the current text passes both while stranding the session.

**Fix:**
- Only a turn end past the entry offset confirms an injection.
  A failing `SessionIdle` probe means "keep waiting, flag unchanged", and a later tick re-checks.
  Once the pane reads idle with still no turn end, that tick re-sends, at most once per tick and bounded by the phase timeout.
  A turn that really is in progress still confirms, through its own turn end, so this gives the same result as D10's "turn in progress → keep waiting" without needing a positive running-turn probe.
  It also keeps `SessionCycler` unchanged.
- A restarted watcher loading a non-idle phase whose `PhaseStrand` equals `Strand` saves `PhaseInjected` false before its first tick's check.
  The check then keeps running on every later tick until a turn end confirms or an idle pane triggers the re-send, rather than stopping after one probe.
  `clearing` is unaffected: it re-calls `ClearSession` on restart as now.
- Reword card 6's `PhaseInjected` line so it names a turn end past the entry offset as the landed evidence.
- Add card 8 cases:
  - a resume `Send` error after a `clearing` timeout, with the probe still failing on the next tick and then passing with no turn end, leads to no re-send while it fails and exactly one re-send of `PendingResume` once it passes;
  - a resume `Send` error while a turn is in progress (probe failing), followed by that turn's end, leads to no re-send and `idle` at that turn end;
  - a handoff `Send` error behind a failing probe that later passes with no turn end leads to exactly one re-send naming the same `PendingHandoff`;
  - a restart in `resuming` with `PhaseInjected` true, a failing probe on the first tick and an idle pane with no turn end on the next, leads to exactly one re-send.
