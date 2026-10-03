# Card 12 — per-cli-copies

**Edit:**
- `internal/burlercli/cli_test.go`
- `internal/fabriccli/cli_test.go`
- `internal/loomcli/cli_test.go`
- `internal/battencli/cli_test.go`
- `internal/orchcli/cli_test.go`
- `internal/idecli/cli_test.go`
- `internal/shuttlecli/cli_test.go`
- `internal/boardcli/cli_unit_test.go`
- `internal/quarrycli/cli_test.go`
- `internal/webstercli/cli_test.go`
- `internal/reedcli/cli_test.go`
- `internal/shedcli/cli_test.go`

**Uses:**
- `cmd/lyx/clitree_test.go`

**Intent:** Delete the per-CLI copies card 11's walk now covers (D5): every `TestRunCLI_NoArgs`/`NoArgsListsSubcommands`, `TestRunCLI_UnknownSubcommand` (and its `_NoGitRepoNeeded` and JSON-envelope variants), `TestCommand_EveryCommandHasShort`, `TestCommand_EveryVerbHasNonEmptyShort` and `TestCommand_ChildrenHaveNonEmptyShort` in these packages, plus the exact-verb-set tests (`webstercli`'s `TestCommand_AllEightSubcommandsRegistered`, `loomcli`'s and `battencli`'s `TestCommand_RegisteredVerbs_ExactSet`).
Delete case by case: a test that also pins one specific behaviour beyond the walk (for example `boardcli`'s hidden aliases in `TestCLIAliasesHiddenWithShort`, `fabriccli`'s `TestCommand_PullAndMergeInDeclareShort`) stays, and a test whose comment cites a finding or crucible round keeps its assertion (D21).
Imports and helpers left unused are removed.
What this lets through is D5's bound, restated in card 11: verb presence and `Short` are now checked only by `go test ./cmd/lyx`.
Every deleted test is named in the commit body for the count accounting.

**ImpactSummary:** Test deletions only across the `*cli` packages; production code and the `cmd/lyx` walk are untouched.
