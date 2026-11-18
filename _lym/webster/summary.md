# Result-level checks in Code-Review, Drift and PR-Review

All five batches recorded `done`, with no `Pattern-Departure` trailers and no renames.

## Batches

- **01 fabric-range-reads** (head `02153143`): fabric lists a range's changed files with diffs and its commit messages.
  The judge request, judgement, records and stencil carry file rulings.
  Deviation: `internal/driftengine/judgement.go`.
- **03 drift-verify-loosened-checks** (head `17a45b72`): Drift-Verify lists, judges, stores and halts on loosened checks and renders them in the verdict.
  The loom key `pr_review_small_max` defaults to 3.
  The pull-request record gained the send-back mark, fix-here count and send-back count.
  Deviations: `internal/driftengine/loosened.go`, `verdict.go`, `verify.go`, `verify_test.go`, `internal/loomcli/commitrecords_test.go`.
- **06 stencil-plain-includes** (head `095dfcfa`): `ReadForEngine` resolves plain fragment includes.
  The review input became a list, and a send-back file forces a full Code-Review generation.
  Deviations: `internal/loomcli/subjects.go`, `internal/shedadapters/amendment.go`, `bouncer.go`, `bouncer_amendment_test.go`,
  `settlerecord.go`, `internal/shedrecipe/entries_burler.go`, `internal/stencilstore/engineblock.go`.
- **08 loom-pr-findings-verb** (head `a46c6fab`): `lym loom pr-findings` decides send-back or fix-here and records the miss.
  PR-Review reroutes a marked send-back and clears it after the reroute persisted, per the arbiter's D4 decision.
  Deviations: `internal/landingshed/prreview.go`, `prreview_test.go`, `prreview_wait_test.go`,
  `internal/loomcli/arm.go`, `landingdeps.go`, `prfindings.go`, `prfindings_test.go`.
- **10 recipes-pr-review-reroute** (head `afc33f8e`): the tapestry, darn and sampler recipes give PR-Review `on_reroute: Code-Bouncer`.
  The shared Loosened-Check fragment is included in the code-committing briefs, and the rubric, PR-Review brief and figures rule gained their wording.
  Deviations: `contracts/stencils/stencils.go`, `internal/loomrecipe/testdata/tapestry-rows-before-motif.yaml`.

## Notes from the forks' digests

Batch gates ran at tier0 and tier1 only.
The batch 10 fork did not confirm that the four briefs taking the fragment include at their end render it.

## Verify gate findings

The plan-level verify failed on `burlerengine`'s `TestEngine_Fix_TableShapeAndFiles` (`single_fixer` and `twill_fixer`)
and `TestTemplate_OptionalDirectives` (`fix_step_renders_an_empty_optional_marker_cleanly`).
No recorded card touched a failing package.
One fixer fork was spawned for them.


## Verify gate fix commits

The verify-gate fixers' commits above the pre-fix head, and the PATTERN departures their messages declare.

- `37027c9107f7b4dc645197ce765cfdcaafb169fe` fix: render the burler fix step with its Loosened-Check include in tests
