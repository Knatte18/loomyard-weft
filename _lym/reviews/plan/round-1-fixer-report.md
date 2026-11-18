---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed-differently
    reason: The brief and the help text now place the findings file under the task worktree's .scratch/, as proposed, but the proposed TestPRFindingsVerb row was not added. That test fakes the dirty seam, so a row there cannot reach git's ignore rules and would pin only the fake's answer. The dirty refusal's way forward names .scratch/ instead, so an agent that wrote the file in the tree is told where it belongs.
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
---

## Changes

- **F1** (`_lym/plan/03-drift-verify-loosened-checks.md`): added `internal/shedrecipe#driftTestEnv` and `internal/shedrecipe#TestDriftEntries` under **Edit:**.
  A new tests bullet says `driftTestEnv` fills `Drift.ChangedFiles` and `Drift.CommitMessages` with empty answers, so `VerifyBuilds` still builds.
  It also adds `VerifyNilChangedFiles` and `VerifyNilCommitMessages` rows beside `VerifyNilCompare`.
- **F2** (`_lym/plan/10-recipes-pr-review-reroute.md`): added `internal/loomrecipe#TestRouting_ProgressAt` under **Edit:**.
  The Intent says its tapestry-tail subtest wants PR-Review's reroute to be `loomshed.NameCodeBouncer`.
- **F3** (`_lym/plan/07-bouncer-review-inputs.md`): took the review's second option and render the rubric when each round starts.
  `BurlerHalfDeps` gains `Rubric func() (string, error)`. `BurlerReviewProducer.Call` reads it before it builds the round's profile, and `roundProfile` sets `Profile.Rubric` from it.
  A failed read fails the call before any spawn.
  On a `rubric_stencil` row, `burlerReviewEntry` sets the seam to a closure that calls `ReadRubric` with `AmendmentNote` and `ReviewInputNote` read at each call.
  `Bouncer.rubricNotes` keeps reading the note at each call.
  Added the producer, its constructor, `Call`, `roundProfile`, `burlerRoundProfile`, `captureReview` and the affected tests and test builders under **Edit:**, and updated the ImpactSummary.
  Tests:
  - `TestBurlerReviewProducer_Call_ProfileAndOpts` gains a row for a changed answer and a row for a failing seam.
  - `TestBurlerReviewEntry_RubricStencil` gains a row that builds the row, writes a full generation's record naming `pr-findings-1.md`, drives one Call, and asserts the recorded rubric names that path.
- **F4** (cards 8 and 12), fixed differently:
  - Card 12's PR-Review brief has the agent write the findings file under the task worktree's `.scratch/` directory.
  - Card 8's help text says the same, and refusal 1's way forward names that directory.
  - I did not add the proposed `TestPRFindingsVerb` row; the frontmatter gives the reason.
  - `.scratch` counts as ignored because of the repository's `info/exclude` and because it is a link to a directory outside the worktree. It is not ignored because of `.gitignore:23`, whose trailing slash does not match a symlink.
- **F5** (card 7): added `//contracts/specs/shed-recipe-spec.md` under **Edit:**.
  The Intent says its Bouncer passage names `Env.ReviewInputs`, every input written after the settle, and the full generation that a full input forces.
- **F6** (card 10): added `//internal/loomrecipe/doc.go` under **Edit:**.
  The Intent rewords `loomrecipe/doc.go`, the header comments of all three recipes and their PR-Review row comments: PR-Review's Stuck halts, except a marked send-back's reroute to Code-Bouncer.
  Tapestry's header carries the exception the way it words Webster's.
- **F7** (card 9): added `internal/landingshed#PRReview` under **Edit:**.
  The Intent says its doc comment names the send-back reroute to Code-Bouncer among the ways the row's wait ends.
- **F8** (card 3): a Bound bullet in the halt passage says a trailer's measured cause is the committer's attestation, which neither the judge nor the code re-measures.
  It also says its bound is the verdict's `Loosened checks` lines, which reach the pull request's body and PR-Review's `drift_verdict`.

## Self-check

- fix-consistency, LOW: card 10's reword list left out tapestry's header (lines 29-31). Fixed as proposed: the list now names the header comments of all three recipes, and tapestry's carries the exception the way Webster's does.
- fix-consistency, LOW: the doc comments of `NewBurlerReviewProducer`, `roundProfile` and `burlerRoundProfile` were left stale by card 7. Fixed as proposed: the Intent has them name the rubric as a per-round field, and `burlerRoundProfile` was added under **Edit:**.
- fix-proposal, LOW: F6's list skipped tapestry's header. This duplicates the first fix-consistency finding and was fixed with it.
- fix-claims, LOW: `.scratch` is ignored through `info/exclude` and the symlink, not through `.gitignore:23`. Fixed as proposed: this report cites the real reason. The card text cites no rule, so it needed no change.
- fix-claims, LOW: card 10's "Every text…" sentence claimed more than it listed. Fixed as proposed: the sentence now says "These texts" and lists the tapestry header. The testdata copy `internal/loomrecipe/testdata/tapestry-rows-before-motif.yaml` is a fixture, not documentation, so it stays out.
- fix-affected-set, LOW: card 7's new producer rows need a `rubric` field on `halfProducerSpec` and `newReviewProducer`. Fixed as proposed: both were added under **Edit:**, and the tests bullet names the field.
- fix-affected-set, LOW: tapestry's header was missing from card 10's set. This duplicates the first fix-consistency finding and was fixed with it.

## Disputed

None.

## Deferred

None.

## Rulings

None.
