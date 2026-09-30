---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    location: _lyx/plan/08-orch-cycle-machine.md (clearing, resuming and restart-rule paragraphs; test list)
    summary: A render or Send error after a phase is persisted is never retried by the same watcher, and the resume prompt is rendered only after /clear, so a cleared session can sit with no resume prompt until the timeout
  - id: F2
    severity: MEDIUM
    location: _lyx/plan/08-orch-cycle-machine.md (handoff-requested and clearing paragraphs); _lyx/plan/06-orch-state.md (CycleCount)
    summary: A ClearSession error still leads to resuming and a CycleCount increment, although the clear never reached the pane and card 6 defines CycleCount as cycles that reached /clear
  - id: F3
    severity: LOW
    location: _lyx/plan/08-orch-cycle-machine.md (idle paragraph)
    summary: The handoff instruction is rendered after handoff-requested is persisted and the cycle request cleared, so a render error consumes the request and waits out a full handoff timeout on every qualifying idle point
---

# Plan-Review round 2

The plan was read in full (overview and cards 1–14) against `_lyx/discussion/decision-record.md`.
Every Decision D1–D16 and every Constraint is carried by some card, and no card introduces unlicensed scope.
The verify section runs every targeted package, with `-tags integration` included and `smoke` vetted rather than run.
Code references checked for this round's findings: `Runner.Send` (`internal/shuttleengine/run.go:558`) runs `validateSendText`, `FindRun`, `requireReadyAgentPane` and `sendVerified`, and `sendVerified` returns "the send was NOT delivered" after two unverified plays;
`playInputs` (the body of card 1's `ClearSession`) fails only on a reed `SendKey`/`SendText` error and verifies nothing.

### [BLOCKING] A failed injection is never retried by the same watcher, and the resume prompt is rendered after `/clear`

**Location:** `_lyx/plan/08-orch-cycle-machine.md` — the `clearing` paragraph ("persists `resuming`, increments `CycleCount`, then `Send`s `RenderResumePrompt`"), the `resuming` paragraph, the restart-rule paragraph ("D10's restart rules run once, on the first tick of a watcher…"), and the test list; together with `_lyx/plan/09-orch-watch-loop.md`'s "A `Tick` error is logged at `Warn` and the loop continues".

**Issue:**
Card 8 persists each phase before its side effect, as D10 requires, but says nothing about what later ticks do when that side effect errors.
The only place the landed check ("an idle pane with no such turn end means it did not, so re-type") runs is a restarted watcher's first tick.
A live watcher whose injection errored therefore just waits in the persisted phase.

For `resuming` this is the outcome D10's rationale names as one to prevent ("strand a cleared session with no resume prompt").
`/clear` has already been typed when `RenderResumePrompt` runs.
A render error (card 5 rejects a multi-line operator override) or a `Send` error (`requireReadyAgentPane` or `sendVerified` failing against a just-cleared pane) leaves `resuming` waiting for a turn end the cleared, unprompted session will never produce.
After `HandoffTimeout()` it returns to `idle` with an abort recorded and a blank context.
The `Send` failure is most likely on a path card 8 itself specifies: the idle probe never passes after `/clear`, and the resume prompt is sent on timeout into a pane the probe could not read as idle.
The render failure is deterministic: an override that renders to two lines strands the session on every cycle, when rendering before `/clear` would have aborted the cycle with the context intact.

The same passive wait applies to a `Send` error on the handoff instruction in `handoff-requested`.
That case is safe (D11 aborts with no `/clear`), but it wastes a full handoff timeout that one re-send would have saved.

None of card 8's test cases makes a render, `Send` or `ClearSession` fail, so the list does not pin any of this.

**Fix:**
- Render the resume prompt in `handoff-requested`, before persisting `clearing` and calling `ClearSession`.
  A render error then aborts to `idle` with no `/clear` and `LastAbortReason` naming the stencil, the same way a handoff timeout does.
  Persist the rendered text in state (a new `State` field in card 6, e.g. `PendingResume`), so the `resuming` send, its retries and a restarted watcher all send exactly the text checked before `/clear`.
- Persist, with each non-idle phase, whether its injection is confirmed (a new `State` field in card 6, e.g. `PhaseInjected`), written false before the side effect and true once `Send` or `ClearSession` returns nil.
  On every tick in a phase whose injection is unconfirmed, run D10's landed check for that phase, not only on a restarted watcher's first tick:
  for `handoff-requested` and `resuming`, a turn end past the entry offset or a failing `SessionIdle` probe means the text landed, so mark it confirmed and keep waiting;
  an idle pane with no such turn end means it did not, so re-send at most once per tick, still bounded by the phase timeout.
  For `clearing`, re-call `ClearSession` (D10 needs no landed check there).
  A restarted watcher treats its loaded phase as unconfirmed on the first tick, which is today's restart rule.
- An injection error is returned from `Tick` after the unconfirmed flag is saved, so card 9's loop logs it and counts it toward its cap.
- Add card 8 cases:
  - a `Send` error on the resume prompt, then an idle pane with no turn end, leads to exactly one re-send on the next tick;
  - a `Send` error on the resume prompt whose text did land (a turn end follows) leads to no re-send;
  - a `Send` error on the handoff instruction, then an idle pane, leads to one re-send of the same `PendingHandoff` path;
  - a failing resume render leads to an abort with no `ClearSession` call and the abort reason recorded.

### [MEDIUM] A `ClearSession` error still moves to `resuming` and counts as a cycle

**Location:** `_lyx/plan/08-orch-cycle-machine.md` — `handoff-requested` ("persist `clearing`, then call `ClearSession`") and `clearing` ("Either way it persists `resuming`, increments `CycleCount`"); `_lyx/plan/06-orch-state.md` — "`CycleCount` — cycles that reached `/clear`".

**Issue:**
When `ClearSession` errors, `/clear` never reached the pane, and the pre-clear pane still shows the idle box the handoff turn left.
On the next tick `clearing` sees a passing `SessionIdle` probe, persists `resuming`, increments `CycleCount`, and sends the resume prompt into the uncleared session.
D10 calls a resume prompt in an uncleared session harmless, so the session itself is safe.
But the counter card 6 defines as "cycles that reached `/clear`" now counts one that did not, and `status` (D14) reports a cycle that freed no context.
It also departs from D10 phase 2, which is "type the engine's clear-session choreography, then wait": a clear that errored was never typed, and D10's own restart rule for `clearing` is to re-type it.
The context stays over the threshold, so the next idle turn end starts another full handoff cycle, one handoff turn later than a retried `/clear` would have freed it.

**Fix:**
- Under F1's confirmed-injection flag, a `ClearSession` error leaves `clearing` unconfirmed, and later ticks re-call `ClearSession` until it returns nil or the phase times out.
- `clearing` moves to `resuming` only after a confirmed clear and a passing probe, or on the timeout (D10), whichever comes first.
- Increment `CycleCount` when leaving `clearing` with the clear confirmed, never on a timeout exit with no successful `ClearSession`.
  Incrementing at that transition keeps a restarted watcher's unconditional re-clear from counting the cycle twice.
- Add card 8 cases:
  - a `ClearSession` error, then success on the next tick, leads to one `resuming` transition and `CycleCount` 1;
  - a `ClearSession` that errors until the timeout leads to `resuming` with `CycleCount` unchanged.

### [LOW] The handoff instruction is rendered after the phase is persisted and the cycle request cleared

**Location:** `_lyx/plan/08-orch-cycle-machine.md` — the `idle` paragraph: "it persists `handoff-requested` (…), clears the cycle request, then `Send`s `RenderHandoffInstruction`."

**Issue:**
A render error (a multi-line or unreadable handoff stencil override, per card 5) surfaces only after `handoff-requested` is persisted and the operator's cycle request is deleted.
Nothing was typed, so the phase waits out the full handoff timeout (default 10 minutes) and aborts.
The next qualifying idle point repeats the same render failure, so a broken override turns the watcher into a loop of 10-minute phantom cycles, and `status` shows `handoff-requested` throughout.
A manual `lyx orch cycle` is silently consumed by the first of them.

**Fix:**
Render the handoff instruction before persisting `handoff-requested` or clearing the cycle request.
A render error leaves the state untouched and is returned from `Tick`, so card 9 logs it at `Warn` with the stencil named.
Add a card 8 case: a failing handoff render leads to no phase change, no `Send`, and the cycle request still present.
