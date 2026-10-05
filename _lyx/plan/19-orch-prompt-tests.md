# Card 19 — Orch stencil wording checks join the claims table

**Edit:**
- `internal/orchengine/prompt_test.go`
- `contracts/stencils/claims_test.go`

**Intent:**
Finding F2, applied to the other new tests of the same kind: `internal/orchengine/prompt_test.go` pins stencil wording in three tests that each check one kind of phrase.

- The heading lists of `TestRenderRoleFile_WritesEveryTheme` and `TestRenderNoteTemplateFile_WritesItsHeadings` become must claims: on the `orch-template-role.md` row card 17 added, and on a new `orch-template-note.md` row.
- `TestOrchStencils_NameNoScribeHandoff` becomes a `scribe:handoff` mustNot claim on a row for each orch stencil it scanned, adding the rows that do not exist yet; the test is deleted.
- What stays in `prompt_test.go` is the render behavior: one table-driven test over `RenderRoleFile` and `RenderNoteTemplateFile` asserting that each writes its file, creating the missing directory, without the stencil's leading comment.

**ImpactSummary:** test-only: three orch prompt tests become one render table test plus claims-table rows.

**Commit:** 19: orch: stencil wording checks join the claims table
