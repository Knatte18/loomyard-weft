# Card 9 — darn-stencil-and-spec

**Create:**
- `contracts/stencils/darn/darn-template-write.md`
- `contracts/stencils#DarnTemplateWrite`
- `internal/darnengine#darnStencilName`
- `internal/darnengine#darnRole`
- `internal/darnengine#darnSkills`
- `internal/darnengine#noneToken`
- `internal/darnengine#DarnInputs`
- `internal/darnengine#DarnSpec`
- `internal/darnengine/spec_test.go`

**Edit:**
- `contracts/stencils#entries`
- `contracts/stencils#roleOpeningStencils`
- `contracts/stencils#wordingClaims`
- `internal/darnengine/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:** D4's session: the darn writer's prompt and the spec that launches it.
The stencil is the whole instruction set of one session that reads the board entry with `lyx board get {{.slug}}`, reads `PATTERN.md` and the background files its change touches through the rendered pattern directive, edits the repository, runs package-scoped tests through `lyx gate test <pkg>` while working, commits on the task branch and never pushes, updates the docs the repository's `CLAUDE.md` names for its change in the same commits, writes the change description at `{{.description_path}}` in the format `{{.specs_dir}}/final-summary-spec.md` defines without restating it, runs `lyx loom validate-description` before handing off, treats `{{.rejection_findings}}` as this spawn's work when it is not `(none)`, and when `{{.prior_work}}` is not `(none)` continues from the commits already on the task branch and fixes what the token names, never restarting the change.
It states that the handoff runs the full verify and re-prompts with the findings, that a re-prompt is fixed in the same session with new commits, and that the way forward for a hub-wide refusal is the parent, never the operator;
it renders `{{.parent_directive}}`, `{{.edit_directive}}`, `{{.pattern_directive}}` and `{{.friction_directive}}` at its head and asks for no skill and no color (`PATTERN-parent-directive`, `PATTERN-edit-directive`, `PATTERN-role-skills-typed`, `PATTERN-spawn-color`, `PATTERN-producer-pointer-rule`, `PATTERN-no-denied-recovery`).
`stencils.go` embeds it as `DarnTemplateWrite`, registers `darn-template-write` in `entries`, and maps the role `darn` to it in `roleOpeningStencils`;
`wordingClaims` gains a row pinning the board read, the validate-description self-check, the never-push rule and the two token sentences.
`DarnInputs` carries the told values: `StencilsDir`, `SpecsDir`, `WorktreeRoot`, `FrictionDir` (empty for Tier 2 off), `ParentName` (empty renders the no-parent variant), `Slug`, `DescriptionPath`, `RejectionFindings` and `PriorWork`;
`DarnSpec` refuses an empty required field by name, resolves `cfg.Writer` against the registry, renders the pattern directive for `pattern.RoleImplementer` over `WorktreeRoot`, the friction directive for `friction.RoleImplementer` over `friction.NotePath(FrictionDir, "Darn")` (a failure is logged and renders empty, as `ReworkSpec` does), substitutes `noneToken` for an empty `RejectionFindings` or `PriorWork`, fills the stencil through `stencil.FillOptional` with the three optional markers, and returns a non-interactive spec with the description as its one output file, role `darnRole`, segment `segmentcolor.Darn`, `darnSkills`, and the timeout `cfg.WriterTimeoutMin` minutes.
Tests: `spec_test.go` covers, with the stencil kit, the rendered tokens (both `(none)` cases and both filled), the role, segment, skills, output file and timeout, the missing-field refusals and the missing-stencil error;
the stencil registry, marker, citation and parent-directive tests cover the new file with no change.
The package doc gains the spec's paragraph.

**ImpactSummary:** One new stencil in the registry's three tables and one new spec composer in darnengine; no spawning module reaches either yet.
