# Card 3 — The machine module and `lym machine name` showing the name

**Create:**
- `internal/machinecli/doc.go`
- `internal/machinecli/cli.go`
- `internal/machinecli/deps.go`
- `internal/machinecli/testmain_test.go`
- `internal/machinecli/cli_test.go`
- `plan:internal/machinecli#Command` -> `func Command() *cobra.Command`
- `plan:internal/machinecli#RunCLI` -> `func RunCLI(out io.Writer, args []string) int`
- `plan:internal/machinecli#RunCLIIn` -> `func RunCLIIn(cwd string, out io.Writer, args []string) int`
- `plan:internal/machinecli#deps` -> `type deps struct`
- `plan:internal/machinecli#productionDeps` -> `func productionDeps() deps`
- `plan:internal/machinecli#newCommand` -> `func newCommand(d deps) *cobra.Command`
- `plan:internal/machinecli#nameCmd` -> `func nameCmd(d deps) *cobra.Command`
- `plan:internal/machinecli#showName` -> `func showName(d deps) (map[string]any, error)`
- `plan:internal/machinecli#TestMain` -> `func TestMain(m *testing.M)`
- `plan:internal/machinecli#TestNameShow` -> `func TestNameShow(t *testing.T)`

**Edit:**
- `cmd/lym#newRoot`
- `cmd/lym/seamsignature_test.go`
- `cmd/lym#excludedModules`

**Extends:**
- `docs/overview.md`
- `pattern/PATTERN-cli-cobra.md`

**Uses:**
- `internal/machinename#FilePath`
- `internal/machinename#ReadFile`
- `internal/machinename#CheckName`
- `plan:internal/machinename#HostnameName`
- `internal/cleanupcli#Command`
- `internal/cleanupcli#RunCLIIn`
- `internal/clihelp#Execute`
- `internal/clihelp#ExecuteIn`
- `internal/clihelp#SetExit`
- `internal/output#Ok`
- `internal/output#ErrFields`

**Intent:** Add the module `machine` (D5) with its one verb showing the name, and mount it.

`internal/machinecli` is a new package with no engine package of its own.
`Command` returns `newCommand(productionDeps())`: the group `machine`, with `clihelp.GroupRunE` and no audience, holding the verb `name`.
`RunCLI` and `RunCLIIn` follow `internal/cleanupcli`'s pair.
`nameCmd` has `Use: "name [<name>]"`, the operator audience and `cobra.MaximumNArgs(1)`; its `RunE` checks `clihelp.ShouldAbort` first.
On this card a call with a name is refused: `lym machine name <name> cannot set the name yet; way forward: edit the machine file by hand`, naming the file's path when one resolves.
Card 5 replaces that refusal with the set path, and no test pins its text.

`deps` holds the seams of D8 this card needs, and `productionDeps` fills them:

- `FilePath func() (file, lock string, ok bool, err error)`, from `machinename.FilePath`;
- `HostnameName func() (string, error)`, from `machinename.HostnameName`.

`showName` reads through `deps` alone and writes nothing, the lock file included.
It never calls `machinename.Name` or `machinename.Names`, which read the developer's file past the seam.
It answers these fields:

- No config directory resolves, or no file exists at the path: `name` is the hostname rule's name, `source` is `hostname`, `former_names` is empty;
  `file` is the path when one resolves and is left out otherwise.
- A file that decodes and whose `name` passes `machinename.CheckName`: `name`, `source` `file`, `file`,
  and `former_names`, the non-empty entries of `names` other than `name`, in the file's order.
- A bad file, one that does not read, does not decode or holds an invalid `name`:
  an error envelope through `output.ErrFields` carrying the file's error, the field `file` and the way forward `lym machine name <name>`, and a non-zero exit.
  It prints no name and never falls back to the hostname.
- A relative variable, `FilePath`'s error: that error as the refusal, which already names its way forward.

`TestMain` in `internal/machinecli/testmain_test.go` is `untagged` support: `gitkit.HermeticGitEnv` and `tmuxkit.Main`, as `internal/cleanupcli`'s.
`TestNameShow` in `internal/machinecli/cli_test.go` is tier0 and parallel, one table over `newCommand` with fake `deps` and a file under `t.TempDir()`:
no directory, no file, a valid file with former names and an empty `names` entry, a file that does not decode, a file with an invalid name, and a relative-variable error.
Each row asserts the whole envelope and the exit code, and that the file's bytes and its directory's listing are unchanged.

`cmd/lym#newRoot` mounts `machinecli.Command()` and adds `machine` to the Available modules list of its `Long`.
`cmd/lym/seamsignature_test.go` gains `machinecli.RunCLI` and `machinecli.RunCLIIn` in its two lists.
`cmd/lym#excludedModules` gains `machine`, with the reason that its verb writes the per-user machine file and pushes owner writes to every recorded hub,
which no sandbox scenario may do, and that `internal/machinecli`'s own tests cover it.

`docs/overview.md` gains the package line for `internal/machinecli` in its tree and a `machine` module entry beside the other modules.
`pattern/PATTERN-cli-cobra.md`'s Package naming list gains one bullet: `machinecli` has no engine package of its own,
and calls the functions of `machinename`, `boardengine`, `fabricengine`, `hubregistry` and `lymcwd`.
The decision record declares no `PATTERN-cli-cobra` change; the entry's overview line stays as it is, and the overview's Shared Decisions states this assumption.

`internal/machinecli/doc.go` documents the package: the verb's show form, its seams and that it has no engine.

**ImpactSummary:** Adds the `lym machine` subtree and its one mount in the root; no other command changes, and the three `cmd/lym` oracles gain the new module.

**Commit:** 3: machine: add lym machine name, showing the machine name
