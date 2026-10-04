# Card 2 — board-alias-removal

**Edit:**
- `internal/boardcli/cli.go`
- `internal/boardcli/cli_unit_test.go`
- `cmd/lyx/clitree_test.go`

**Delete:**
- `internal/boardcli/notes_test.go`
- `internal/boardcli/promotenote_test.go`

**Intent:**
D5: delete the hidden `lyx board notes` alias group and the hidden `lyx board promote-note` verb, whose vocabulary the coming `kind: note` contradicts.

- In `cli.go`, remove `notesCmd` and `promoteNoteCmd` and their `AddCommand` entries.
  `storeVerbs` is now called once, so fold its comment ("built fresh on every call, so the top level and the notes alias group each get their own command instances") down to what it does now; keep the function, since splitting the verb list out of `Command` still reads better.
  Rewrite the file header's sentences about the alias group and `promote-note`.
- `cli_unit_test.go`: delete `TestCLIAliasesHiddenWithShort`; keep `runCLI`, which the integration tests share.
- Delete `notes_test.go` and `promotenote_test.go`.
- `cmd/lyx/clitree_test.go`: delete the `board notes#bare` and `board notes#bogus` entries of `cliTreeFindings`; the stale-entry check would fail on them once the group is gone.

`Board.PromoteNote` loses its last caller here; card 3 deletes it with the tier model it is written in.

**ImpactSummary:** Removes two hidden CLI aliases no prompt, skill or script calls, their tests and two clitree findings entries.
