# Card 17 — Merriam and recovery prompts carry the edit directive

**Edit:**
- `internal/websterengine#RenderMasterPrompt`
- `internal/websterengine#RenderRecoveryPrompt`
- `contracts/stencils/webster/webster-template-master.md`
- `contracts/stencils/webster/webster-prefix-recovery.md`
- `internal/websterengine/template_test.go`
- `contracts/stencils/claims_test.go`

**Uses:**
- `internal/editdirective#Directive`
- `internal/editdirective#MarkerName`

**Intent:**
Decision 14 for `webster-master` and `webster-recovery`; forks inherit it through Merriam's opening stencil.

`webster-template-master.md` and `webster-prefix-recovery.md` each gain a `{{.edit_directive}}` line directly below `{{.parent_directive}}`.
Their header comments name it as required.
`RenderMasterPrompt` and `RenderRecoveryPrompt` each call `editdirective.Directive(stencilsDir)`, return its error, and fill `editdirective.MarkerName` outside the optional-names slice.

The Master stencil's "What you never do" line "You read files with Read and Grep and write your two contract files with Write; NEVER run a script, an interpreter or a heredoc to read or write a file." keeps only its role-specific half, "You read files with Read and Grep and write your two contract files with Write.".
The script ban it restated is now the edit directive's, and card 21's scan refuses a second statement (`PATTERN-producer-pointer-rule`).
`claims_test.go`'s pin of that line is updated to the shortened text.

Tests: `template_test.go`'s hand-built values maps for the Master and recovery templates gain an `edit_directive` value, so the templates still fill.
Its existing parent-directive rendering test (the one asserting "No parent is recorded") also asserts the rendered Master and recovery prompts contain the directive's `Edit or Write` sentence.

**ImpactSummary:** Two webster renderers fill one more required marker, and one Master sentence loses its restated script ban.

**Commit:** 17: webster: Merriam and recovery prompts carry the edit directive
