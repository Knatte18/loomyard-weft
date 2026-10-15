---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: _lyx/plan/04-judge-parts.md:75
    summary: The TestDirective_RendersEveryValueVerbatim wants pin the judge part's trailing edge only, at one host only, though the Intent says they pin its placement with no blank line on either side
    fix: In card 4's TestDirective_RendersEveryValueVerbatim bullet, move the want "returned no report.\n6. Run `lyx check --final` again" from the "without a writer fork" row into the shared wants, and add the shared want "forks are outstanding is allowed.\n4. Judge every finding", which both judge parts satisfy since each opens with "4. Judge every finding" right after step 3's "ending your turn while forks are outstanding is allowed.". Then both edges of the judge_steps marker are pinned at both hosts, which is what the sentence "which pins the part's placement with no blank line on either side" and the ImpactSummary's "Plan-Write's rendered prompt is unchanged" claim. No target or verify command changes.
---

### [LOW] The judge part's placement is pinned on one side and at one host only

**Location:** `_lyx/plan/04-judge-parts.md:75`, against `internal/twilldirective/twilldirective_test.go:40-131` (`TestDirective_RendersEveryValueVerbatim`)

**Issue:** The card puts `{{.judge_steps}}` on its own line between step 3 and step 6 of `twill-directive.md`, so a part without trailing newlines leaves the step list's bytes as they were.
Of the two wants the card gives the "without a writer fork" row, `"   REJECTION-RULE-SENTENCE\n5. Record the lens review at …"` lies inside the plain part and pins nothing about its placement.
Only `"returned no report.\n6. Run `lyx check --final` again"` pins an edge, the trailing one.
No want pins the leading edge, step 3's `"… outstanding is allowed."` followed directly by `"4. Judge every finding"`.
The test's existing `"\n\n\n"` check catches a run of two blank lines, not one, so a directive written `3. Wait …\n\n{{.judge_steps}}` would pass every planned row.
The "with a writer fork" row, which mirrors Plan-Write, pins neither edge, though the ImpactSummary says Plan-Write's rendered prompt is unchanged.
So the Intent's sentence "which pins the part's placement with no blank line on either side" does not hold for the rows it plans.

## Verdicts on the focus questions

- **Fidelity.**
  Card 4 carries every part of the live generation's F1 fix (`_lyx/loom/rework/round-1/findings.md`, round 1, whose `first_card` 4 matches the overview's).
  It covers the critical part, the plain part restored from `main`, both host values in `internal/loomengine/twill.go`, `contracts/stencils/markers_test.go` and `claims_test.go`, the `twilldirective` tests, the `discussiontable_test.go` rows, `docs/overview.md`'s Discussion-Write line and `internal/twilldirective/doc.go`.
  It also carries the decision record's last entry, "Critical fixer only at Plan-Write", while keeping D3's re-prompt rule and D5's boilerplate at both hosts.
  I checked the quoted `main` text against `git show main:contracts/stencils/twill/twill-directive.md` (lines 14-17), `main`'s `discussionLensDirective` `RejectionRule` and `main`'s `docs/overview.md:532`, and each matches byte for byte.
  The F1 fix asks for an "optional marker", but the card makes `judge_steps` required, which is sound: every host gets exactly one judge part, so a required marker turns a wiring fault into an error.
  No scope is left uncarried.
  There is no `internal/loomengine/doc.go`; the loomengine twill comments are `plan.go:13-22`, which describes Plan-Write only, `twill.go`, whose comments never name the critical fixer, and `template.yaml`'s `plan_lenses` comment, which is Plan-Write only.
  The card says the first and last stay, and they need no edit.
- **Targets.**
  Every test glyph the Intent edits is under `**Edit:**`: `TestDirective_RendersEveryValueVerbatim`, `TestDirective_MissingValueNamesTheMarker`, `TestDirective_MissingStencilNamesIt`, `filledValues`, `TestStencils_TopLevelMarkers`, `wordingClaims` and `TestDiscussionTable`.
  `TestPlanTable` is correctly under `**Uses:**`: its wants at `plan_test.go:315-316` keep holding once the critical part renders at Plan-Write, and no other want of that test reads the moved text.
  No edited member changes its signature (`Directive`, `filledValues`, `planLensDirective` and `discussionLensDirective` keep theirs, and `Values` is a struct), so no re-sign arrow is missing.
- **Reachability.**
  `stencil.FillOptional` reports only markers the template declares (`internal/stencil/stencil.go:292`), and `Directive` checks nothing before its fills.
  An empty `rejection_rule` in the shared map is therefore never refused by the directive, the writer-fork part or the relaunch part, and it is refused only by the plain part, which is the planned `RejectionRule` row.
  The `GoverningInput` and `DisputeEvidence` rows set `filledCriticalJudge()` and reach the critical part's fill.
  The two missing-stencil rows that expect an error reach `fillPart`'s read error, and the two clean rows pin that the unasked part is never read.
- **Test economy.**
  Every case is a row or assertion of an existing test, so the claim of no new top-level test holds.
  The new rows follow the existing writer-fork/relaunch row pattern, and none repeats a case another row pins.
  `cmd/lyx/testdata/test-budget.yaml` needs no edit.
- **Verify.**
  `## verify:` runs `go build`, `go vet` under `integration`, `tmux` and `llm`, then `go test` and `go test -tags integration`, all over `./...`.
  That covers `contracts/stencils`, `internal/twilldirective` and `internal/loomengine` in the order the plan template states.

I checked the stencil registry too: `stencilkit.Seed`, which both test packages' stencil directories use, seeds every registry entry, so registering the two parts in `entries` (`contracts/stencils/stencils.go:483`) is the one registration they need.
`lyx loom validate-plan --overlay` over `{}` reports `ok: true`, with two informational `create-new-unit` findings for the new stencil files.
