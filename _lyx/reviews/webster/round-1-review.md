---
verdict: APPROVED
---

No findings.

Reviewed range: the card 4 commit `998b856d8`, the only first-parent non-merge commit after the rejected head `4712107f3`.

- **Judge parts per host.**
  `Directive` fills exactly one judge part through `judgePart`: `twill-judge-critical` when `Values.CriticalJudge` is set, otherwise `twill-judge-plain`.
  `planLensDirective` sets `CriticalJudge` and no `RejectionRule`; `discussionLensDirective` sets `RejectionRule` and no `CriticalJudge`.
  An empty `RejectionRule` with no critical judge, or an empty `GoverningInput` or `DisputeEvidence` with one, fails `fillPart` with an error naming the marker and the stencil.
- **Stencils.**
  `twill-directive.md` carries one required top-level marker `{{.judge_steps}}` and no conditionals.
  `twill-judge-plain.md` matches `main`'s steps 4 and 5 byte for byte, and `twill-judge-critical.md` holds the moved critical-fixer steps verbatim.
  Both hosts keep the re-prompt rule and the fork boilerplate, which the directive carries outside the judge steps.
- **Tests.**
  No new top-level test was added; the cases are rows or assertions of the existing tests the plan names.
  The writer-fork rows pin the critical part's placement and the plain part's absence, and the new `TestDirective_MissingStencilNamesIt` rows pin that only the chosen part is read.
  `go test` over `contracts/stencils`, `internal/twilldirective` and `internal/loomengine` passes.
- **Docs.**
  `internal/twilldirective/doc.go` describes the host-chosen judge parts, and `docs/overview.md` returns the Discussion-Write lens line to `main`'s wording while the Plan-Write line keeps the critical fixer.
