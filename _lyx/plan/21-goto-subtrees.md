# Card 21 — goto-subtrees

**Edit:**
- `internal/shedcli/cli.go`
- `internal/shedcli/table.go`
- `internal/loomcli/cli.go`
- `internal/loomcli/arm.go`
- `internal/battencli/cli.go`
- `internal/shedcli/table_test.go`
- `internal/shedcli/cli_test.go`
- `internal/loomcli/parity_test.go`
- `cmd/lyx/helptree_test.go`

**Uses:**
- `internal/shedverbs/goto.go`

**Intent:**
Register the fifth generic verb in all three subtrees, so `lyx shed goto`, `lyx loom goto` and `lyx batten goto` exist, each addressing a run the way its siblings do.

- Each subtree's `VerbTexts` literal (`shedVerbTexts`, `loomVerbTexts`, `battenVerbTexts`) gains a `Goto` text: `Use: "goto [<run-id>] --to <producer>"` (loom's spelling matches its siblings' `[run-id]`), a non-empty `Short`, and a `Long` saying it moves a halted run onto the named row as `paused`, records a `goto` history entry that resets that row's segment bounce budget, refuses while a driver holds the run lock and on a done run, and never re-opens a finished run.
  Each `Long` carries an example.
- Each `Command()` destructures the fifth command, sets `Args: cobra.MaximumNArgs(1)`, and adds it; each parent `Long` lists `goto`.
- `internal/shedcli/table.go`: both recipes' `Verbs` gain `goto`.
- `internal/loomcli/arm.go`: `genericShedVerb` accepts `goto`, so the seed-presence refusal covers it.
  `internal/loomcli/cli.go`: `verbUsesLightweightWiring` includes `goto`, since it needs only the status paths and the recipe routing that `loadRouting` fills.
- Batten's arming path already requires an existing seed for every verb but `run` and `step`, so `goto` needs no batten-specific branch.

Tests:

- `table_test.go`: `allGenericVerbs` names five verbs, and both recipes carry all five.
- `cli_test.go` (shedcli): the subtree listing includes `goto`.
- `parity_test.go` (loomcli): the arity loop covers `goto`.
- `helptree_test.go`: the loom, batten and shed rows name `goto`.

**ImpactSummary:** Adds one subcommand to three CLI subtrees and one entry to each recipe's verb gate; no existing verb's behavior changes.
