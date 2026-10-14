# Fixer report — discussion, round 1

Every finding's premise was checked against the code and holds; every finding is fixed as proposed in `_lyx/discussion/decision-record.md`.

## Changes

- F1 (D2): a review-row entry belongs to the round its output's review file names; a hand-on entry joins the open round and opens none; a goto or reroute opens a new round only when that visit runs a review; the output-to-round mapping goes through `shedadapters`.
  Premise confirmed in `BurlerReviewProducer.Call` (`handOn`, BudgetExempt Stuck) and `ResolveRound` (counts files on disk).
  Added the hand-on Acceptance criterion and narrowed the goto criterion to a goto that runs a review.
- F2 (D2): a Bouncer entry that routes nowhere ends the round in progress, followed by any D3 arbiter line, or stands before round 1; only a segment-ending Done stands outside any round.
  Added the round-3 escalation and `continue` Acceptance criterion.
- F3 (D11): the `discussion_producer`-keyed rewrites apply only to a document that sets `discussion_producer` itself; string entries in other documents stay unmigrated and are refused after migration, worktree-layer files included.
  Premise confirmed in `configreg.PrepareRunSettings` (`Migrate` before `ValidateLocal`) and `configsync.reconcileSparseFile`.
  Extended the migration Acceptance criterion with the `run_settings` and sparse worktree cases.
- F4 (D7, D8): D7 states the new reach of a non-empty `discussion_advisors` in any layer and its bounds (named and checked at load, registry-resolved model, no gate skipped, cost on the run that sets it), and sets no maximum count; D8's bounds sentence is limited to the lens forks and points to D7 for advisors.
- F5 (D7, D8, D9): template defaults of `discussion_advisors`, `discussion_lenses` and `plan_lenses` are empty; added the Acceptance criterion for a `loom.yaml` that sets none of them.
  Premise confirmed in `internal/loomengine/template.yaml`, which ships `discussion_advisors: ["sonnet[high]"]`.
- F6 (D6, D7): a later seat's name itself must pass the role grammar, as its formed role must; D7 cites D6's rule; a numeric seat name joins the Validate Acceptance criterion.
  Premise confirmed in `agentname.MatchesRole`.
- F7 (Goal 2): added the exception for D11's migrated `advisor-<n>` names.
- F8 (D3): attempts are read per directory through `loomengine.ReadArbiterAttempt`; an unreadable one shows as `arbiter attempt unreadable (<directory name>)` at the end of the segment's block; the Acceptance criterion follows.
  Premise confirmed in `loomengine.ListArbiterAttempts`, which returns on the first read error.
- F9 (D3): an attempt with no verdict reads `running` before its `Deadline` and `no verdict` after; the Acceptance criterion follows.
- F10 (D5): added the `pauserecord.Retention` sentence and a pointer under Auto-mode assumptions.
- F11 (D8): the guard bound now says no local override or `run_settings` can turn the lens forks off or change their fan.

## Disputed

None.

## Deferred

None.
