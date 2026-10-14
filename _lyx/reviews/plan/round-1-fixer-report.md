# Round 1 fixer report

## Changes

- F1, fixed as proposed: card 3 adds `internal/yamlengine#SetValues`, `#SetSparse`, `#TestSetValues` and `#TestSetSparse` to `**Edit:**`.
  It also adds an `internal/yamlengine` Intent section: a value set on a `!!null` template leaf takes the tag its own text resolves to, in `SetValues` and `SetSparse`.
  Each of the two tests gains a row setting `plan_max_bounces=2` over a `~` template that asserts an integer 2 with no `!!null` tag, decoding into an `*int` as 2.
  The ImpactSummary names the yamlengine change.
  Premise checked: `set.go:134` assigns only `.Value`, `applyExistingOverrides` copies an existing leaf's tag, `layers.go:204` takes the template leaf's tag, and yaml.v3's `encoder.node` keeps a tag that differs from the value's resolved tag.
- F2, fixed as proposed: card 7's `HoldReading` gains the exception.
  An `awaiting` stop that `Check` refuses, whose escalation reader reports nothing pending, reads `Eligible` true when an attempt at `StopKeyOf(status)` has no applied mark.
  For an awaiting stop it lists the attempts before deciding `Eligible`.
  `TestHoldReading` gains the Plan-Bouncer row with a round-1 circling decision and a `resume` attempt with no applied mark, which reads eligible with `HandOnAt` and `AppliedAt` zero.
  Card 8's incident-shape row of `TestNotice_EligibleStopHold` adds the reading with the marker gone and a `resume` envelope with no `AppliedAt`, holding until `AppliedAt` or the backstop.
  Premise checked: `PendingEscalation` returns not pending once the decision file exists (`circling.go:164`).
- F3, fixed as proposed: card 3 moves `loomCLI.reviewBudget` from `**Uses:**` to `**Edit:**`.
  The Intent bullet now has it test `c.cfg.ReviewHardMaxBounces > 0`, which card 1's `CheckResolved` keeps at least 1, and drop `skip_merge_review` from its doc comment.
- F4, decision: the first route was taken.
  Card 3's MigrateConfig bullet now states the departure from point (4), names the sparse worktree-layer case it leaves, and says why the hook cannot tell that file from a `run_settings` or local document.
  The second route was not taken because it changes `configreg.Module.Migrate`'s signature for every module to serve one transition.
- F5, fixed as proposed: in card 2, the `New`/`Routing` bullet and the doc bullet now say a zero-round segment's rows carry `MaxBounces` 0, which the progress projection reads as `shedengine`'s inherited budget.
  They also say the `Routing` doc's never-the-engine-default claim excepts a perch at 0.
- F6, fixed as proposed: card 4's `ValidateLocalValues` bullet now words the decode failure as a fail-loud guard that wraps the error marked `configengine.ErrInvalid`, with no refusal-spec row and no test row.

## Disputed

None.

## Deferred

None.
