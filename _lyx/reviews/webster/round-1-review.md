---
verdict: APPROVED
---

No findings.

Range: `product.parent` is `main` in `_lyx/shed/twill-fork0/status.json`, and `_lyx/loom/rework/` does not exist, so the range is `git diff $(git merge-base main HEAD)..HEAD`, the three card commits.
`lyx gate test` over `./contracts/stencils`, `./internal/twilldirective`, `./internal/loomengine` and `./internal/burlerengine` passes.

## Rejected

- Focus 1, `Directive` and the stencils: exactly one of `writer_fork` and `relaunch_rules` is non-empty, set from `WriterFork != nil` in `internal/twilldirective/twilldirective.go`.
  The directive's two optional markers sit on no line of their own, so the other mode's bytes are unchanged.
  An empty host value fails `fillPart` with an error naming the marker and the part's stencil, pinned by the new `TestDirective_MissingValueNamesTheMarker` rows.
  Not a defect.
- Focus 2, `internal/loomengine/twill.go` and `plan.go`: `planLensDirective` alone sets `WriterFork`, and `discussionLensDirective` leaves it nil.
  `TestDiscussionTable` asserts no "fork0", and the relaunch paragraphs and the host-neutral re-prompt rule render there.
  Not a defect.
- Focus 3, `twill-directive.md`: the old rejection wording and "a later turn of the same session" are gone, and `wordingClaims` pins their absence.
  `planTwillRejectionRule` has no remaining reference.
  Not a defect.
- Focus 4, `internal/burlerengine/template.yaml` and `config_test.go`: `branch-coverage` is in the `twill-plan` fan in the card's order.
  The text of the three lenses matches the card, and the test pins the counts, the order and the three sentences.
  Not a defect.
- Focus 5, test economy: the new `wordingClaims` rows pin stencil-only wording, and the host tests pin the host values and where they land.
  Each new case is a row or assertion of an existing test, with no new top-level test.
  I found no duplicate.
- Focus 6, docs: the `internal/twilldirective/doc.go`, `docs/overview.md` and `internal/loomengine/template.yaml` changes match the behavior the diff adds.
  The remaining mentions of the old wording are under `docs/research/`, which records benches.
