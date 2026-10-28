---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lym/plan/07-test-tier-check.md (Intent, the test-tier-unstated pass rule and its way-forward text)
    summary: one Intent line naming two created test files with different tiers fails both files, and the way forward then pushes the writer to a wrong tier statement
    fix: >-
      In card 7's Intent, extend the way-forward text of `test-tier-unstated` so a tier statement sits on a line that names no other test file of the card:
      "way forward: add an Intent line naming the file and its `//go:build` line, such as `//go:build tier0`, on a line that names no other test file; where the lines name several tags, join them with || only for a helper several tiers share, else keep one; drop untagged or the tag when both appear".
      State the same line-per-file rule in the `test-tier-unstated` row and card rule that the Docs paragraph adds to `contracts/specs/loom-plan-spec.md`.
      Add one row to the D7 rows of `TestValidatePlan_CardChecks`: a card creating `internal/x/a_test.go` and `internal/x/b_test.go` whose single Intent line names both files with `tier0` and `tier1` fails both files with `test-tier-unstated`, each Detail naming both tags and the own-line clause, and the same two statements split over two Intent lines pass.
      No target list or verify line changes, so no `validate-plan --overlay` check constrains this fix.
  - id: F2
    severity: LOW
    class: consistency
    location: _lym/plan/09-doc-go-check.md (the Extends entry internal/loomcli#loomCLI.validatePlanCmd)
    summary: card 9 lists validatePlanCmd under Extends, but naming a third check in its Long help rewrites the sentence cards 7 and 8 already rewrote
    fix: >-
      In card 9, remove `internal/loomcli#loomCLI.validatePlanCmd` from the `**Extends:**` group and add it to the `**Edit:**` group, after `plan:internal/loomshed#TestValidatePlan_CardChecks`.
      Checked with `lym loom validate-plan --overlay` over card 9 with that move: the result equals the baseline `{}` run, with no new finding.
---

### [MEDIUM] A shared Intent line gives every file on it the union of the tags, and the way forward does not say to split the line

**Location:** `_lym/plan/07-test-tier-check.md`, the paragraph beginning "For each ref of a card's `Create` group", and the `test-tier-unstated` way-forward string in the same paragraph.

**Issue:** The pass rule collects "the card's Intent lines that name the file" and then "the tier tags those lines name as whole words".
A card that creates two test files at different tiers will naturally state them in one sentence, such as "`a_test.go` carries `//go:build tier0`, `b_test.go` carries `//go:build tier1`".
That line names both files and both tags, so each file collects `tier0` and `tier1` and each fails, though the statement is true and complete.
The way forward the card gives for that state is "join them with || for a helper several tiers share, else keep one".
Joining states a false tier for each file, and keeping one is impossible for the file whose tier is the other one.
The same trap holds for `untagged` beside a tag: "`helper_test.go` is untagged; `x_test.go` is `//go:build tier0`" fails both files with the "drop untagged or the tag" clause.
A re-prompted writer following the text either records a wrong tier or finds the split by trial.
The focus question for this card asked whether the way forward leads to a passing plan for each failing state the card pins; the no-tier-line state does, and the several-tags and `untagged`-plus-tag states do not when one line carries two files.
D7 does not state this bound, so the plan's text, which implements D7 faithfully, is the place to close it; the fix changes only the way-forward text, the spec row and one test row, never D7's rule.

### [LOW] Card 9 lists `validatePlanCmd` under Extends, but its Long help change rewrites an existing sentence

**Location:** `_lym/plan/09-doc-go-check.md`, the `**Extends:**` entry `internal/loomcli#loomCLI.validatePlanCmd`, and the Docs paragraph "`validatePlanCmd`'s Long help names `doc-go-unlisted`".

**Issue:** The current Long help in `internal/loomcli/validate.go` has one sentence naming the checks every mode but `--require-approved` runs, "…also runs rename-tool-missing and pattern-changes-mismatch, which read language.yaml and the decision record".
Cards 7 and 8 list `validatePlanCmd` under Edit for the same help change, which already rewrites that sentence, since the new checks do not read `language.yaml` or the record.
Card 9 adds a third name to the same list, which moves an "and" and reshapes the clause, so it rewrites existing content rather than only adding to it; the rubric says to move such a ref under Edit.

## Rejected

- Card 7, no tier line: the way forward "add an Intent line naming the file and its `//go:build` line, such as `//go:build tier0`" yields exactly one tag and passes; no finding.
- Card 6, a test reflecting over a struct declared outside the module: unmarked is a violation, and marked with the import path as unit (`//lym:seam time#Time`) passes, because the marker's first path segment names no directory under `root`; the fixture rows pin both states and a marker under a module directory is still checked, so there is a passing state.
  The unmarked violation line names the generic `<unit>#<Type>` form and the outside-module spelling is in the PATTERN background only; a writer reading the failure can reach the background, so this is not raised.
- Card 6, the marker list against the tree: every `NumField`, `VisibleFields` and `NilSeams` caller in the `_test.go` files of the worktree maps to a test the card marks, including the two tests that reflect through a helper (`assertStructFieldsSideFree`, `assertNoZeroFields`), and `MergeStateForTest` is a struct declared in `export_test.go`, which the scan's declaration check reads; no finding.
- Cards 8 and 9, Tests paragraphs against card 7's rows: card 8's `TestGateParity_PlanGate` case and `TestPlanWriteEntry_GateConfig` row pin the wiring of the new `SeamTests` field, which card 7's rows pin only for `TierTags`; card 9 adds no parity or verb row and relies on card 7 and 8's for mode reach.
  The done-card and `language: csharp` rows of each card pin a separate `plan.GoToolchain()` branch and a separate walk over the told cards in each check, as the three cards describe them.
  Every row is a row of an existing table, which is not the new-test-function shape the rubric flags; no finding.
- Card 9, `doc-go-unlisted` on a card whose only target in a `doc.go` package is a non-Go file: the card skips a file target not ending `.go` and carries a row (`internal/d/template.yaml`), which agrees with D8's "touch non-test Go code"; no finding.
- Cards 4 and 8 edit non-test code of `internal/shedrecipe` without a `shedrecipe/doc.go` listing: the check they add is not in force on this plan, and the doc names `Env` generally without listing the plan-gate fields, so no doc text is falsified; not raised.
