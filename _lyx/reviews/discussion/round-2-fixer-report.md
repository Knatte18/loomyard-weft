# Round 2 fixer-report — discussion

## Changes

- **F1** (fixed as proposed): D7 moves `skip_alignment` to the closed set beside `alignment` and `alignment_pr_approve`, giving the reason that a local `skip_alignment: false` under a hub `true` hands both points back to the agent and re-opens the PR-Gate landing; only `alignment_timeout_min` stays open, bounded by the coherence check.
  The acceptance criterion now refuses a local-layer or `run_settings` `skip_alignment` and accepts only `alignment_timeout_min`.
  The `PATTERN-config-layers` Constraints line ("classified as D7 says") still matches and is unchanged.
  Premise checked: `scopeChanged` in `internal/boardengine/runlock.go` never compares `RunSettings`, and `pattern/PATTERN-config-layers.md` puts an approval gate under Guard.
- **F2** (fixed as proposed): D8 routes a decision on a held task to a board note of its own through `lyx board upsert` with kind note, naming the held task's slug, whenever `lyx loom decision add` refuses for want of a decision record (pre-loom batten rows, a halt before Discussion-Write, any darn run); the message to the Discussion-Write session is kept only for the refusal that names it.
  D8's Rationale names the note route, and D5's Bound sentence names both places a held task's decision lives.
  Premise checked: `decisionWayNoRecord` and `decisionWayWriter` in `internal/loomcli/decision.go` are distinct refusals, and `RunLockedError` in `internal/boardengine/runlock.go` names the note route.
- **F3** (fixed as proposed, using the content-hash option): D5 adds a decision snapshot, the content hash of the run's decision record and of the board notes naming the task, recorded at spawn; an approve against a changed snapshot is applied as an escalation.
  The snapshot covers the board notes as well because F2's route can change them mid-run.
  D5's Bound states the window beside the head and pull request check, D2's Bound mentions the check, and a new acceptance criterion pins it.
  The same check applies to D4's parent-review approve, since F2's note route and a message to the Discussion-Write session can both change what the agent read while it runs.
- **F4** (fixed as proposed): D5's skip condition, D5's escalation branch, and the PR-Gate acceptance criteria now say "approval or rejection record"; every remaining "decision record" means `decision-record.md`.
- **F5** (fixed as proposed, first option): D6's Bound lists the read set the Decision gives: board fields (title, status, parent branch), decision record, plan, changed-path list, and the caller's parent's commit list (short SHA and subject).

## Disputed

None.

## Deferred

None.
