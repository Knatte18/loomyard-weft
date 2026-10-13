---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md (D3, third paragraph; Scope In)
    summary: D3 takes twill-ness from what the Plan-Write spawn recorded, but no spawn record carries the fork-subagent authorization, and the attach branch rebuilds the spec from a fresh plan_producer read, so the stated guarantee that a config edit between spawn and gate changes nothing is false
    fix: In D3, replace the sentence beginning "Whether the spawn was twill is taken from what the Plan-Write spawn recorded" with a decision that names a persisted record, for example "The run record (shuttle's RunState) records the spec's ForkSubagents at spawn; the Plan-Write gate reads it from the run it judges, attached or spawned, and never from the spec composed for that Call." Add that run-record field to Scope In (internal/shuttleengine run record, tolerant of its absence on an older record, which reads as not twill) and add an Acceptance criterion - a twill-spawned Plan-Write run attached by a later Call whose plan_producer now reads single still gets the lens-review check, and the reverse does not. If the author instead accepts the fresh read on the attach branch, D3 must say so and drop the "changes nothing" claim.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md (D2 classification sentences; Acceptance criteria, claudeengine fixture bullet; Open risks first bullet)
    summary: D2 classifies by content ending on a tool use, but every transcript line of a split assistant message carries the message's final stop_reason, so a thinking or text line with stop_reason tool_use, which can stand as the newest entry for tens of seconds while a tool input streams, is neither in flight nor over under D2, and the criteria leave the thinking-then-text case without an expected reading
    fix: Rewrite D2's in-flight/over rule to classify an assistant entry by its message's stop reason, not by its last content block - stop reason tool_use reads in flight, end_turn and stop_sequence (which Claude Code's synthetic API-error entry carries) read over, any other stop reason reads unknown - and say the interrupt marker reads unknown because D1 releases on it first. In the claudeengine Acceptance criterion, replace "a thinking-then-text assistant message is covered" with explicit rows - a thinking or text line whose stop reason is tool_use, with no tool_use line yet, reads in flight; a thinking line whose stop reason is end_turn reads over; a synthetic API-error entry reads over. Keep the Open risks bullet, which the evidence confirms (thinking lines carry end_turn before the text line).
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md (Auto-mode assumptions, second bullet)
    summary: The real-run audit bullet is evidence, not an assumption the plan relies on; its one requirement (no backfill) is already Scope Out, and the audit itself is in support-log.md Interview turn 8 and Q3
    fix: Delete the "Real-run audit" bullet from decision-record.md's Auto-mode assumptions; change nothing in support-log.md, which already holds the audit in Interview turn 8 and Q3, and change nothing in Scope Out, which already holds "No backfill of a lens review for a landed run."
---

### [BLOCKING] D3's twill detection rests on a spawn record that does not exist

**Location:** `_lyx/discussion/decision-record.md`, D3 third paragraph.

**Issue:** D3 says twill-ness "is taken from what the Plan-Write spawn recorded (its fork-subagent authorization, which `PlanSpec` sets exactly under twill), never from a fresh `plan_producer` read, so a config edit between spawn and gate changes nothing."
The only place the authorization lives is `shuttleengine.Spec.ForkSubagents` in memory and one `run started` Info log line (`internal/shuttleengine/run.go:590`).
`run.json` holds no such field; this run's own record has `createdAt`, `eventsPath`, `interactive`, `outcome`, `outputFiles`, `pid`, `promptLine`, `promptOffset`, `promptPath`, `recordsSessionStarts`, `runId`, `sessionId`, `settingsPath`, `started`, `step`, `strandGuid`, `strandName` and nothing about forks.
`SingleLLMProducer.Call` (`internal/shedadapters/singlellm.go:126`) composes the spec on every Call through the wiring's `PlanSpec` closure (`internal/loomcli/wiring.go:763`), which reads `cfg.PlanProducer` (`internal/loomengine/plan.go:150`), and hands that freshly composed spec to `AttachGated` when it finds a live writer.
So on a resumed loom run that attaches to a live twill writer after a `plan_producer` edit, "what the spawn recorded" is exactly a fresh config read, which is the case the lens review's MEDIUM finding (support log turn 9, third bullet) asked to close.
The gate itself is a `GateSpec` fixed at wire time (`p.gate`), so the Decision also leaves unnamed how the gate closure learns the authorization of the run it judges.
The plan writer cannot implement the Decision as stated without adding a persisted field that Scope does not list.

### [MEDIUM] D2's in-flight rule misreads split assistant messages

**Location:** `_lyx/discussion/decision-record.md`, D2; the `claudeengine` Acceptance criterion; Open risks first bullet.

**Issue:** D2 reads in flight for "an assistant entry that ends on a tool use" and over for "an assistant entry whose stop reason ends the turn with no tool use."
In the recent main-chain transcripts under `~/.claude/projects`, one assistant message is written as several lines sharing a `message.id`, and every line carries the message's final `stop_reason`.
A text line with `stop_reason: tool_use` precedes its `tool_use` line: in one transcript, `msg_011CfyypgYh8562jgWTXwbot` wrote its text line at 09:13:52.772 and its `tool_use` line at 09:14:44.700, 52 s later, the shape a writer streaming a large `Write` input produces.
That text line does not end on a tool use, so it is not in flight under D2's wording, and its stop reason does not end the turn, so it is not over: it reads unknown and falls back to the idle-pane override, the release this task removes.
D2 also leaves "ends the turn" open: across 40 recent transcripts the stop reasons are only `end_turn` and `tool_use`, and Claude Code's synthetic API-error entry (`model: <synthetic>`, `isApiErrorMessage: true`) carries `stop_sequence`, which the plan writer must classify without guidance.
The Acceptance criterion says only that "a thinking-then-text assistant message is covered", with no expected reading, so it does not pin the case the Open risk names.
The Open risk's premise does hold: `msg_011CfyyipvnvdEdQX3kRZYhz` wrote a thinking line carrying `end_turn` 0.86 s before its text line.

### [LOW] Real-run audit sits in Auto-mode assumptions

**Location:** `_lyx/discussion/decision-record.md`, Auto-mode assumptions second bullet.

**Issue:** The bullet records an audit result (`lyx-halt` lacks `lens-review.md`, cause unconfirmed), not an assumption the plan depends on.
Completeness-before-leanness: its only plan-relevant content, "not backfilled", already stands in Scope Out as "No backfill of a lens review for a landed run," so nothing the plan writer needs is lost by deleting it.
The audit is already recorded in `support-log.md` Interview turn 8 and Question ledger Q3.

## Directive verdicts

- D1 boundedness: every release path is bounded in the Decision.
  A later session start, a reported interrupt, a turn end, `send_ready_timeout_s` and the run deadline cover the gate and the send.
  `Runner.SessionIdle` has no deadline of its own, and D1 states that it reports busy for as long as the turn reads in flight.
  That only lengthens a hold that today's classifier also keeps for a hung turn whose pane shows it running, so it opens nothing; not a defect.
  The Acceptance criteria cover the hold, the session-start release, the over-transcript release, the interrupt and the unreadable transcript; the `send_ready_timeout_s` and deadline bounds are explicitly unchanged and stay pinned by existing tests.
  Session starts are recorded for every Claude launch (`RunState.RecordsSessionStarts`), so the session-start lift is reachable.
- D2 against the criteria: F2.
  The interrupt marker reads "not in flight" in the criteria and is outside both of D2's classes; since D1 releases on the interrupt first, this is harmless, and F2's fix states it.
- D3 premise and bounds: F1 for the premise.
  The bounds are stated and sound: presence only, the gate's existing re-prompt budget, and `lens-review.md` kept across an in-run relaunch because `NewPlanDirRotator` runs only as the fresh-spawn preparation after the attach probe finds no live writer (`internal/loomshed/planwrite.go`, `internal/shedadapters/singlellm.go`).
  A gate evaluation at a turn end with forks outstanding is already deferred (`turn end waiting on background work`, `internal/shuttleengine/wait.go:910`), so the new finding does not spend the budget while forks still run.
- D3/D4 pairing: consistent.
  The directive's step 3 and step 8 both render `{{.self_check_command}}` today (`contracts/stencils/twill/twill-directive.md`), so D4's second value is a real stencil change, which Constraints already ties to `markers_test.go` and `claims_test.go`.
  The writer's final mode (`--lens-review`) and the twill gate run the same check set; the flag-less mode ignoring `lens-review.md` is described the same way in D3, D4 and the criteria.
- Notes for the plan writer: kept.
  The scratch paths locate the bench transcripts the `claudeengine` Acceptance criterion says the fixtures are cut from, so the plan writer needs them.
- Goal and Scope premises: supported.
  Support log turn 2 records the release Warn 10.2 s after the hold and the unanswered task-notification entry as the transcript's newest; turn 7 records that the running-turn hint shares a width-truncated footer.
  Scope Out keeping the classifier and skipping a Discussion-Write check follows from D1 and from D3's presence-only check.
- Constraints: the `twill-fork-timing` line holds for D4, whose steps exist under either spawn point; for D3 it holds once F1 names a persisted record, since that record does not depend on when forks spawn.
  The `PATTERN-sole-parsers` and `PATTERN-gate-self-check-parity` lines name which invariant each Decision satisfies rather than restating it as a new requirement; not a defect.
