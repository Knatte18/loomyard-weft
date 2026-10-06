# Card 21 — cli-ide-prune

**Edit:**
- `internal/clihelp#`
- `internal/idecli#`
- `internal/ideengine#`
- `internal/quarrycli#`
- `internal/selfreportcli#`
- `internal/selfreportengine#`
- `internal/vscode#`

**Intent:**
Run the prune procedure (Shared Decisions, `prune-procedure`) on the test files of the small CLI and IDE packages listed above.

`internal/selfreportcli`'s `TestGroup_BareListsCreate` is a `failed` entry (Decision 8).
`internal/idecli`'s `cli_test.go` and `internal/ideengine`'s `integration` files hold this group's tier-move and scenario-merge leads.

**ImpactSummary:** Test files of the small CLI and IDE packages only; no production change.

**Commit:** 21: Prune CLI and IDE package tests
