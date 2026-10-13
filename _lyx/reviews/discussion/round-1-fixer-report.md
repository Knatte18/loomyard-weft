# Fixer report — discussion round 1

## Changes

- **F1** (BLOCKING), fixed as proposed.
  Premise confirmed: `RunState` (`internal/shuttleengine/rundir.go`) has no fork field, and `SingleLLMProducer.Call` composes the spec through `PlanSpec` before `AttachGated`.
  D3 now has `RunState` record the spec's `ForkSubagents` at spawn, and the Plan-Write gate reads it from the run it judges, attached or spawned, never from the spec composed for that Call.
  D3 also says the gate evaluation receives that recorded authorization, since the `GateSpec` is fixed at wire time.
  Scope In gains the run-record field, tolerant of its absence (reads as not twill).
  Acceptance criteria gain the attach-after-config-edit row in both directions.
- **F2** (MEDIUM), fixed as proposed.
  D2 classifies an assistant entry by its message's stop reason: `tool_use` reads in flight, `end_turn` and `stop_sequence` read over, any other reads unknown; the interrupt marker reads unknown because D1 releases on it first.
  The `claudeengine` Acceptance criterion replaces the thinking-then-text line with explicit rows for a `tool_use`-stop thinking or text line, an `end_turn` thinking line and a synthetic API-error entry, and the interrupt marker row now reads unknown.
  The Open risks bullet is kept.
- **F3** (LOW), fixed as proposed.
  The real-run audit bullet is deleted from Auto-mode assumptions; `support-log.md` and Scope Out are unchanged.

## Disputed

None.

## Deferred

None.
