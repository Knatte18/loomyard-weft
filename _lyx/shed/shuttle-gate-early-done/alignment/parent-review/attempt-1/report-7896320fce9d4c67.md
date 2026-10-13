---
verdict: APPROVED
---

## Scope against the board entry

The entry `shuttle-gate-early-done` has no Done-when list; its goal is its title (the shuttle gate ends a twill writer mid-turn) and its three `Scope to discuss` points.

- Why the held turn start is released against a pane that only looks idle, and whether the idle test can see a thinking model: answered in support-log turns 3 and 7 and Q1/Q4; D1 and D2 add a transcript in-flight veto to `turnStartHold`, and Scope Out keeps the pane classifier unchanged.
- Whether the twill gate should refuse Done until the lens review is written: D3 adds the refusal for twill Plan-Write with the `validate-plan --lens-review` parity mode, and D4 has the twill Plan writer run that mode last; Discussion-Write is ruled out with a stated reason.
- Whether real runs lost their lens review: support-log turns 5 and 8 and Q3; only `lyx-halt` lacks one, cause unconfirmed, no backfill.

D4 touches the twill directive, which the entry does not name; it is the parity verb's caller in the writer's own loop that `PATTERN-gate-self-check-parity` pairs with D3, so it serves the entry's second point and is not another entry's work.
No decision duplicates or contradicts another board entry.
`twill-fork-timing` (Ready, not running) also edits the twill directive stencil and `internal/loomengine/twill.go` at the fork spawn point; the record names it in Constraints, and D3 and D4 hold under either spawn point.
`twill-plan-followups`, `llm-agnostic-engine`, `codex-engine` and `stale-binary-daemons` plan nothing on the turn-start hold or the Plan gate; a second engine without the capability is named in Open risks.

## Overlap with runs in flight

`lyx loom peers` lists no parent commits.

- `logging-cleanup` (row 12, Webster): text-only overlap in `internal/shuttleengine` (`run.go`, `run_test.go`, `doc.go`, `wait.go`, `session.go`) and `claudeengine/claudeengine.go`, plus `cmd/lyx/testdata/test-budget.yaml`.
  Its plan's D11 audit keeps or guards shuttle log lines and does not touch `turnStartHold`, `startHoldReleased` or the release Warn this run keeps as its only line.
  Its D8 changes how the orch watcher logs a disagreement between the idle probe and the session state; this run's D1 changes the probe's answer (`Runner.SessionIdle` reads busy while a turn reads in flight). The two change different behaviour and do not conflict.
  Landing order: `logging-cleanup` is further along (row 12 against own row 3), so it lands first and this run takes its changes at Merge-In.
- `twb-ftp-s-1`, `twb-ftp-r-1` (row 8) and `twb-ftp-s-2`, `twb-ftp-r-2` (row 3): bench pairs replaying `shuttle-startup-ready` (startup probe, input-box reading), with no changed paths; they are torn down after the run and do not touch the turn-start hold or the Plan gate. No overlap.
