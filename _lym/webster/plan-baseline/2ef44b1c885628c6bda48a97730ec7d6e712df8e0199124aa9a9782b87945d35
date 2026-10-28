# Card 4 — repo-invariants-lens

**Edit:**
- `internal/lenses/template.yaml`
- `internal/lenses#TestConfigTemplate_DecodesAndIsSelfConsistent`
- `plan:internal/lenses#TestTemplateChecks_FlagBrokenTemplates`

**Create:**
- `plan:internal/lenses#writerFanGaps` -> `func writerFanGaps(orders map[string][]string, writerOf map[string]string, absorbedBy map[string]string, reviewerOnly map[string]bool) []string`

**Uses:**
- `plan:internal/lenses#emphasisLineGaps`

**Intent:**
D2: in `internal/lenses/template.yaml`, add the procedure lens `repo-invariants`, and `review-plan` becomes `affected-set`, `guard-reach`, `repo-invariants`.
`way-forward` stays defined and stays in `twill-plan`, which is unchanged; its prose gains the emphasis line, "Report anything else you notice too — emphasis, never exclusion.", after its list, as `coverage`, `row-validity` and `forced-edits` end, since it leaves `review-plan`, the only procedure fan that exempted it.

`repo-invariants`' prose owns the repository invariants a card breaks without touching the line that breaks:
- Test seams: a field a card adds to a shared struct that a reflective seam test must cover, and each new test file's one tier tag.
- Doc text a card falsifies: the package `doc.go`, `PATTERN.md` entries and `pattern/` background files that describe behaviour the card's Intent changes, with no card listing them.
- Card order: a card that cannot pass its own gate from the state the cards before it leave.
- "A broken invariant no card lists is a finding.", and it ends in its own list, opening "Your list:", per card, each invariant checked, held or broken; no emphasis line follows, since `review-plan` is a procedure fan.
It names no check of `validate-plan` or `validate-discussion`, so it stays right whether or not `validator-checks` lands first.

Tests in `internal/lenses`:
- `writerFanGaps` is a test helper in `config_test.go` holding the writer-covers-reviewer check now inline in `TestConfigTemplate_DecodesAndIsSelfConsistent`: for each reviewer fan and its writer fan in `writerOf`, it returns one message per lens of the reviewer's order that the writer's order lacks, neither itself nor through `absorbedBy`, and that `reviewerOnly` does not hold.
- `TestConfigTemplate_DecodesAndIsSelfConsistent`: `wantLenses` becomes 38; `wantOrders["review-plan"]` is `affected-set`, `guard-reach`, `repo-invariants`; the inline check is replaced by a call to `writerFanGaps` with `reviewerOnly` holding `repo-invariants` alone, reporting one error per message.
  `wantSentences` gains `repo-invariants`: "A broken invariant no card lists is a finding." and "reflective seam test".
  `way-forward` is no longer in a `review-*` order, so the existing emphasis check, through `emphasisLineGaps`, now requires its line.
- `TestTemplateChecks_FlagBrokenTemplates` gains `writerFanGaps` rows over hand-built orders: a reviewer fan naming `way-forward` beside a writer fan lacking it gives one message naming `way-forward`; `affected-set` absorbed by `forced-edits` gives none; `repo-invariants` with an empty `reviewerOnly` gives one message.
  This pins D2's criterion that the check passes with `repo-invariants` as the one reviewer-only lens and still fails for any other reviewer lens missing from its writer fan.
- `config_test.go`'s file header, whose line 4 lists "each writer fan holding its reviewer fan's lenses", gains the exception: "each writer fan holding its reviewer fan's lenses but the reviewer-only repo-invariants".

The carry-over finding `d2-way-forward-loses-emphasis-exemption` is met by `way-forward`'s new emphasis line, which the emphasis check pins.

**ImpactSummary:** one template file and two tests in `internal/lenses`; `review-plan` is read only through `plan_reviewer_fan`, whose default names it, and no stencil names its lenses.
