# Card 5 — reed-add-unless-flag

**Edit:**
- `internal/reedcli/add.go`

**Create:**
- `internal/reedcli/unlessname_integration_test.go`

**Uses:**
- `internal/reedengine#Engine.AddStrandUnless`

**Intent:**
Expose the engine's skip as `lyx reed add --unless-name <name>` (D2).
The `add` command calls `AddStrandUnless(spec, unlessName)` in place of `AddStrand`.
On a skip it exits 0 with an `output.Ok` envelope carrying `skipped: true` and an `unless` object holding the live strand's `guid` and `name`;
an add that proceeds keeps today's `guid`/`name` envelope unchanged.
The flag stands alone: it requires neither `--name` nor `--if-absent`.
It is evaluated first, before `--if-absent`'s match, so a live `<name>` strand also suppresses the relaunch `--if-absent` would otherwise give a dead `--name` strand; without a live match, `--if-absent` and `--name` behave as today.
The flag's usage line and the `Long` help gain a paragraph: the add is a successful no-op while this worktree holds a live, visible strand matching `<name>` (role segment or full name, as `--name`); a dead or hidden match does not count; the check runs before `--if-absent`'s, so a skip also suppresses its relaunch; the flag never removes, resizes or refocuses that strand.

New `unlessname_integration_test.go` (`//go:build integration`), against a `hubforge.NewHub` prime with real tmux, skipping through `skipWithoutMultiplexer`, and running `down` in `t.Cleanup`:

- With a live `orch` strand added first (`--name orch` and a long-running `--cmd`), `add --unless-name orch --if-absent --name claude --cmd <long-running>` exits 0, reports `skipped: true` with the orch strand's guid, and leaves `status`'s strand list, the orch pane's height and the session's active pane unchanged (read from the multiplexer's `list-panes` on the hub's socket before and after).
- With no `orch` strand, the same command adds the `claude` strand as today.
- With the `orch` strand's pane killed through the multiplexer, the same command adds the `claude` strand.

**ImpactSummary:** One reed CLI verb gains a flag and a skip envelope; adds without the flag are unchanged.
