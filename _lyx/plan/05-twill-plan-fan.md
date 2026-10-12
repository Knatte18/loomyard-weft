# Card 5 — twill-plan-fan

**Edit:**
- `internal/burlerengine/template.yaml`
- `internal/burlerengine#TestConfigTemplate_DecodesAndIsSelfConsistent`

**Intent:**
D8: the embedded `burler.yaml` template gains four lenses and the fan `twill-plan` listing them in this order: `criteria-coverage`, `row-reachability`, `forced-edits`, `test-economy`.
Each lens text is the record's D8 text verbatim, wrapped as a YAML block scalar like the existing lenses:

```text
criteria-coverage: Focus on the plan against the decision record: list every decision and every acceptance criterion, and for each find the card whose Intent carries it and the planned test row that would fail if it were broken. A criterion the plan argues holds without a test, waives or defers is a finding, and so is a decision, failure exit or branch the record names that no row reaches. Apply the Plan-Review rubric's "Fidelity to the answer key". Report anything else you notice too — emphasis, never exclusion.
row-reachability: Focus on whether each planned test row can do what the card says: open every test function a card extends or names, and the fakes, fixtures and helpers it runs on, and check that the row's input reaches the branch it claims (the fake can produce that state at that point of the sequence), that its assertion would fail if the code were wrong, and that each new fake field or helper the row needs is listed on the card. Apply the Plan-Review rubric's "Reachability of a prescribed refusal" and "Verify coverage". Report anything else you notice too — emphasis, never exclusion.
forced-edits: Focus on what each change forces outside the members a card names: for every member a card changes, find its callers, the tests that assert its output or wording, and the comments, doc files and stated reasons (such as a `//testtiming:keep` reason) that describe it, and report each one the change makes wrong that no card lists. Check that each `Create` handle's package is the package clause of the file the Intent puts it in, an external `_test` package included, and that each listed glyph is the one `lyx quarry` gives. Apply the Plan-Review rubric's "Member glyphs on an `Edit` card" and "The re-sign arrow". Report anything else you notice too — emphasis, never exclusion.
test-economy: Focus on test economy: for every new top-level test and every new row, open the test file it lands in and the existing tests of the same function, and report a new top-level test that should be a row of an existing test, a row that repeats a behavior an existing test already reaches, and a new top-level test that its package's row in `cmd/lyx/testdata/test-budget.yaml` does not budget. Apply the Plan-Review rubric's "Test economy". Report anything else you notice too — emphasis, never exclusion.
```

The existing `plan` fan is unchanged, and so is every other fan and lens.
The template's lens names must not collide with an existing lens; none of the four exists today.

`TestConfigTemplate_DecodesAndIsSelfConsistent` raises its lens count by four and its fan count by one, adds `"twill-plan": 4` to the entry-count map, and adds `"twill-plan": {"criteria-coverage", "row-reachability", "forced-edits", "test-economy"}` to the order map it resolves through `ResolveFan(Config{}, …)`; acceptance: "The embedded burler template resolves `twill-plan` to the four lenses."
Its existing per-lens checks (non-empty text, the emphasis line ending every lens but `generic`, no "ignore ") cover the new texts.

**ImpactSummary:** the embedded template gains one fan and four lenses; no configured fan changes, and a hub picks them up only when card 9's twill writer resolves `twill-plan`.

**Commit:** `5: Add the twill-plan fan and its four lenses to the burler template`
