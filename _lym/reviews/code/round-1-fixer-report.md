---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed
  - id: F12
    disposition: fixed
  - id: F13
    disposition: fixed
  - id: F14
    disposition: fixed
  - id: F15
    disposition: fixed
  - id: F16
    disposition: fixed-differently
    reason: "The review proposed reading the Plan-Burler-Review row's run_subdir; the recipe API reads a run_subdir only for a Bouncer row (loomrecipe.BouncerRunSubdir), so validate-review reads the Plan-Bouncer row's, which the recipe declares equal to the review row's (contracts/recipes/tapestry-recipe.yaml:234) and which internal/loomcli/wiring.go:1140 already uses the same way."
---

# Fixer-report, round 1

## What changed

Each commit is `<target>: fix <ids> — <why>`; the gate (`lym gate test` over the touched packages) is green after the last one.

- F1: `withoutDroppedCreates` (`internal/runplan/postbatch.go`) also removes a dropped Create's `plan:` declaration from the group and card `Declarations`.
  `PostBatchChecks` hands `BindHandles` those filtered cards, and keeps `in.Cards` for the re-sign rebind, the scope guard and drift.
  `TestPostBatchChecks_DroppedHandleDeclarationIsNotBound` has a card declare two handles, drops one, and asserts binding is asked about the other alone.
- F2: the `DroppedCreates` read error is wrapped with the record path and a way forward naming `lym loom decision replace` and a re-run of the webster verb.
  `TestPostBatchChecks_UnreadableDecisionRecordNamesTheWayForward` reaches it with a directory in the record's place.
  A malformed added-decision heading is not reachable: `addedDecisionSpans` only treats a parseable heading as an entry, so it is no entry and no error.
- F3: `CardNotDoneError.DroppableCreate` is set for create-not-done alone and `createTargetWayForward` offers `--drops-create` only then.
  The rename-not-done row now expects the amendment and recover-batch, and a mixed rename-and-create row keeps the drop.
  The `websterengine` doc names the ways forward per finding kind.
- F4: `priorFindingsBlock` lists every `AmendingTexts` entry with the amendment mark, an earlier-round one under its own lead-in, and writes the amending paragraph whenever any exists.
- F5: `namedByAmendment` matches only `.md` files directly in the plan directory (also spelled under another root, since `_lym` is a link), and the stem or slug as a whole token.
  Rows: a longer named slug, a non-card file sharing a stem, a card spelled through the link.
- F6: the cluster-fan, assistant cluster-fan and assistant consolidation stencils keep `origin: webster` on the halt finding.
- F7: the `websterengine` doc states that refusal and prompt text name a loom verb only as a step for the run's parent.
- F8: the parent directive tells the parent to pass `--drops-create` again for a re-recorded call that carried `Drops Create:` lines.
- F9: `TestBurlerReviewProducer_Call_HaltReasonReachesTheReviewer` (spawned reviewer, probed and resumed reviewer under a differing recorded fan, no conflict directory) and a `TestBurlerReviewEntry_ProfileMapping` subtest (Plan-Burler-Review reads the halt file, another review row does not).
- F10: the selective-drop test drops one of two refs and asserts `ErrCardNotDone` names only the undropped ref and the warning names only the dropped one.
- F11: a `TestRecordBatch_DoneChecksBlockOnUnresolvedCreate` subtest records a batch whose geometry's decision record drops the unresolved target, with the warning.
  Weaver tests: a `Run` subtest and a re-entry subtest drop the Create through the decision record, covering `record.go` and `resume.go`.
- F12: the "master renders none" row asserts the parent decisions paragraph followed by `none`.
- F13: only a new `origin: webster` finding outside the changed files counts against the halt exemption; a second one is refused with a way forward that names a changed file.
- F14: `//testtiming:keep` above `TestParseAddedDecisions`.
- F15: `decision add` help restores the `--from` duplicate refusal, the `withdraws` refusal and the sampler's `--by operator` limit; the CLI length tests pass.
- F16: `validate-review` reads the plan segment's `run_subdir` from the recipe, and `planReviewRunSubdir` is deleted; see the disposition's reason.

Later commits rewrap the comment lines the line-length gate flagged, one sentence per line, with no behaviour change:
`roundcheck.go`, `postbatch.go`, `postbatch_test.go` and `burlerreview_halt_test.go`.
The `postbatch_test.go` comment needed a second pass.

The F15 restoration pushed the `decision add` help to 2089 bytes, over the 2000-byte cap `TestCLIForm_EveryCommand` enforces.
The help now drops the notice key names and the restore note, and keeps the three restored rules (the `--from` duplicate refusal, the `withdraws` refusal and the sampler's `--by operator` limit).

## Self-check

Lens reports are saved beside this report as `round-1-fixer-report.lens-<lens>.md`.
No fork returned without a report.

- fix-consistency, MEDIUM: `burler-assist-cluster-fan.md` lacked the F6 exception — fixed.
- fix-affected-set, MEDIUM: the same twin as above — merged with it and fixed.
- fix-consistency, MEDIUM: `burler-parent-consolidation.md` lacks the exception — disputed, premise false (see Disputed).
- fix-consistency, LOW: `websterengine/doc.go` still said a rename-not-done finding offers three ways — fixed.
- fix-affected-set, LOW: `burler-review-format.md:55` does not name the webster exception — disputed, premise false (see Disputed).
- fix-code-correctness, MEDIUM: the F13 refusal counted every webster finding and its way forward could not be met — fixed: only a finding outside the changed files counts, and the way forward names a changed file.
- fix-code-correctness, LOW: `resume.go`'s `DecisionRecordPath` had no test — fixed with a re-entry subtest.
- fix-code-correctness, LOW: F16 differs from the review's fix key — recorded as `fixed-differently`.
- fix-proposal, LOW: F5's plan-directory check refused a location spelled through the `_lym` link — fixed: the directory may end with the plan directory under another root.
- fix-proposal, LOW: a dropped handle Create that a recovery builds anyway leaves its handle unbound — deferred (see Deferred).

## Disputed

- fix-consistency, `burler-parent-consolidation.md:15,22` — ground: premise false.
  A parent finding is exempt from the changed-file rule before the origin check: `parentCovered` returns true for an `origin: parent` finding with `parent_ids` in the report, and `newFindings` then `continue`s (`internal/burlerengine/roundcheck.go:295`).
  A halt finding merged into a parent finding therefore never meets the changed-file rule, so the origin it keeps loses nothing.
- fix-affected-set, `burler-review-format.md:55` — ground: premise false.
  The format's last origin bullet already reads "A finding that raises the plan executor's halt reason, when the review step quotes one, carries `origin: webster`" (`contracts/stencils/burler/burler-review-format.md:58`), so the exception is named there.

## Deferred

- fix-proposal, F1: a dropped handle-declared Create that a recovery builds anyway stays unbound, so a later card's `plan:` reference to it stays unbound too.
  Binding it needs a call on a tradeoff that belongs to the run's parent: whether a decision that drops a Create also lets a recovery's built symbol bind, which means `BindHandles` (`internal/planglyph/handle.go`) must learn which handles were dropped, and a dropped handle the delta lacks must stay non-blocking.
  The review's decision is that a dropped target does not block, and the change follows it.

## Rulings

None reached me this round.
