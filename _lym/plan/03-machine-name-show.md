# Card 3 — machine-name-show

**Create:**
- `internal/machinecli/doc.go`
- `plan:internal/machinecli#Command` -> `func Command() *cobra.Command`
- `plan:internal/machinecli#RunCLI` -> `func RunCLI(out io.Writer, args []string) int`
- `plan:internal/machinecli#RunCLIIn` -> `func RunCLIIn(cwd string, out io.Writer, args []string) int`
- `plan:internal/machinecli#newCommand` -> `func newCommand(d deps) *cobra.Command`
- `plan:internal/machinecli#deps` -> `type deps struct`
- `plan:internal/machinecli#productionDeps` -> `func productionDeps() deps`
- `plan:internal/machinecli#current` -> `type current struct`
- `plan:internal/machinecli#readCurrent` -> `func readCurrent(d deps) current`
- `plan:internal/machinecli#runName` -> `func runName(t *testing.T, d deps, cwd string, args ...string) (int, map[string]any)`
- `plan:internal/machinecli#TestMain` -> `func TestMain(m *testing.M)`
- `plan:internal/machinecli#TestName_Show` -> `func TestName_Show(t *testing.T)`

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
- `internal/machinename#File`
- `plan:internal/machinename#FromThisHost`
- `internal/gitkit#HermeticGitEnv`
- `internal/testkit/tmuxkit#Main`

**Intent:**
The new module `machine`, package `internal/machinecli`, mounts `lym machine` with its leaf `lym machine name`, which this card makes show the name (D5, first bullet); card 4 adds the set.

Shape, per `PATTERN-cli-cobra` and modelled on `internal/hibernatecli/cli.go`:
- `Command()` returns `newCommand(productionDeps())`; `RunCLI` and `RunCLIIn` run `Command()` through `clihelp.Execute` and `clihelp.ExecuteIn`, as hibernatecli's do.
- `newCommand(d)` builds the group `machine` (`RunE: clihelp.GroupRunE`, no audience) with the leaf `name` (`Use: "name"`, `cobra.NoArgs`, `clihelp.AudienceOperator`, lower-case `Short`, `Long` under the cap), whose `RunE` checks `clihelp.ShouldAbort` first.
- `deps` holds the seams D8 names; this card's fields are `locate func() (file, lock string, ok bool, err error)` and `hostName func() (string, error)`.
  `productionDeps` fills them with `machinename.FilePath` and `machinename.FromThisHost`.
  Tests build `deps` by hand and run `newCommand(d)` through `clihelp.ExecuteIn`, so each test hands in a file path under its temporary directory and no test reads or writes the developer's machine file.
  A hand-built `deps` sets every field the path under test calls; the show calls only `locate` and `hostName`, so a field card 4 adds stays nil in a show test and is never called there.
- `current` is what `readCurrent(d)` reads: the file and lock paths and whether a directory resolved, the locate error, whether the file exists, the decoded `machinename.File`, the read or decode error, the valid current name (the file's `name` when `machinename.CheckName` accepts it, else the hostname rule's name when no file or no directory, else ""), its source (`file` or `hostname`), the former names (the file's non-empty `names` other than the current name) and the hostname rule's name ("" when `hostName` errors).
  `readCurrent` reads the file through `machinename.ReadFile`, never `machinename.Name`, so card 4 can repair a bad file from it.
  `machinename.ReadFile` returns exists false with a read error and exists true with a decode error; `current` keeps the two apart.

The show, `lym machine name` with no argument, writes nothing and prints `output.Ok` under `machine`: `name`, `source`, `file` (the path, "" with no directory) and `former`.
A file that does not decode or holds an invalid name prints `output.ErrFields` naming the file's error and path and `lym machine name <name>` as its way forward, with no name, and exits non-zero.
A file that exists but does not read prints the same, its way forward naming removing the file or fixing its permissions, since a set refuses that file too (card 4).
A relative variable prints the locate error, which names the variable.
With no file, or no directory, and a `hostName` error, the show prints that error, which names its own way forward (give the machine a hostname, or name it with `lym machine name <name>`), and exits non-zero.

Bound: the show reads only; it opens no hub and writes no file.

Test, tier 0, `TestName_Show`, through `runName` (a helper that runs `newCommand(d)` via `clihelp.ExecuteIn` in `cwd` and decodes the envelope), rows:
`NoFileReadsTheHostnameRule`, `NoDirectoryReadsTheHostnameRule` (file ""), `FileNameWithFormerNames` (`{"name":"osl46","names":["neve","","osl46"]}` shows `osl46`, `file`, former `["neve"]`), `UndecodableFileShowsItsErrorAndPath`, `InvalidNameShowsItsErrorAndPath`, `UnreadableFileShowsItsPathAndRemoval` (a directory at the file's path), `RelativeVariableShowsItsError`, `UnreadableHostnameShowsItsError` (no file, `hostName` errors).
Each row asserts the machine file's bytes, or its absence, are unchanged after the run.
`TestMain` runs `gitkit.HermeticGitEnv()` then `tmuxkit.Main(m)`, as `internal/adoptcli`'s does, since card 4's tests drive git.

Wiring and its pins:
- `cmd/lym/main.go`'s `newRoot` imports `internal/machinecli`, adds `machinecli.Command()` to `root.AddCommand`, and adds `machine` to the `Available modules:` line of `Long`.
- `cmd/lym/seamsignature_test.go` adds `machinecli.RunCLI` and `machinecli.RunCLIIn` to its two lists; `RunCLIIn` belongs there since card 4 reads the cwd it is given.
- `cmd/lym#excludedModules` gains `machine`, why: "name <name> rewrites the per-user machine file and the owners on every hub of the machine record, which no sandbox scenario may do; exercised by internal/machinecli's own tests instead".
- `docs/overview.md` gains the tree line `internal/machinecli/` (the `lym machine name` command, its seams and the owner sweep) and a **machine** module bullet; card 4 completes the bullet with the set.
- `pattern/PATTERN-cli-cobra.md`'s Package naming deviations gain: "`machinecli` calls the functions of `machinename`, `boardengine`, `fabricengine` and `hubregistry`, with no engine package of its own."
- `internal/machinecli/doc.go` documents the package: the verb, its seams and that it is the only writer of the machine file.

**ImpactSummary:** mounts one new module on the root; the root's module walk (`cmd/lym/clitree_test.go`, `registration_test.go`) and the sandbox and seam pins pick it up, and no existing command changes.

**Commit:** 3: machinecli: add lym machine name to show the machine name
