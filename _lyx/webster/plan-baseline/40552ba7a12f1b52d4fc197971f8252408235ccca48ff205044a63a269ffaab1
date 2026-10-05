# Card 13 — lyx selfreport refuses an unknown subcommand

**Edit:**
- `internal/selfreportcli#Command`
- `internal/selfreportcli/cli_test.go`

**Intent:**
The `selfreport` group gets `RunE: clihelp.GroupRunE` (D10), so `lyx selfreport bogus` exits 1 with the shared unknown-subcommand envelope and bare `lyx selfreport` still lists `create`, exit 0.

Tests, untagged: `RunCLI` with `bogus` exits 1 with an envelope containing `unknown subcommand`; with no argument it exits 0 and names `create`.

**ImpactSummary:** the bare and unknown-argument behaviour of the selfreport group; `create` is unchanged.

**Commit:** 13: lyx selfreport refuses an unknown subcommand
