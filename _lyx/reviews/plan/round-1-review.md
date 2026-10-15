---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/01-replace-stencil-claims.md (the TestLoomRubrics paragraph)
    summary: >-
      Card 1 has TestLoomRubrics assert that every rendered rubric contains /abs/specs/dir, but loom-rubric-darn-review.md has no specs_dir marker, so the prescribed assertion fails on a correct tree and card 1's own gate goes red
    fix: >-
      In card 1's Intent, replace "every rendered rubric contains the `specs_dir` value it renders with, `/abs/specs/dir`" with an assertion scoped to the rubrics that carry the marker. Give TestLoomRubrics' table a per-row boolean (for example `namesSpecs`) set true for loom-rubric-discussion-review, loom-rubric-plan-review, loom-rubric-webster-review and loom-rubric-merge-review and false for loom-rubric-darn-review, and assert that the rendered text contains `/abs/specs/dir` exactly when the flag is set, the same equality shape the publish_failure, webster_record and merge_review checks already use. State that the darn-review rubric carries no `{{.specs_dir}}` today and that card 8 adds none.
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/plan/01-replace-stencil-claims.md (the TestLoomRubrics paragraph, its last sentence)
    summary: >-
      Its testtiming:keep reason refers grammatically to TestLoomRubrics, which carries no testtiming:keep line, while the quoted new reason belongs to TestStencils_TopLevelMarkers in markers_test.go
    fix: >-
      In card 1's Intent, move the sentence that starts "Its `//testtiming:keep` reason stops naming the wording claims" out of the TestLoomRubrics paragraph and into the TestStencils_TopLevelMarkers paragraph, and name the test outright, for example "`TestStencils_TopLevelMarkers`' `//testtiming:keep` reason stops naming the wording claims: ...", keeping the quoted replacement reason as it is.
---

### [BLOCKING] Card 1's new TestLoomRubrics assertion fails for the darn-review rubric

**Location:** `_lyx/plan/01-replace-stencil-claims.md`, the paragraph that starts "`TestLoomRubrics` gains one assertion".

**Issue:** The card says "every rendered rubric contains the `specs_dir` value it renders with, `/abs/specs/dir`".
`TestLoomRubrics` in `contracts/stencils/rubric_test.go` iterates five rubrics, `loom-rubric-darn-review` among them, and renders each through `shedadapters.ReadRubric(dir, tt.name, "/abs/specs/dir", ...)`.
`grep -c '{{.specs_dir}}'` over the rubrics gives discussion-review 1, merge-review 1, plan-review 6, webster-review 2 and darn-review 0, and `grep specs` finds nothing in `loom-rubric-darn-review.md`.
The rendered darn-review rubric therefore never contains `/abs/specs/dir`, so the assertion as prescribed fails on an unchanged tree.
Card 8 states that "No rubric gains a marker", so nothing later in the plan closes the gap.
The card's own justification only needs the marker kept in the plan-review and webster-review rubrics, the two whose claims rows named it, so a scoped assertion keeps the D2 coverage it is there for.

### [LOW] The testtiming:keep sentence is attached to the wrong test

**Location:** `_lyx/plan/01-replace-stencil-claims.md`, the last sentence of the `TestLoomRubrics` paragraph.

**Issue:** The sentence "Its `//testtiming:keep` reason stops naming the wording claims: 'pins the exact marker set of each stencil, which the rubric test's subset check and the render tests do not'" follows two sentences about `TestLoomRubrics`, so "Its" reads as `TestLoomRubrics`.
`TestLoomRubrics` in `rubric_test.go` has no `//testtiming:keep` line.
The reason being replaced, "pins the exact marker set of each stencil, which the rubric test's subset check and the wording claims do not", sits on `TestStencils_TopLevelMarkers` in `markers_test.go`, which the card lists under `Edit:`.
A fork reading the card literally could add a `//testtiming:keep` line to `TestLoomRubrics` and leave the stale reason on `TestStencils_TopLevelMarkers`, which still names the wording claims that this card deletes.

## Checked and found sound

- Fidelity: D1 is carried across cards 1 to 7, D2 by card 1, D3 by cards 1, 2 and 8, D4 by card 8, and D5 by the overview's Shared Decisions.
  None of the cards touches the PATTERN scans over stencils (`parentdirective_test.go`, `roleskills_test.go`).
  The overview keeps recipe-carried prose and Go-built text, and no card prunes either: card 3 keeps `prompt.go`'s blocks, and card 5 keeps the Bouncer's Go-built decision-rule sentence.
- Card 1's Go-value rows match `claims_test.go` (the grep for `gatecli.`, `shedrun.` and `shuttleengine.` hits lines 596, 1058, 1088, 1134 and 1142, and the `status:`/`head_sha:` row is line 563).
  `websterengine.Report` exports `Status` and `HeadSHA` with `yaml:"status"` and `yaml:"head_sha"` tags, and `go list -deps ./internal/websterengine` shows no import of `contracts/stencils`, so the in-package test does not create an import cycle.
- Card 1's marker positions hold on the current stencils.
  `webster-body-implementer.md` ends in `{{.kickstart_pack}}\n`, line 51 is followed by a blank line and `## The sandbox rule`, and each section-scoped marker sits inside its named heading's section.
  The four new `TestStencils_TopLevelMarkers` rows match the markers grep finds in those files.
- No later card uses a symbol from the deleted `claims_test.go`, and card 2's pointer to card 1's position table respects card order.
- Card 4's Go-parsed values exist (`JudgeVerdict` constants, `judgeHeader.Rationale`, `handoffHeader.CoversRounds`/`Ledger`), and each judge stencil names its own set.
- Card 5's claim that the driver envelope keys have no referenceable Go definition holds: `trace_file` is written through map literals in `internal/shedverbs/step.go`.
- Tests that no card names hold only kept anchors, so no card is missing: `internal/loomcli`'s `TestArbiterPrompt` locates each Go-filled `none` by its line, and `internal/seatengine`'s seat-heading and operator-ban checks anchor Go-chosen fragments.
- `## verify:` follows the plan template's order, and `./...` with `-tags integration` covers `contracts/stencils`, `cmd/lyx` and every targeted `internal/` package.
- Card 8's Intent states what the downgraded guard lets through and what bounds it, in line with D4, and its `Prosa` typing over the seven `.md` files fits the glyph alphabet.
