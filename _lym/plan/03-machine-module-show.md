# Card 3 — machine-module-show

**Create:**
- `internal/machinecli/doc.go`
- `plan:internal/machinecli#Command` -> `func Command() *cobra.Command`
- `plan:internal/machinecli#RunCLI` -> `func RunCLI(out io.Writer, args []string) int`
- `plan:internal/machinecli#RunCLIIn` -> `func RunCLIIn(cwd string, out io.Writer, args []string) int`
- `plan:internal/machinecli#commandWith` -> `func commandWith(depsFor func(cwd string) nameDeps) *cobra.Command`
- `plan:internal/machinecli#nameDeps` -> `type nameDeps struct`
- `plan:internal/machinecli#productionDeps` -> `func productionDeps(cwd string) nameDeps`
- `plan:internal/machinecli#showName` -> `func showName(cmd *cobra.Command, deps nameDeps)`
- `internal/machinecli/testmain_test.go`
- `plan:internal/machinecli#TestMain` -> `func TestMain(m *testing.M)`
- `internal/machinecli/name_test.go`
- `plan:internal/machinecli#TestName_Show` -> `func TestName_Show(t *testing.T)`

**Edit:**
- `cmd/lym#newRoot`
- `cmd/lym#excludedModules`

**Extends:**
- `//cmd/lym/seamsignature_test.go`
- `//docs/overview.md`
- `//pattern/PATTERN-cli-cobra.md`

**Uses:**
- `internal/machinename#FilePath`
- `internal/machinename#ReadFile`
- `internal/machinename#CheckName`
- `plan:internal/machinename#HostnameName`
- `internal/clihelp#Execute`
- `internal/clihelp#ExecuteIn`
- `internal/output#Ok`
- `internal/output#ErrFields`

**Intent:** Add the module `machine`, package `internal/machinecli`, with `lym machine name` showing this machine's name (D5's no-argument form), wired under the root per `PATTERN-cli-cobra`.

`Command` returns `commandWith(productionDeps)`: the group `machine`, a non-invocable group, and its leaf `name [<name>]`, audience operator, runnable from any directory, inside a hub or not.
`RunCLI` and `RunCLIIn` follow `internal/quarrycli`'s shape: an empty cwd runs `clihelp.Execute`, any other `clihelp.ExecuteIn`.
The leaf checks `clihelp.ShouldAbort` first and reports every error through `internal/output`.
`nameDeps` holds the seams D8 names that this card needs:
`FilePath`, filled from `machinename.FilePath`, giving the machine file's path, its lock's path, whether a directory resolves and a relative variable's error;
and `HostnameName`, filled from `machinename.HostnameName`, giving the hostname rule's name.
`productionDeps` fills them for the working directory it is given.

With no argument the leaf runs `showName` and writes nothing.
It reads the file at the seam's path with `machinename.ReadFile` and judges its name with `machinename.CheckName`.
A file with a valid name prints `name`, `source: file`, `path` and `former`, the file's non-empty `names` entries other than the name.
No file, or no directory, prints the hostname rule's name with `source: hostname` and the path when one resolves.
A file that does not read or decode, or holds an invalid name, prints the error and the path through `output.ErrFields` instead of a name, naming `lym machine name <name>` as the way forward, and exits 1;
a relative variable prints its error the same way.

`internal/machinecli/doc.go` documents the module, the verb's show form, the seams D8 names and that `machinecli` has no engine package of its own.
`internal/machinecli/testmain_test.go` is untagged, a support file whose `TestMain` runs `tmuxkit.Main`, with `gitkit.HermeticGitEnv()` first, since a later card's tests spawn git.
`internal/machinecli/name_test.go` is tier0: `TestName_Show` drives `commandWith` with fake deps whose file path lies under `t.TempDir()`.
Its rows cover a valid file, no file, no directory, an undecodable file, an invalid name and a relative variable.
Each row asserts the printed fields or the error and path, the exit code, and that the file's bytes are unchanged.

`cmd/lym#newRoot` mounts `machinecli.Command()` and names `machine` in the root `Long`'s "Available modules" list, which `TestCLITree_EveryCommand` checks.
`cmd/lym#excludedModules` gains `machine`, its reason that the verb writes the per-user machine file and pushes owner commits to every registered hub's origin.
`cmd/lym/seamsignature_test.go` gains `machinecli.RunCLI` and `machinecli.RunCLIIn` in its two lists.
`docs/overview.md` gains the package tree line for `internal/machinecli/` and a **machine** entry in the module list.
`pattern/PATTERN-cli-cobra.md`'s "Package naming" deviations gain `machinecli`, which calls the functions of `machinename`, `boardengine`, `fabricengine` and `hubregistry`, with no engine package of its own.

**ImpactSummary:** Mounts one new module on the root and adds it to the root's module lists; no existing command changes.
