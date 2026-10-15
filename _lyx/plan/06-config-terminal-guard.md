# Card 6 — config-terminal-guard

**Create:**
- `plan:internal/configcli#noTerminalRefusal` -> `func noTerminalRefusal(module string) string`
- `plan:internal/configcli#stdinIsTerminal` -> `func stdinIsTerminal(in io.Reader) bool`

**Edit:**
- `internal/configcli#dispatch` -> `func dispatch(l *lyxcwd.Location, out io.Writer, args []string, edit configengine.EditorFunc, sync syncFunc, commit hubCommitFunc, printOnly bool, setFlags []string, layer string, hubWriteGuard func() (string, error), terminal bool) int`
- `internal/configcli#menu` -> `func menu(dirs configDirs, in io.Reader, out io.Writer, edit configengine.EditorFunc, sync syncFunc, commit hubCommitFunc, terminal bool) int`
- `internal/configcli#runConfig` -> `func runConfig(ctx context.Context, in io.Reader, out io.Writer, args []string, printOnly bool, setFlags []string, layer string) int`
- `internal/configcli#Command`
- `internal/configcli#runMenu`
- `internal/configcli#runMenuWith` -> `func runMenuWith(t *testing.T, input string, terminal bool, editor configengine.EditorFunc, seed ...string) (int, string, *fakeSyncTracker)`
- `internal/configcli#TestMenu`
- `internal/configcli#TestPrint`
- `internal/configcli#TestDispatchSet`
- `internal/configcli#TestDispatchHubWideBoard`
- `internal/configcli#TestDispatchLayers`
- `internal/configcli#TestConfigOverRealHub`
- `internal/configcli#TestConfigReconcileInGitRepo`
- `internal/configcli/doc.go`

**Intent:**
Decision D6: every editor path of `lyx config` refuses without a terminal and writes nothing.

`stdinIsTerminal(in)` reports whether `in` is an `*os.File` that `golang.org/x/term`'s `IsTerminal` accepts on its descriptor; any other reader is not a terminal.
`noTerminalRefusal(module)` words the refusal: `lyx config opens an editor only from an interactive terminal, and stdin is not one; way forward: "lyx config <module> --print" to read, "lyx config <module> --set key=value" to write`, with `<module>` the module when one is named and left as the placeholder otherwise.

The terminal answer is injected: `dispatch` and `menu` each take a `terminal bool`.
`dispatch` checks it after the `--set` and `--print` branches and before `editOne`, so a `--print` or `--set` call needs no terminal, and a bare `lyx config <module>` on any layer with `terminal` false returns `output.Err(out, noTerminalRefusal(module))` before the layer is dispatched, the hub-write guard is consulted, a file is scaffolded or the editor runs, with no sync and no commit.
`menu` checks it first, before printing the list or reading a line, and refuses with `noTerminalRefusal("")`, so it reads no choice.
`runConfig` takes the command's input, which `Command`'s RunE passes as `cmd.InOrStdin()`, and passes `stdinIsTerminal(in)` to `dispatch`;
`runMenu` passes `stdinIsTerminal(in)`, the menu command's own input.
Both production paths therefore read the same input cobra hands the command, which a test sets with `SetIn`.
The bound: it guards a caller with no terminal, such as an agent's shell; a caller with a terminal and an editor that exits at once is covered by card 5's empty-result abort.
`internal/configcli/doc.go` states the guard, its way forward, and that `--print` and `--set` are unaffected.

Callers re-signed in this card: every `dispatch` call in `TestPrint`, `TestDispatchSet`, `TestDispatchHubWideBoard`, `TestDispatchLayers`, `TestConfigOverRealHub` and `TestConfigReconcileInGitRepo` passes `true`, except the new rows below;
`runMenuWith` takes the terminal answer and passes it to `menu`;
`Command`'s RunE passes `cmd.InOrStdin()` to `runConfig`.

Tests (the package is at its test budget, so every test is a row):
`TestDispatchLayers` gains a `noTerminal` row field passed as `!noTerminal`, and three rows.
- "no terminal refuses the editor and writes nothing": `loom` with `--layer default`, its board and local files seeded and its worktree file absent, a never-called editor and a hub-write guard that fails the test when called; it exits 1 with an error naming `--print` and `--set`, runs no sync and no commit, and its `check` asserts the board and local files are byte-identical to their seeds and the worktree file is still absent.
  The guard is before layer dispatch, so one row stands for every layer, and the failing hub-write guard pins that it also runs before that guard.
- "no terminal still prints": `loom` with `--print`, exit 0 with the seeded bytes.
- "no terminal still sets": `loom` with `--set discussion_timeout_min=60`, exit 0 and the value written.
`TestMenu` gains a `noTerminal` field and the row "no terminal refuses before reading a choice": input `1\n`, never-called editor, exit 1, output naming `--print` and `--set` and no module listing, no sync.
`TestConfigOverRealHub` gains a subtest that drives the real command over the hub fixture's code worktree: `Command()` with `SetIn` set to a regular temp file, run through `clihelp.ExecuteIn` with `loom`, exits 1 naming `--print`, and the worktree's `loom.yaml` keeps its bytes;
and `Command()` with `SetIn` set to a `strings.Reader` holding `1\n`, run with `menu`, exits 1 naming `--print`.
These reach `runConfig`'s and `runMenu`'s wiring and both non-terminal branches of `stdinIsTerminal`, a file that is no terminal and a reader that is no file; its terminal branch needs a real terminal, which no test has.

**ImpactSummary:** Three internal entry points re-signed with `Command`'s RunE and six test callers updated in this card, plus one test helper; `--print`, `--set`, `--resolved` and `reconcile` are untouched, and only the interactive editor paths gain a refusal.
