---
verdict: BLOCKING
rationale: "Round 2's F1-F3 are all fixed in cards 6 and 8, but card 8's new landed check counts any failing SessionIdle probe as proof the injection landed. After a clearing timeout, the probe was already failing before the resume Send errored, so the resume prompt is marked landed and never re-sent. That strands a cleared session, the harm D10 exists to prevent, on the path F1 called most likely."
---

# Plan-Review round 2 — judgment

The review report has no `## Focus departures` section, so there is nothing to ratify.

## Round 2 findings: all three fixed

- **F1 (BLOCKING):** fixed, apart from the residual gap below.
  In `handoff-requested`, card 8 now renders `RenderResumePrompt` before `/clear`, and a render error aborts with no `ClearSession`.
  `PendingResume` is persisted and sent verbatim.
  `PhaseInjected` is saved false before each side effect, and every tick in an unconfirmed phase runs the landed check.
  The restart rules reuse the same check, and the test list gains every case the review asked for.
- **F2 (MEDIUM):** fixed.
  `clearing` re-calls `ClearSession` until the clear is confirmed.
  `CycleCount` increments only when the phase leaves `clearing` with a confirmed clear, and card 6 now defines the field by that rule.
  Tests cover both a clear that recovers and one that fails until the timeout.
- **F3 (LOW):** fixed.
  The handoff instruction is rendered before state is persisted, and a render error leaves the state and the cycle request untouched.
  A test covers this.

Coverage of the decision record, the verify section and the granularity checks is unchanged from round 1.
None of those raises a finding.

## Remaining finding: a failing idle probe is read as proof the injection landed

**Location:** `_lyx/plan/08-orch-cycle-machine.md`.
The first bullet of the confirmation rule reads "a turn end past the entry offset, or a failing `SessionIdle` probe (a turn in progress), means the injection landed, so mark it confirmed".
The test list is also affected.

D10's restart rule counts only "a turn in progress on the pane" as evidence of landing.
A failing `SessionIdle` probe is a weaker signal.
Card 3's classifier also fails on a permission prompt, on a select dialog and on any capture it cannot frame as an idle box.
Card 8 itself names such a state: the case "the idle probe never passing after `/clear` → resume prompt after the timeout".

On that path, `clearing` times out with the probe still failing and moves to `resuming`.
If the resume `Send` then errors (for example, a capture error in `requireReadyAgentPane`, `internal/shuttleengine/run.go:613`), the next tick's landed check sees the same failing probe.
It marks `PhaseInjected` true, and once the flag is true no later tick re-checks or re-sends.
The cleared session never receives its resume prompt, and `resuming` ends in a timeout abort.
That is D10's "strand a cleared session with no resume prompt", in the scenario round 2's F1 named as most likely.
The same over-confirmation applies to the handoff instruction behind a permission prompt.
It is safe there (D11 aborts), but it wastes the retry that F1 added.

**Fix:**
- Only a turn end past the entry offset confirms an injection.
- A failing probe means "keep waiting, flag unchanged".
  A later tick then re-checks, and it re-sends once the pane reads idle with still no turn end.
  A turn really in progress still confirms through its turn end.
- If a positive in-progress signal is wanted, use card 3's running-turn hint ("esc to interrupt"), not the negation of the whole idle classification.
- Add card 8 tests:
  - a resume `Send` error while the probe fails (say, a pane that is not framed as idle), then a later idle pane with no turn end, leads to exactly one re-send;
  - the same while a turn is in progress, followed by that turn's end, leads to no re-send.
