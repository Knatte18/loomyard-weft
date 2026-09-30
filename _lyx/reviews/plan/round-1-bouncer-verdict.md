---
verdict: BLOCKING
rationale: "All seven round-1 findings (F1-F7) are fixed in the plan as it stands, but card 8 persists `resuming` and increments CycleCount after `/clear` and only then renders and Sends the resume prompt; a render or Send error there is logged by card 9's loop and never retried, since D10's landed check runs only on a watcher's first tick, so the cleared session sits with no resume prompt until the timeout, against D10's rationale"
---

# Plan-Review round 1 — judge verdict

## The round-1 report

The report raised seven findings.
The fixer's edits landed before this judgment, so each one was checked against the plan as it now stands.

- **F1 (BLOCKING), return to `idle`:** resolved.
  Card 8 now has a return-to-idle rule: every completed resume, resume timeout and handoff abort persists `LastInjectionOffset` as the cursor read through and forgets the in-memory newest event.
  A resume timeout also marks the reading unknown.
  Card 6's `LastInjectionOffset` and the overview's `events-cursor` decision were updated to match, and the four test cases the report asked for are in card 8's list.
- **F2 (MEDIUM), smoke threshold:** resolved.
  Card 14 step 1 seeds a threshold no smoke session can reach and says why.
  This departs from the acceptance bullet's literal "with a low threshold".
  I ratify the departure: the bullet's purpose is a cycle driven through `lyx orch cycle`, a low threshold makes that cycle count non-deterministic, and card 8's unit cases cover the threshold-triggered path.
- **F3 (LOW), restart tests versus the `clearing` rule:** resolved.
  The landed/not-landed pair is scoped to `handoff-requested` and `resuming`, and a restart-in-`clearing` case is added.
- **F4 (LOW), liveness probe starving a watcher:** resolved by `Run`'s bounded lock retry and its test case.
- **F5 (LOW), untested Agent-tool override:** resolved by the pure `orchShuttleConfig` and its `cli_test.go` case.
  I agree with the report's focus verdict that D16 and the Goal license the override:
  an operator's orchestrator session uses subagents, and `claudeengine/settings.go` denies the `Agent` tool whenever `ClaudeDenyAgentTool` is set.
- **F6 (LOW), card 4's ImpactSummary:** resolved; it names `lyx fabric clone`'s reconcile and the registry-derived clone-config test.
- **F7 (NIT), card 12's ImpactSummary:** resolved.

The report has no `## Focus departures` section.
Its focus verdicts on D13 versus `session-cycler-capability`, and on granularity, hold:
the optional capability keeps `/clear` grammar in `claudeengine`, which is D13's stated rationale, and the overview gives a grounded reason for not widening `Engine`.

Spot checks against the code found the plan's references accurate:
`readEventsFrom`, `pollEventsTick`, `FindRun`, `requireLiveStrand`, `playInputs`, `requireReadyAgentPane`, `gateCaretMarker`, `ReadyFooterFixture`, `AuditForks`, `PrimeName`, `RefuseUnlessPrime`, `reedDriverPaneProbe`, `runnerDriverStarter`, `AttachArgv`, `proc.Detach` and `TryAcquireWriteLock` all exist.
`orch` sorts between `models` and `reed` in `configreg.Modules()`.

## New finding

### [BLOCKING] A failed resume injection is never retried, so a cleared session can sit with no resume prompt

**Location:** `_lyx/plan/08-orch-cycle-machine.md`, the `clearing` and `resuming` paragraphs and the restart-rule paragraph, together with `_lyx/plan/09-orch-watch-loop.md`'s handling of `Tick` errors.

**Issue:**
Card 8's `clearing` phase persists `resuming`, increments `CycleCount`, then `Send`s `RenderResumePrompt`.
By then `/clear` has already been typed.
If that render or `Send` returns an error, card 9 logs it at `Warn` and keeps looping.
The persisted phase is `resuming`, which only waits for a turn end past its entry offset.
Nothing on the pane will produce one, because the session is cleared and was never prompted.
D10's landed check ("an idle pane with no such turn end means it did not, so re-type the phase's stencil") runs only on the first tick of a restarted watcher.
The same watcher therefore never re-sends, and after `HandoffTimeout()` it returns to `idle` with an abort recorded, leaving a blank session.
D10's rationale names exactly this outcome as one to prevent: a cleared session stranded with no resume prompt.

Both triggers are plausible:

- `Runner.Send` runs `requireReadyAgentPane` and `sendVerified` (`internal/shuttleengine/run.go:558`), and either can fail against a pane that has just been cleared.
  That is most likely on the path card 8 itself covers: the idle probe never passes after `/clear`, and the resume prompt is sent on timeout into a pane the probe could not read as idle.
- `RenderResumePrompt` rejects a multi-line operator override (card 5).
  Rendered after `/clear`, that failure strands the session on every cycle, when rendering first would have aborted the cycle before `/clear` ran.

None of card 8's test cases makes `Send` or `ClearSession` fail, so the list does not catch this.

**Fix:**
- Render the resume prompt in `handoff-requested`, before persisting `clearing` and calling `ClearSession`.
  A render error then aborts to `idle` with no `/clear`, the same way a handoff timeout does.
- When an injection in a non-idle phase returns an error, have later ticks run D10's landed check for that phase rather than waiting passively.
  For example, persist a flag that the phase's injection is unconfirmed, and clear it once `Send` succeeds or a turn end past the entry offset is read.
  A turn end past the entry offset, or a failing `SessionIdle` probe, means the injection landed.
  An idle pane with no such turn end means it did not, so re-send once per tick, still bounded by the phase timeout.
- Add card 8 cases:
  - a `Send` error on the resume prompt, then an idle pane, leads to exactly one re-send on a later tick;
  - a `Send` error whose text did land (a turn end follows) leads to no re-send;
  - a failing resume render leads to an abort with no `ClearSession` call.
