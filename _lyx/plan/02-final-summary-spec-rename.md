# Card 2 — final-summary-spec-rename

**Rename:**
- `contracts/specs/final-summary-spec.md` -> `contracts/specs/loom-final-summary-spec.md`

**Create:**
- `plan:contracts/specs#LoomFinalSummarySpec` -> `var LoomFinalSummarySpec []byte`

**Edit:**
- `contracts/specs#entries`
- `contracts/specs#TestRegistry_NamesAndDefaults`
- `contracts/specs#TestRegistry_RelPathPlacesBothUnderLoomFamily`
- `contracts/specs/webster-spec.md`
- `contracts/stencils/darn/darn-template-write.md`
- `contracts/stencils/landing/landing-template-describe.md`
- `docs/overview.md`
- `internal/darnengine#TestDarnSpec`
- `internal/cliwire#requireSpecsSeeded`
- `cmd/lyx#TestStencilSeeding_HubScenario`

**Uses:**
- `contracts/specs#LoomPlanSpec`
- `contracts/specs#Registry`

**Intent:** Decision D2.
The darn stencil names a spec path that exists only in loomyard's own repository; PATTERN-stencil-ownership routes a normative spec through the deployed specs directory instead.
Rename the spec with `git mv` per the Rename mechanic, keeping its content byte-identical (it opens with a `# Heading`, which the stamp round-trip test relies on).
In `specs.go`, declare `LoomFinalSummarySpec` with a `//go:embed loom-final-summary-spec.md` directive and a doc comment modelled on `LoomPlanSpec`'s, and append `{"loom-final-summary-spec", &LoomFinalSummarySpec}` to `entries` after the plan spec, so `stencilstore.RelPath` places it at `loom/loom-final-summary-spec.md` and seeding deploys it beside the plan spec.
Update `TestRegistry_NamesAndDefaults`'s `want` slice to both names in registry order and add the `{"loom-final-summary-spec", "loom/loom-final-summary-spec.md"}` row to `TestRegistry_RelPathPlacesBothUnderLoomFamily`.
Move every reference to the old name to the new one: the darn stencil's Step 4 now reads `{{.specs_dir}}/loom/loom-final-summary-spec.md` and keeps pointing at the spec without restating its format (PATTERN-producer-pointer-rule); the landing describe stencil's parenthetical mention becomes `loom-final-summary-spec.md`; `webster-spec.md`'s two links and `docs/overview.md`'s contract-doc list, summaryparser bullet and See-also link all name and link the new file so PATTERN-markdown-link-integrity holds.
Update `TestDarnSpec`'s first-spawn `wantInPrompt` to `filepath.ToSlash(in.SpecsDir) + "/loom/loom-final-summary-spec.md"`, and add `"loom-final-summary-spec"` to the registered-name lists `requireSpecsSeeded` and `TestStencilSeeding_HubScenario`'s `specNames` iterate, so a seeded specs dir is asserted to carry the new file.
After the edits, grep the repository for `final-summary-spec.md` and confirm the only hits spell the new name.

**ImpactSummary:** One file rename, one new embedded registry entry, and name updates in two stencils, two spec docs, the overview and four tests; `lyx stencil list` gains one spec name (risk R3).
