# Card 17 — config-hubwide-write-refusal

**Create:**
- `internal/configcli#hubWriteRefusal`

**Edit:**
- `internal/configcli#configDirs`
- `internal/configcli#dispatch` -> `func dispatch(l *lyxcwd.Location, out io.Writer, args []string, edit configengine.EditorFunc, sync syncFunc, commit hubCommitFunc, printOnly bool, setFlags []string, hubWriteGuard func() (string, error)) int`
- `internal/configcli#runConfig`
- `internal/configcli#runMenu`
- `internal/configcli#setModule`
- `internal/configcli#editOne`
- `internal/configcli/configcli_test.go`
- `internal/configcli/configcli_integration_test.go`
- `internal/configcli/reconcile_integration_test.go`
- `internal/configcli/doc.go`
- `contracts/specs/refusal-spec.md`

**Uses:**
- `internal/fabricengine#IsPrimeWorktree`
- `internal/agentname#Parse`
- `internal/agentname#StrandNameEnv`

**Intent:**
Make `lyx config` refuse a write to any `HubWide` module from a task pair or from a session spawned in one (D10), before `landing` becomes hub-wide in the next card, so that move cannot turn a pair-local hole into a hub-wide one.
`hubWriteRefusal` returns a refusal when `l` is not the hub's prime (`fabricengine.IsPrimeWorktree`), or when `strandName` parses through `internal/agentname` as a name carrying a slug, wherever the worktree resolves; the hub orch's slug-free name and an empty name pass.
The message names the way forward: the operator or the hub orch runs the write from the prime.
`configDirs` gains a `hubWriteGuard func() (string, error)`, called only on a hub-wide write, so a print or a per-worktree write spawns no git; `setModule` and `editOne` call it on their `HubWide` branch and, on a refusal, return it as the JSON error with the hub file unchanged and no editor opened.
`dispatch` takes the guard and stores it on its dirs; `runConfig` passes a guard over the resolved location and `os.Getenv(agentname.StrandNameEnv)`, and `runMenu` sets the same guard on the menu's dirs.
Existing `dispatch` calls in the tests pass a nil guard, which allows the write.
`configcli_integration_test.go` gains a table over a `hubforge` hub: `lyx config board --set …` in a task pair is refused naming the prime with the hub file unchanged; the same write from the prime under a slug-bearing `LYX_STRAND_NAME` is refused; from the prime with no strand name, and under the hub orch's slug-free name, it succeeds.
Bound: this guards a mistaken write, not a determined agent; a direct file write to the board dir's config, an environment cleared without naming the variable, and an unset hidden behind a script still pass (D10's Bound).
`doc.go` states the refusal and its bound.
`refusal-spec.md` gains the row in the loom table, since loom-run agents reach it, and the Out-of-scope paragraph's exception list names it.

**ImpactSummary:** The config CLI's dispatch gains one parameter with one production caller and three test files; hub-wide writes from pairs and pair-spawned sessions are refused.
