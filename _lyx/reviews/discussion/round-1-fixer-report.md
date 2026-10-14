# Round 1 fixer-report — discussion

## Changes

- **F1** (BLOCKING), fixed with my own change: the proposed fix's first branch was to settle the cause and say which one the hold fixes.
  The evidence cannot single out one window.
  `logging-cleanup`'s arbiter was still running after the 09:26:22Z notice, and batten's trace (`trace-20261013T073850Z-…`) logged no seam error, which leaves the pre-marker window or a marker reading not alive.
  The orch handoff `handoff-20261013T153428Z.md` places `tender`'s arbiter `continue` at the notice's minute.
  That also fits a third window the review did not name: `finish` in `internal/loomcli/arbiter.go` removes the marker before the driver acts on the verdict.
  The task worktrees holding the attempt records are gone.
  Rather than add an Open risk over a hold built for one presumed cause, D9's hold is now marker-independent: it runs from the stop's time whether or not a marker exists or reads alive, bounded by the later of stop time plus `arbiter_timeout_min` and the attempt's deadline, each plus the report wait.
  The deferral bullet is gone, and the rationale names both incidents with their notices.log times and the evidence.
- **F2** (MEDIUM): the criteria match D9's end conditions, including the park marker and the arbiter switched off.
  A separate criterion covers the cases where the hold does not start, and the notice follows `stateChangeReady`'s existing rules after the hold, so the relay wait stays.
  D9 states that switching the arbiter off mid-hold ends it at the next pass, and that spawns run out mid-hold only through an attempt at this stop key, which keeps the hold.
  The spawns precondition reads "spawns left or an attempt at this stop key" so the hold's own arbiter spawning the last allowed attempt does not end it.
- **F3** (MEDIUM): D2 now has `internal/reviewbudget` expose one local-values check that both loom's and darn's `CheckValues` hooks call.
  The loom-unchanged criterion names the one change, and a new criterion pins the upsert refusal.
  The Constraints refusal list names the upsert refusals.
- **F4** (MEDIUM): D5 states the not-eligible-never-error contract and each consumer's resulting route.
  The Go driver criterion now also covers an unreadable escalation record.
- **F5** (MEDIUM): D4 keeps an explicit `skip_review`, raises the budget to the template default and names both keys in the migration line, and a criterion pins it.
- **F6** (LOW): D4 widens `configreg.Module.Migrate`'s doc comment.
- **F7** (LOW): Scope In lists `internal/shedadapters` for an exported reader of the pending escalation's cause.
- **F8** (LOW): D3 carries the orch stencil's change.
  I removed the Notes line that duplicated it.
- **F9** (NIT): removed the commit reference from D6.
- Support log: turn 4 no longer says the cause was left to Plan.
  Q9 is updated, Q11 and Q12 record the F3 and F5 picks, and Review rounds has the round-1 entry.

## Disputed

None.

## Deferred

None.
