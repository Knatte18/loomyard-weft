# Card 3 — orch-role-title-form

**Create:**
- `internal/orchengine#decisionNoteTitleMarker`

**Edit:**
- `internal/orchengine#RenderRoleFile`
- `internal/orchengine#orchengineAllowedImports`
- `internal/orchengine#TestRenderFiles_WriteWithoutLeadingComment`
- `internal/orchengine#TestRenderRoleFile_RefusalsWriteNoFile`
- `contracts/stencils#TestStencils_TopLevelMarkers`
- `contracts/stencils/orch/orch-template-role.md`
- `internal/orchengine/doc.go`

**Uses:**
- `internal/alignment#DecisionNoteTitle`

**Intent:** D3: the orch role stencil's decision-note rule renders the title form from `internal/alignment`'s one declaration, so the orch writes the note the snapshot counts.

In `contracts/stencils/orch/orch-template-role.md`, the bullet for a refused `lyx loom decision add` with no decision record (the one ending "with kind note, naming the held task's slug") says instead that the note is titled `` `{{.decision_note_title}} <the decision>` ``, with the held task's slug in place of `<slug>`, because only a note titled that way counts as a decision for the run's alignment agent.
The leading comment's marker sentence names three markers, adding `decision_note_title`, filled with the decision-note title form for the placeholder slug `<slug>`, and says `RenderRoleFile` refuses to render without any of them.

In `internal/orchengine/prompt.go`, `decisionNoteTitleMarker` joins `commandIndexMarker` and `repositoryRulesMarker` in the marker const block.
`RenderRoleFile` checks it after the two existing markers in its required-marker loop, so an operator-edited override without it fails naming the stencil and the marker with the existing way forward, restore the marker or delete the override, and writes no file.
It fills it with `alignment.DecisionNoteTitle("<slug>")`, the literal placeholder because the role stencil is hub-wide.
Its doc comment names the third marker.
`internal/alignment` joins `orchengineAllowedImports`; `internal/alignment` does not import `internal/orchengine`, so no cycle forms.
`internal/orchengine/doc.go`'s role-file paragraph gains that a role stencil without the `decision_note_title` marker fails the render naming the stencil, as for the other two.

Tests:
`TestRenderFiles_WriteWithoutLeadingComment`'s `role` row gains `Decision for <slug>:` in its `want` list, so the rendered role file names the title form.
`TestRenderRoleFile_RefusalsWriteNoFile` gains the row "no decision note title marker", whose override carries `{{.command_index}}` and `{{.repository_rules}}` but not the new marker, wanting the error to name `roleStencilName` and `decisionNoteTitleMarker`; the existing rows keep passing because the new marker is checked last.
`TestStencils_TopLevelMarkers`'s `orch-template-role` row lists `command_index`, `decision_note_title` and `repository_rules`.

**ImpactSummary:** `RenderRoleFile`'s signature is unchanged; its callers `internal/orchcli/start.go`, `resumecontext.go` and `watcher.go` render the new marker with no edit, and only an override lacking the marker newly fails.

**Commit:** 3: Render the decision-note title form into the orch role stencil
