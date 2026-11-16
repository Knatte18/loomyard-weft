# Card 3 — machine-name-show

**Create:**
- `internal/machinecli/doc.go`
- `internal/machinecli/testmain_test.go`
- `plan:internal/machinecli#Command` -> `func Command() *cobra.Command`
- `plan:internal/machinecli#RunCLI` -> `func RunCLI(out io.Writer, args []string) int`
- `plan:internal/machinecli#RunCLIIn` -> `func RunCLIIn(cwd string, out io.Writer, args []string) int`
- `plan:internal/machinecli#deps` -> `type deps struct`
- `plan:internal/machinecli#productionDeps` -> `func productionDeps() deps`
- `plan:internal/machinecli#showName` -> `func showName(d deps, out io.Writer) int`
- `plan:internal/machinecli#TestShowName` -> `func TestShowName(t *testing.T)`

**Edit:**
- `cmd/lym#newRoot`

**Extends:**
- `pattern/PATTERN-cli-cobra.md`

**Uses:**
- `internal/machinename#FilePath`
- `internal/machinename#ReadFile`
- `internal/machinename#CheckName`
- `internal/machinename#File`
- `plan:internal/machinename#HostnameName`
- `internal/output#ErrFields`

**Intent:** Create the module `machine`, package `internal/machinecli`, with `lym machine name` showing the machine name (D5, D8).

`Command` returns the `machine` group, which carries no audience and prints help on a bare call, with one leaf `name`, `Use: "name"`, audience operator, `cobra.NoArgs` on this card; card 4 widens it to `name [<name>]`.
`RunCLI` runs `RunCLIIn` with an empty cwd, and `RunCLIIn` runs the tree with cwd as the working directory, the way `internal/hibernatecli`'s seams do.
Every `RunE` checks `clihelp.ShouldAbort` first, and errors go out as JSON through `internal/output`.

`deps` is the verb's seam (D8), every outside read a field, so a test hands in the file's path and its own hostname name:
`filePath func() (file, lock string, ok bool, err error)`, filled from `machinename.FilePath`,
and `hostnameName func() (string, error)`, filled from `machinename.HostnameName`.
Card 4 adds the hub fields.
`productionDeps` fills them; the cobra leaf builds its deps through a package-level `var newDeps = productionDeps`, which tests replace.

`showName` writes nothing. It prints, through `output.Ok`:
- with a file that decodes and whose name `CheckName` passes: `name`, `source: "file"`, `path` and `former`, the file's non-empty `names` entries other than the name;
- with no file, or no config directory: the hostname rule's name, `source: "hostname"`, the path when one resolves, and no former names;
- with a file that does not read, does not decode or holds an invalid name: no name, the file's error and its path, and the way forward `lym machine name <name>`, through `output.ErrFields`, exiting non-zero;
- with a relative variable: `FilePath`'s error, which names it.
It resolves through `machinename.ReadFile` at the seam's path instead of `machinename.Name`, which reads the developer's own file, so the rule `Name` applies is restated here: file first, hostname only with no file.

`doc.go` states the package: the verb, its seam, and that it writes only the machine file at the seam's path and owners on the hubs the seam lists.
`testmain_test.go` is `untagged` support: it runs the package under `tmuxkit.Main`.
`TestShowName` (tier0) runs `RunCLI` with a temporary file path and a fixed hostname name and asserts the envelope for each case above: a valid file with an empty and a former entry, no file, no directory, an undecodable file, an invalid name and a relative variable; each row also asserts that the file's bytes, or its absence, are unchanged.

`newRoot` adds `machinecli.Command()` to the root, and its `Long` lists `machine` among the available modules.
`pattern/PATTERN-cli-cobra.md`'s Package naming deviations gain: `machinecli` calls the functions of `machinename`, `boardengine`, `fabricengine`, `hubregistry`, `lymcwd` and `state`, with no engine package of its own.

**ImpactSummary:** Mounts one new module under the root; the root's help text gains one module name; no other command changes.
