# Card 41 — old-policy-citations

**Edit:**
- `loomshed/fixture_test.go`
- `loomcli/parity_test.go`
- `loomcli/validate_test.go`
- `websterengine/template_test.go`

**Intent:**
Remove the last citations of the reversed "don't share test helpers" policy (D1) and close the test-count accounting.

At the plan-fixture sites left to #327 (`writeGlyphRepoFixture` in `loomshed/fixture_test.go`; `glyphRepoPlanFixture`, `writeGlyphRepoForParityTest` and `reworkParityFixture` in `loomcli/parity_test.go`; `glyphPlanFixture` and `writeGlyphRepoForCLITest` in `loomcli/validate_test.go`), only the doc comment changes: it keeps its pointer to the copy's origin ("duplicated from internal/planglyph/repo_test.go's writeFixtureRepo") and drops "per the duplicate-test-helpers-rather-than-share-them Shared Decision".
The code of those fixtures is untouched; `test-suite-derived-guards` replaces them.
`websterengine/template_test.go`'s `requireContains` comment drops "Kept package-local rather than shared, since test-helper packages are deliberately not shared across modules."

Then `grep -rni 'duplicate-test-helpers\|not shared across modules\|deliberately not shared' internal cmd tools` must find nothing; reword any further hit the same way.

Record the after leaf test count (see the overview's Shared Decisions) in this card's commit body, with the difference from card 1's baseline and a pointer to the cards whose commit bodies itemise the intended deletions and consolidations; any drop not itemised there is a defect to fix before this card commits.

**ImpactSummary:** Comment-only edits in four test files; no code or test changes.
