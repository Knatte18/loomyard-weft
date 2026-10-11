# Card 6 — row-name-prose

**Edit:**
- `internal/loomshed/doc.go`
- `internal/loomshed#PlanReviewSkippable`
- `internal/loomshed/publishfailure.go`
- `internal/loomrecipe/doc.go`
- `internal/loomrecipe#NewDarn`
- `internal/impactset/doc.go`
- `internal/verifytree#Site`
- `internal/webstercli#websterCLI.verifyCmd`
- `internal/planglyph#ValidateFormat`
- `internal/discussionparser/validate.go`
- `internal/shuttleengine#dispositionCandidate`
- `internal/shedrecipe#configRejectUnknown`
- `internal/shedrecipe#resolveUnderRoot`
- `contracts/specs#LoomPlanSpec`
- `contracts/specs/loom-plan-spec.md`
- `contracts/specs/loom-status-spec.md`
- `contracts/stencils/loom/loom-rubric-discussion-review.md`
- `contracts/stencils/loom/loom-rubric-plan-review.md`
- `contracts/stencils/loom/loom-rubric-webster-review.md`
- `contracts/stencils/loom/loom-rubric-darn-review.md`
- `contracts/stencils#wordingClaims`
- `pattern/PATTERN-gate-self-check-parity.md`
- `pattern/PATTERN-verified-tree.md`
- `pattern/PATTERN-test-speed.md`
- `pattern/PATTERN-comment-line-breaks.md`

**Intent:**
Card 4 renamed the Burler rows; this card brings every comment, help text, spec and background file that names a retired row up to date, so none points at a row that no longer exists.
A sentence about a gate names the fix row (`Discussion-Burler-Fix`, `Plan-Burler-Fix`, `Webster-Burler-Fix`, `Darn-Burler-Fix`), since the `discussion`, `plan` and `verify` gates moved there; a sentence about the review or about the rubric's reader names the review row, or the review segment where it means both.
Covered: `loomshed`'s package doc, `PlanReviewSkippable`'s doc and `publishfailure.go`'s header; `loomrecipe`'s package doc, `NewDarn`'s doc and `darnrecipe.go`'s header; `impactset`'s package doc; `verifytree.Site`'s `Label` doc; `webster verify`'s help; `planglyph.ValidateFormat`'s doc; `discussionparser`'s `validate.go` header; `shuttleengine`'s `dispositionCandidate` comment; the `BurlerRound` mentions in `configRejectUnknown`'s and `resolveUnderRoot`'s docs; `specs.LoomPlanSpec`'s doc; `loom-plan-spec.md` and `loom-status-spec.md`; the four loom rubric stencils' banners and the Webster rubric's plan-format line, with the `wordingClaims` entry that pins that line; and the four PATTERN background files.
No behavior changes; `TestStencilClaims` is the only test that reads any of this text.

**ImpactSummary:** comments, help text and markdown only, across the packages and specs that name a Burler row.

**Commit:** 6: docs: name the Burler review and fix rows
