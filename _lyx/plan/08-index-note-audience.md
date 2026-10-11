# Card 8 — index-note-audience

**Create:**
- `plan:internal/clihelp#IndexNoteAudienceAnnotation` -> `const IndexNoteAudienceAnnotation = "lyx.index-note-audience"`
- `plan:cmd/lyx#indexNoteProblems` -> `func indexNoteProblems(cmd *cobra.Command) []string`

**Edit:**
- `internal/clihelp#IndexNoteAnnotation`
- `internal/clihelp#indexLines`
- `internal/clihelp#groupLine` -> `func groupLine(cmd *cobra.Command, audience string) string`
- `internal/clihelp#leafLine` -> `func leafLine(cmd *cobra.Command, audience string) string`
- `internal/clihelp#withNote` -> `func withNote(cmd *cobra.Command, audience, line string) string`
- `internal/clihelp/exec.go`
- `internal/clihelp#TestAnnotationLiterals`
- `internal/clihelp#TestRenderIndex_FormsAndFiltering`
- `internal/boardcli#Command`
- `internal/boardcli/doc.go`
- `cmd/lyx#formProblems`
- `cmd/lyx#TestCLIForm_EveryCommand`

**Uses:**
- `internal/clihelp#Audiences`
- `internal/clihelp#AudienceAnnotation`
- `internal/clihelp#RenderIndex`

**Intent:** Decision D7.
The `board` group's payload legend prints under every audience, including the internal index where no payload-taking board verb lists.
Add `IndexNoteAudienceAnnotation`, whose value is one of `Audiences`, with a doc comment stating the rule: the command index appends a command's index note only when rendering the audience the annotation names, and a note without it never prints; say the same on `IndexNoteAnnotation`'s comment and in the package doc in `exec.go`.
Thread the audience being rendered from `indexLines` through `groupLine` and `leafLine` into `withNote`, which appends the note only when `cmd.Annotations[IndexNoteAudienceAnnotation] == audience`.
In `boardcli.Command`, add `clihelp.IndexNoteAudienceAnnotation: clihelp.AudienceOperator` beside the group's index note, the audience of every payload-taking board verb, and say so in `internal/boardcli/doc.go`'s sentence about the index note.
Tests: add the new key to `TestAnnotationLiterals`; in `TestRenderIndex_FormsAndFiltering`, give the `board` fixture's note the operator audience so the operator expectation keeps the legend, add a second noted group whose note carries no audience and assert its line prints without the note under every audience, and a third whose note names `role` and assert the note prints under `role` only.
Guard over the live tree: `indexNoteProblems` reports a command carrying an index note whose note-audience annotation is absent or outside `Audiences`; add it to `formProblems`'s check list and two synthetic rows to `TestCLIForm_EveryCommand` (a noted command without the audience, wanting "note audience"; one with a value outside the set, wanting "outside"), so the walk of `newRoot()` fails on such a command.
Bound: the change can only drop a note from an index, never add one.

**ImpactSummary:** Three unexported clihelp renderers re-signed with callers inside the same file, one annotation added on the board group, and one new form check in the CLI tree test.
