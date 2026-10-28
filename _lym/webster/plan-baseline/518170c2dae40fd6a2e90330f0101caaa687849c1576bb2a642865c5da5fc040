# Card 9 — doc-go-check

**Create:**
- `plan:internal/loomshed#docGoFindings` -> `func docGoFindings(plan *planparser.Plan, cards []planparser.Card, worktreeRoot string, index planindex.Index) ([]planindex.Finding, error)`

**Edit:**
- `internal/loomshed#planContextFindings`
- `plan:internal/loomshed#TestValidatePlan_CardChecks`
- `internal/loomcli#loomCLI.validatePlanCmd`

**Extends:**
- `contracts/specs/loom-plan-spec.md`
- `contracts/stencils/loom/loom-template-plan.md`
- `internal/loomshed/doc.go`
- `internal/loomcli/doc.go`

**Uses:**
- `plan:internal/planindex#GlyphAnswer`
- `internal/planparser#ParseRef`
- `internal/planparser#PathEntryPath`
- `internal/testkit/plankit#Write`

**Intent:**
D8: a card that changes a package's non-test Go code covers the package's `doc.go`.

`planContextFindings` appends `docGoFindings(plan, cards, worktreeRoot, index)` after the seam-test findings when `plan.GoToolchain()` holds; its error is returned.
For each card, `docGoFindings` takes the refs of its `Create`, `Edit`, `Extends`, `Delete`, `Rename` and `Move` groups (never `Prosa` or `Custom`), a `plan:` prefix stripped, and maps each to its package directory through `planparser.ParseRef` and the glyph's unit path (a file unit's directory, a `<dir>_test` unit's `<dir>`).
A file target whose path does not end in `.go`, such as `internal/d/template.yaml`, is not Go code and is skipped, since D8 holds only a card touching non-test Go code.
A ref is test code when it is a file ending `_test.go`, has a `_test` pseudo-unit, is a member glyph `index.ResolveGlyphs` reports declared only in `_test.go` files, or is a member that does not resolve yet and either is a `Rename` new side whose old side is test code or has a member name beginning `Test`, `Benchmark`, `Fuzz` or `Example`; every other ref, an ambiguous answer included, is non-test code.
The name rule applies to any unresolved member, a `Create` handle included, so a card creating `plan:<dir>#TestNew` is test-only, while one creating a helper handle whose name begins with none of the prefixes is not, as the bound says.
All member refs of the told cards are resolved in one `ResolveGlyphs` call, made only when there is one.
A directory with no `doc.go` on disk is skipped, which covers a package the plan creates.
The card covers `<dir>/doc.go` when any of its own groups or its `Uses` names that file, when a ref is the package self glyph `<dir>#`, or when a later card names the file under `Edit` or `Extends`.
Otherwise it is one blocking `doc-go-unlisted` finding per card and package, with "way forward: list it under Edit when the card changes what it says, under Extends when it only adds to it, under Uses when it stays as is and no later card targets it, leave it to the later card that edits it, else list it under Edit".
The closing "else list it under Edit" covers a `doc.go` a later card deletes or moves, where `uses-later-target` refuses the Uses listing and no later Edit or Extends covers it.
A `Uses` listing records that the writer read the doc and found nothing the card falsifies.
Bound, stated in the spec: a `doc.go` under `Extends` or `Uses` whose text the card falsifies, or one a later card edits for another reason, passes and stays with review; a test-only card, `Prosa` and `Custom` groups, and `docs/overview.md` are not checked; a card creating a test helper by a handle whose name begins with none of the test prefixes counts as non-test code and lists the `doc.go`.

Tests:
`TestValidatePlan_CardChecks` gains D8 rows over a temp worktree holding `internal/d/doc.go`, `internal/d/d.go` with `func Do()`, `internal/d/template.yaml`, `internal/d/twice_a.go` and `internal/d/twice_b.go` both declaring `func Twice()` under no build constraint, and `internal/d/d_test.go` with `func TestDo` and a helper `func fixture()`:
a card editing `internal/d#Do` alone fails with `doc-go-unlisted` naming the card, the package and the way forward; a card editing `internal/d#Twice` alone, an ambiguous answer, fails; a card creating the helper handle `plan:internal/d#newFixture` fails; listing `internal/d/doc.go` under Uses with no later editor passes; a later card editing `internal/d/doc.go` passes, and so does a later card extending it; a card targeting `internal/d#` passes; a card deleting `internal/d/doc.go` passes; a card editing only `internal/d#TestDo` passes, and so does one editing only `internal/d#fixture`, which is declared only in a test file; a card creating the file `internal/d/new_test.go` alone passes, and so does one creating `plan:internal/d_test#TestNew`; a card creating `plan:internal/d#TestNew` passes; a card renaming `internal/d#fixture` to `plan:internal/d#fixture2` passes, through the old side alone; a card renaming `internal/d#TestDo` to `plan:internal/d#TestDoes` passes; a card whose only group naming `internal/d/d.go` is `Prosa` passes, and so does one whose only such group is `Custom`; a card creating `plan:internal/n#New` in a package with no `doc.go` passes; a card editing only the non-Go file `internal/d/template.yaml` passes; and a done card and a `language: csharp` plan pass.

Docs: `contracts/specs/loom-plan-spec.md` gains the `doc-go-unlisted` row with its rule, test-code classification, the rule that a non-Go file target is skipped, way forward and bound, and its card rules say a card changing a package's code covers its `doc.go`.
`contracts/stencils/loom/loom-template-plan.md` gains, in the step that writes the cards, one sentence naming `seam-test-unlisted`, `test-tier-unstated` and `doc-go-unlisted` as gate checks whose rules `loom-plan-spec.md`'s validation checks hold, restating none.
`internal/loomshed/doc.go` and `internal/loomcli/doc.go` list the three card checks the plan gates and `validate-plan` add, and `validatePlanCmd`'s Long help adds `doc-go-unlisted` to the sentence card 7 rewrote, the one starting "Every mode but --require-approved also runs", among the checks it lists, outside the clause on what `rename-tool-missing` and `pattern-changes-mismatch` read.

**ImpactSummary:** one appended check inside `planContextFindings`, whose callers do not change; the new refusal fires on a go plan whose card edits a documented package's code without covering its `doc.go`.

**Commit:** `9: Refuse a card that changes a documented package without its doc.go`
