```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. I looked for places outside the touched lines that the split leaves wrong.

**Sets found per change**

1. **New spec files `loom-plan-checks-spec` and `loom-plan-csharp-spec`:**
   - Registry consumers all go through `specs.Registry()` and need no per-file edit: `internal/cliwire/standalone.go:122`, `internal/stencilcli/cli.go:90,189`, `cmd/lym/stencilseed.go:138`. Covered.
   - The tests that name specs are subsets or follow the registry and stay valid: `cmd/lym/stencilseed_integration_test.go:69`, `internal/cliwire/cliwire_test.go:586`, `internal/stencilcli/cli_integration_test.go:149`, `internal/stencilstore/stencilstore_test.go:200`. Covered.
   - `specs_test.go` has the new names and `RelPath` rows. Covered.
   - `docs/overview.md:100` lists the new files. Covered.
2. **Check catalogue moved out of `loom-plan-spec.md`:**
   - Stencils (`loom-block-plan-cards`, `loom-block-plan-format`, `loom-rubric-plan-review`, `webster-template-plan-fix`), `internal/planparser/validate.go`, `validate_test.go`, `internal/webstercli/validate.go` and `internal/planglyph/doc.go` all repoint. Covered.
   - Other references to `loom-plan-spec.md` only cite sections that still exist: `Plan vs. schedule`, `Card fields and order`, `Card path resolution: root: and //`, `## Pattern-Departure trailer`, and the verify model section. Nothing outside `docs/research/` still names "Validation checks".
   - The two `tapestry-recipe.yaml` references (lines 347 and 512) call the plan spec the format authority or Card model. They name no moved section. Covered.
   - Covered with a gap: `webster-template-plan-fix.md` step 3 adds only the checks spec, so a fixer on a `language: csharp` plan is not pointed at `loom-plan-csharp-spec.md`. See finding 2.
3. **C# rules moved to `loom-plan-csharp-spec.md`:**
   - `loom-block-plan-overview.md` repoints to it. Covered.
   - No other stencil mentions C#. I grepped `loom-block-plan-glyphs`, `loom-template-plan`, the webster stencils and `loom-rubric-plan-review`.
4. **Part scope and join moved to `loom-parts-spec.md`:**
   - The part-writer, chair and parts stencils point at `loom-parts-spec.md` only for the parts format, and `validate-plan --part` is a command. No stencil or Go comment cites the old plan-spec "Part scope" or "Joined plans" headings. Covered.
5. **PATTERN and commit messages:**
   - The diff declares no PATTERN change and carries no `Pattern-Departure:` trailer, so nothing is owed. Covered.

**Findings**

1. **LOW, `consistency`.** `contracts/stencils/loom/loom-rubric-plan-review.md:49` says `ImpactSummary` "is required for `Edit` and `Delete` only, per the per-type table in `loom-plan-spec.md`". The surviving table at `contracts/specs/loom-plan-spec.md:116` and the field list at line 84 require it for `Extends` too. The diff edited this rubric and cut the old spec prose that said "Edit and Delete only", so the line now contradicts its cited source.
2. **LOW, `design`.** `contracts/stencils/webster/webster-template-plan-fix.md:23-24` tells the fixer to read the plan spec and the checks spec but not `loom-plan-csharp-spec.md`. A check the fixer must fix on a C# plan (`resign-head-mismatch`, `rename-namespace-pair`, `resign-overload-collision`) has its C# way forward only in the C# spec; the checks spec points there only for some of them. This is a dropped read-path the split created.

**Tool calls (1)**
- One combined grep and read: users of `LoomPlanSpec`, `specs.Registry` and `loom-final-summary-spec`, plus the plan-review rubric's `ImpactSummary` lines. I used it to find enumerations of the spec registry that the diff might have missed.
