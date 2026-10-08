# Card 9 — shuttle-state-cli

**Create:**
- `internal/shuttlecli/state.go`

**Edit:**
- `internal/shuttlecli/cli.go`
- `internal/shuttlecli/cli_test.go`
- `tools/sandbox/SANDBOX-SHUTTLE-SUITE.md`
- `docs/overview.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/shuttleengine#ReadSessionStates`
- `internal/shuttleengine#RunSessionState`

**Intent:**
Add the read-only verb `lyx shuttle state` (D6) under the existing `shuttle` cobra subtree.

It prints one JSON envelope listing, for every run `ReadSessionStates` returns for the current worktree's run-directory root: the strand name, the strand guid, the state, the cause, the detail (an `api-error`'s text), the outstanding tasks of a `busy` `background`, `since` (RFC 3339, empty when unknown), and the history, each entry with the same fields.
`--strand <name>` narrows the list to the run whose strand name or guid equals the value; a value that matches no running record prints an empty list, since a finished run is not an error.
It passes `time.Now()` as the reading time, so a call-time state's `since` is the call (D3).
It writes no file and changes no run or session.
It has a `Short`, a `Long` naming the states and that the model is in shadow mode, and JSON errors like its siblings.

The `shuttle` group's `PersistentPreRunE` already resolves the layout and the shuttle config; `shuttleCLI` keeps the config, the anchor path and the engine it built, so `state` reads through them without re-resolving geometry (`PATTERN-cli-cobra`, `PATTERN-told-geometry`); it builds no reed engine use of its own.

Tests: `cli_test.go` gains one test that runs `state` with `RunCLIIn` in a git worktree fixture whose run-directory root holds a running record with a stamped Stop line and its output file, and asserts the envelope's run, state, cause and history, then `--strand` with a non-matching name returning an empty list; the existing positional-argument test gains `state` with a stray argument.
`cmd/lyx/testdata/test-budget.yaml`: the `internal/shuttlecli` row rises by 1, for the one new top-level function.

Docs: `cli.go`'s module comment lists `state`; `docs/overview.md`'s shuttle entry and module list read `lyx shuttle run|interrupt|send|state` and say what `state` shows; `SANDBOX-SHUTTLE-SUITE.md` gains a scenario, covering `shuttle`, that starts an interactive run, reads `lyx shuttle state` while the agent works, after its held turn end and after `--strand` narrowing, and checks the printed states against what the pane shows.

**ImpactSummary:** Adds one read-only shuttle CLI verb and keeps the resolved config and engine on the shuttle CLI receiver; existing verbs are unchanged.
