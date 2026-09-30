# Card 35 — batten-goto-teardown

**Edit:**
- `internal/shedverbs/spec.go`
- `internal/shedverbs/goto.go`
- `internal/shedverbs/goto_test.go`
- `internal/battencli/arm.go`
- `internal/battencli/cli.go`
- `internal/battencli/cli_test.go`
- `internal/loomcli/cli.go`
- `internal/shedcli/cli.go`

**Create:**
- `internal/battencli/goto.go`
- `internal/battencli/goto_test.go`

**Uses:**
- `internal/battencli/wire.go`
- `internal/battenrecipe/names.go`
- `internal/shedrun/paths.go`

**Intent:**
Fix PR #321 finding 1's batten half and the goto help texts.
`lyx batten goto <slug> --to Worktree-Teardown`, which the batten help shows as its example, makes teardown reachable from a failed or blocked Run-Shed, breaking the batten recipe header's rule that teardown is unreachable from any failure path.
Card 33's backward-only rule already refuses that move from Run-Shed; this card adds the explicit batten guard the finding asks for, so teardown also stays unreachable when the run's current row is missing from the recipe.

- `shedverbs.Hooks` gains `PreGoto func(ctx context.Context, target string) error`, nil by default and skipped when nil.
  The goto verb body calls it after the `--to` check and before `shedengine.Goto`; a non-nil error is reported verbatim on the error envelope and the status file is not touched.
- `internal/battencli/goto.go` declares batten's `PreGoto` hook, and `arm.go` sets it on the spec.
  For any target other than `Worktree-Teardown` it returns nil.
  For `Worktree-Teardown` it admits the move only when the batten run's own current row is already `Worktree-Teardown`, or when the task worktree's child loom run (`shedrun.StatusFile` of the task worktree's location and `shedrun.SelfRunID`, located as `wire.go`'s `ResolveStatus` locates it) reads `state: done`.
  Otherwise it refuses: `batten goto: Worktree-Teardown is reachable only once the child run is done (child state %q); way forward: "lyx batten step <slug>" drives Run-Shed until the child run finishes, and "lyx batten status <slug>" shows where it is`, the slug spelled out.
  A child status file that does not exist, or a task worktree that does not resolve, counts as not done.
- `battencli/cli.go`: drop the `lyx batten goto some-slug --to Worktree-Teardown` example from both the goto verb's and the parent command's help, replacing it with a backward move (`--to Run-Shed`).
- The goto `Long` texts in `battencli/cli.go`, `loomcli/cli.go` and `shedcli/cli.go` state the admission rule in one sentence each: goto moves a halted run only back, to a row at or before its current row (strictly before when the run is awaiting), and refuses a running run.
  Loom's and shed's examples keep `--to Plan-Write`, a backward move from any later row.

Tests:

- `internal/shedverbs/goto_test.go`: a `PreGoto` hook that refuses leaves the status file byte-identical and puts the hook's message on the error envelope; a nil hook changes nothing.
- `internal/battencli/goto_test.go` (untagged, status files under `t.TempDir()`, both status files written as the verbs write them): a batten run blocked at `Run-Shed` whose child reads `running` → `Worktree-Teardown` refused naming `lyx batten step`; the same with a child reading `done` → admitted by the hook; a run whose current row is `Worktree-Teardown` → admitted; a missing child status file → refused; `--to Run-Shed` → admitted without reading the child.
- `cli_test.go`: the batten help no longer carries `--to Worktree-Teardown`.

**ImpactSummary:** Adds a nil-by-default PreGoto hook to shedverbs.Hooks, filled only by batten; three goto help texts change.
