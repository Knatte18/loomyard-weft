# Card 14 — loom-start-never-attaches

**Edit:**
- `internal/loomcli/start.go`
- `internal/loomcli/bootstrap.go`
- `internal/loomcli/bootstrap_test.go`
- `internal/loomcli/cli.go`
- `internal/loomcli/cli_test.go`
- `internal/loomcli/smoke_starttail_test.go`
- `README.md`
- `docs/overview.md`
- `docs/reference/claude-trust-dialog-repro.md`
- `pattern/PATTERN-cli-cobra.md`

**Delete:**
- `internal/loomcli/handover_test.go`

**Intent:** D13, loom half: `lyx loom start` (and its root alias `lyx start`) never attaches or switches a tmux client, for any caller, with or without `$TMUX`; attaching to reed is only `lyx reed attach` (operator decision 2026-10-06).

`start.go`: after the bootstrap lock is released, the verb always prints the success envelope and returns.
Remove the `$TMUX`/`TMUX_PANE` reads, the `decideHandover` call, the switch-client and attach spawns, the terminal-size read and the `attachHint` constant, and drop the imports only they used.
`--no-attach` stays registered as an accepted no-op, hidden with `MarkHidden` and its usage text reduced to saying it is kept for existing callers (batten passes it in `internal/battencli/wire.go`).
Rewrite the verb's `Short` and `Long` help: the bootstrap launches the driver session and returns its envelope, the terminal handover step and the `--no-attach` paragraph go, and the help names `lyx reed attach` as the way to watch the session.
Update the file comment, which says the verb finally hands the terminal over.

`bootstrap.go`: delete the `handover` type, its constants and `decideHandover`.
Rename `noAttachFields` to `startEnvelopeFields(driver, slug, runID, statusFile string)`, dropping the `attached` and `hint` keys, which only described the skipped tail; the envelope carries the driver, slug, run id and status file.

`cli.go`: the `run` and `step` help texts that contrast themselves with start's terminal handover say instead that they spawn no detached driver.

Docs, per the repo's CLAUDE.md rule for observable CLI behaviour: `README.md`'s quickstart line (`lyx start` hands the terminal to its driver) says the command bootstraps the task and `lyx reed attach` shows its session; `docs/overview.md`'s loom entries stop listing `start` as an interactive-handoff exception and drop the `$TMUX` handover and `--no-attach` sentences; `docs/reference/claude-trust-dialog-repro.md` drops `--no-attach` from its `lyx loom start` step; `pattern/PATTERN-cli-cobra.md`'s interactive-handoff exception list drops `lyx loom start` and `lyx start`.

Tests:

- Delete `handover_test.go`, whose decision table no longer exists.
- `bootstrap_test.go`: drop the default-attach test that calls `decideHandover`; `TestNoAttachFields_PinsSuccessEnvelope` pins `startEnvelopeFields`' key set under its new name.
- `cli_test.go`: the two `--no-attach` lookups stay and also assert the flag is hidden.
- `smoke_starttail_test.go` (tmux tier, the real built binary): `TestSmokeStart_AttachTailAddsNoOperatorStrand` asserts that `lyx loom start` without `--no-attach` now exits 0 with an `ok` envelope, where the attach used to fail without a TTY, and keeps its strand-table assertions; add a case running `lyx loom start` with `TMUX` set to reed's own server socket, with and without `--no-attach`, asserting exit 0, an `ok` envelope and no change to the reed session's attached clients.

**ImpactSummary:** Removes loom start's handover tail and the handover decision; the hidden --no-attach stays for batten, and only help, docs and the start envelope's two attach keys change.
