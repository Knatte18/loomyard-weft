# Card 1 — finding-class

**Edit:**
- `internal/burlerengine/verdict.go`
- `internal/burlerengine/verdict_test.go`
- `internal/burlerengine/engine_test.go`
- `internal/burlerengine/prompt.go`
- `internal/burlerengine/prompt_test.go`
- `internal/burlerengine/doc.go`
- `contracts/stencils/burler/burler-step-1-explore.md`
- `contracts/stencils/burler/burler-step-2-review.md`
- `contracts/stencils/claims_test.go`
- `CONSTRAINTS.md`

**Intent:** Give every burler review finding a required `class` (D4) and pin `design` as the one gating class (D5).

In `verdict.go`, add a `Class` string type with the four constants `ClassDesign` (`design`), `ClassScope` (`scope`), `ClassDecision` (`decision`) and `ClassConsistency` (`consistency`), plus `GatingClass = ClassDesign`, documented as the one class that decides when a review loop stops in every segment.
`Finding` gains `Class Class` with yaml key `class`.
`validateFindings` rejects a finding whose class is missing, empty after trimming, or outside the four values, fail-loud and burler-prefixed like the severity checks, naming the finding ID and the four legal values.
Update `ParseReview`'s rule list in its doc comment.

Tests in `verdict_test.go`: each of the four values parses and is carried on the returned `Finding`;
a missing `class`, an empty `class` and an unknown `class` (`Design`, `nit`) are each a parse error.
Every existing review fixture in `verdict_test.go` and `engine_test.go` that carries findings gains a `class:` line so the suite stays green.

Carry the field into what reviewers are told:
`burler-step-2-review.md`'s format example and strict rules add `class`, exactly one of the four values, and state the four generic meanings once, worded as D4 gives them (including that a recurring `scope` gap is raised once as a `design` finding about the method).
`burler-step-1-explore.md`'s rubric paragraph says each finding also carries one of the four classes and that the rubric says what `design` means for its target.
The cluster boilerplate in `prompt.go` asks forks to return each finding with a severity, a class, a location and a one-line summary;
update whatever `prompt_test.go` pins of that text.
Add claims in `claims_test.go` for `burler-step-2-review.md`: it names the `class` key and all four class names.

`doc.go` takes the lasting content of the draft design doc: the four classes, that `design` gates the loop in every segment, and the severity-ladder trap — class decides who decides and when the loop stops, never whether a finding is fixed; the fixer still fixes every finding of every class and severity.

`CONSTRAINTS.md`'s Review Round Invariant gains the D9 line: a finding's class decides who decides and when the loop stops, never whether it is fixed.

**ImpactSummary:** Makes `class` a required review-file key, so every review written without it now fails `ParseReview` and takes burler's existing malformed-review retry; no Go caller constructs `Finding` outside burlerengine's own tests.

**Commit:** 1: burlerengine: every review finding carries a class
