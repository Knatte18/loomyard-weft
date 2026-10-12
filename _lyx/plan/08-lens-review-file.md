# Card 8 — lens-review-file

**Create:**
- `plan:internal/planparser#LensReviewFileName` -> `const LensReviewFileName = "lens-review.md"`

**Edit:**
- `internal/planparser#knownNonCardFiles`
- `internal/planparser_test#TestValidate_IndexFileMismatch`
- `internal/websterengine#planFileNames`
- `internal/websterengine#fingerprint`
- `internal/websterengine#TestFingerprint`
- `internal/loomshed#NewPlanDirRotator`
- `internal/loomshed#TestNewPlanDirRotator`
- `internal/loomshed#TestPRRework_FirstRoundMovesGenerationZero`

**Uses:**
- `internal/planparser#AmendmentsFileName`
- `internal/loomshed#stalePlanRotation`
- `internal/loomengine#PriorPlanBlock`

**Intent:**
D7: the tracked lens-review file in the plan directory, `lens-review.md`, where the twill plan writer records its lens review (card 9 names it).
The name is admitted under every producer, and its content is never parsed or validated.

- `internal/planparser` declares `LensReviewFileName` once, beside `AmendmentsFileName` in `amendment.go`'s const block or as its own documented const there, with a doc line: the plan-directory file the twill writer records its lens review in, tracked with the generation and never parsed (`PATTERN-sole-parsers`, `PATTERN-plan-generation`).
- `knownNonCardFiles` gains `LensReviewFileName`, and its doc names it, so `index-file-mismatch` never counts the file as a card.
- `websterengine`'s `planFileNames` skips `planparser.LensReviewFileName` as it skips the amendments file, so the file is no part of plan identity; the doc comments of `planFileNames` and `fingerprint` name both exclusions and why (a record about the plan, written by the pipeline's writer, not plan content).
- `loomshed.NewPlanDirRotator` still moves `lens-review.md` with every other top-level `.md` file into the `archive-*` directory, but the prior-plan block it renders names the moved files minus `planparser.LensReviewFileName`.
  When the lens review is the only stale file, the rotation still moves it and returns an empty amendment without rendering a block, since there is no plan for a re-run writer to carry forward and `PriorPlanBlock` refuses an empty list.
  Its doc comment states both.
- PR-Rework's round archive already moves every plan-directory entry into `prior-generation/plan/`, and `planCommitPathspec` stages the whole plan directory, so the file is committed and rotated with the plan with no code change there.

Tests:
- `TestValidate_IndexFileMismatch` gains a subtest beside the amendments one: a plan directory holding `lens-review.md` beside the indexed cards produces no `index-file-mismatch` finding.
- `TestFingerprint`'s mutation table gains a row adding `lens-review.md` to the plan directory, with the fingerprint required not to follow.
- `TestNewPlanDirRotator` gains two subtests: a plan directory holding `00-overview.md`, `01-card-one.md` and `lens-review.md` moves all three into the archive, and the amendment names the two plan files and not `lens-review.md`; a plan directory holding only `lens-review.md` moves it into the archive and returns an empty amendment and a nil error.
- `TestPRRework_FirstRoundMovesGenerationZero` seeds `lens-review.md` in the plan directory and asserts it lands under the round's `prior-generation/plan/` beside the cards.

Acceptance: "`lens-review.md` in the plan directory raises no `index-file-mismatch`, changes no webster plan fingerprint, is not named in the prior-plan block, and is committed and rotated with the plan."

**ImpactSummary:** two plan-directory enumerations and the prior-plan block skip one more fixed name; plans without the file behave as today.

**Commit:** `8: Admit lens-review.md in the plan directory`
