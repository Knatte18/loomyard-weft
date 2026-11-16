# Card 3 — machine-name-verb

**Create:**
- `//internal/machinecli/doc.go`
- `plan:internal/machinecli#Command` -> `func Command() *cobra.Command`
- `plan:internal/machinecli#RunCLI` -> `func RunCLI(out io.Writer, args []string) int`
- `plan:internal/machinecli#RunCLIIn` -> `func RunCLIIn(cwd string, out io.Writer, args []string) int`
- `plan:internal/machinecli#nameDeps` -> `type nameDeps struct`
- `plan:internal/machinecli#productionDeps` -> `func productionDeps() nameDeps`
- `plan:internal/machinecli#nameResult` -> `type nameResult struct`
- `plan:internal/machinecli#showName` -> `func showName(deps nameDeps) nameResult`
- `plan:internal/machinecli#setName` -> `func setName(deps nameDeps, cwd, name string) (nameResult, error)`
- `plan:internal/machinecli#sweptHubs` -> `func sweptHubs(deps nameDeps, cwd string) ([]string, error)`
- `plan:internal/machinecli#hubOwners` -> `type hubOwners struct`
- `plan:internal/machinecli#pullOwners` -> `func pullOwners(hubPath string) hubOwners`
- `plan:internal/machinecli#checkCollision` -> `func checkCollision(name string, mine []string, hubs []hubOwners) error`
- `plan:internal/machinecli#rewriteFile` -> `func rewriteFile(file machinename.File, decoded bool, hostName, newName string) (machinename.File, []string)`
- `//internal/machinecli/testmain_test.go`
- `//internal/machinecli/cli_test.go`
- `//internal/machinecli/name_integration_test.go`

**Edit:**
- `cmd/lym#newRoot`

**Extends:**
- `pattern/PATTERN-cli-cobra.md`

**Uses:**
- `plan:internal/machinename#HostnameName`
- `internal/machinename#FilePath`
- `internal/machinename#ReadFile`
- `internal/machinename#CheckName`
- `internal/machinename#FromHostname`
- `internal/hubregistry#Live`
- `internal/standalonestate#UserStateDir`
- `internal/lymcwd#Resolve`
- `internal/lymcwd#CwdFrom`
- `internal/fabricengine#BoardDir`
- `internal/fabricengine#NewBolt`
- `internal/fabricengine#Bolt.Pull`
- `internal/fabricengine#NewMutations`
- `internal/boardengine#ReadOwners`
- `internal/state#WriteJSON`
- `internal/clihelp#ShouldAbort`
- `internal/output#Err`
- `internal/hubforge#CopyHub`
- `internal/hubforge#CloneSecond`
- `internal/gitkit#HermeticGitEnv`
- `internal/testkit/tmuxkit#Main`

**Intent:** Add the module `machine`, package `internal/machinecli`, with `lym machine name [<name>]` showing and setting this machine's name (D5), short of the owner sweep, which card 4 adds.
It is a cobra subtree per `PATTERN-cli-cobra`: `Command()` returns the `machine` group (`RunE: clihelp.GroupRunE`, no audience) holding the leaf `name [<name>]` (audience operator, one lower-case `Short`, `cobra.MaximumNArgs(1)`), plus `RunCLI` and `RunCLIIn`.
Every `RunE` checks `clihelp.ShouldAbort` first, and every refusal goes out through `output.Err` with its way forward.
The verb runs from any directory, inside a hub or not.
`cmd/lym#newRoot` mounts `machinecli.Command()` in its `AddCommand` list and adds `machine` to the root `Long`'s "Available modules" list.
`pattern/PATTERN-cli-cobra.md`'s Package naming deviations gain one bullet: `machinecli` calls the functions of `machinename`, `hubregistry`, `fabricengine`, `boardengine` and `state`, with no engine package of its own.

Seams (D8): `nameDeps` carries `FilePath func() (file, lock string, ok bool, err error)`, `Hostname func() (string, error)` and `Hubs func() ([]string, error)`.
`productionDeps` fills them from `machinename.FilePath`, `machinename.HostnameName` and the `HubPath`s `hubregistry.Live` answers for `standalonestate.UserStateDir()`.
The working directory comes from `lymcwd.CwdFrom` on the command's context, which `RunCLIIn` seeds, so tests hand in a directory under their temporary directory.
Tests set a package-level deps value the `RunE` reads; production leaves it at `productionDeps()`.

No argument (`showName`): prints `name`, `source` (`file` when the machine file exists, else `hostname`), `path` and `former` (the file's non-empty `names` other than `name`), and writes nothing.
A file that does not read, does not decode or holds a name `machinename.CheckName` refuses prints that error and the path instead of a name, and exits non-zero.
With no resolvable config directory it prints the hostname rule's name with source `hostname` and no path.

With a name (`setName`), in this order, every refusal writing nothing:
1. `machinename.CheckName` refuses an invalid name with the rule as its way forward.
2. `FilePath` with `ok` false or an error refuses, naming the variables the config directory resolves from (`XDG_CONFIG_HOME` and `HOME`, `APPDATA` on Windows, as `standalonestate.AbsoluteUserConfigDir` reads them) and the way forward of setting one to an absolute path.
3. The file is read through `machinename.ReadFile`, never `machinename.Name`, so a bad file is repaired, not refused.
4. `sweptHubs` lists `Hubs()` plus the hub of the working directory when `lymcwd.Resolve` resolves one, deduplicated by path; a directory in no hub adds none.
5. `pullOwners` pulls each hub's Bolt (`fabricengine.NewBolt(fabricengine.BoardDir(hub)).Pull` with a fresh `NewMutations`), then reads its owners through `boardengine.ReadOwners`.
   A pull that returns an error or a non-empty skip marks the hub checked locally only, and the output names it.
6. This machine's names are the file's valid current name, its non-empty `names` and the `Hostname()` result when that yields one.
   `checkCollision` refuses a name a swept board records as an owner, the prime owner or a beam project's, that is none of them, naming that board's hub and the owner: that name belongs to another machine.
   The refusal names the way forward of choosing another name; for a repair back to a lost name it names writing `{"name": "<name>"}` to the file's path by hand.
7. `rewriteFile` builds the new file: `name` is the new name; the old valid name and the hostname rule's name are each appended to `names` when absent; empty entries are dropped and reported; the new name moves to the end of `names`.
   A file that decodes keeps its `names` and drops an invalid `name`; a file that does not decode is replaced, so `names` holds only the hostname rule's name and the new one, and the output says the old names were lost.
8. The file is written through `state.WriteJSON(file, lock, …)` unless the name is already set and `names` would not change.

The result names the new name, the file path, `names`, whether the file was written, each dropped empty entry and each hub checked locally only.
The verb never writes an owner on this card.

`testmain_test.go` is untagged: `TestMain` runs `gitkit.HermeticGitEnv()` and `os.Exit(tmuxkit.Main(m))`.
`cli_test.go` is tier0 and spawns nothing: `TestNameShow` covers, as table rows, a file source, a hostname source with no file, a bad file printing its error and path, and no config directory; `TestNameRefusesWithoutWriting` covers an invalid name and a config directory that does not resolve, each leaving the file absent or unchanged.
Both hand in the file path under `t.TempDir()`, a fixed hostname and an empty hub list.
`name_integration_test.go` is tier1: `TestNameSet` copies hubs through `hubforge.CopyHub` and seeds owners on their boards, and covers as rows:
- `osl46` over `{"name":"osl-1046","names":["osl-1046"]}` writes `names` `["osl-1046","osl46"]`;
- `neve` refused while a swept board records `neve`, an owner pushed only to origin from a `hubforge.CloneSecond` clone included, leaving the file unchanged;
- a file holding an invalid name keeps its non-empty `names`, adds the hostname rule's name and writes the new name;
- a file that does not decode yields `names` of the hostname rule's name then `osl46`;
- a hand-broken `name` whose `names` ends in `osl46` succeeds while a swept board records `osl46`;
- an empty `names` entry is dropped and reported;
- a hub whose origin is unreachable is named as checked locally only.
Every row hands in its hubs and file path under the test's temporary directory, so no test writes the developer's machine file or a real hub's Bolt.

**ImpactSummary:** A new CLI package mounted in newRoot, one line added to the root Long and one deviation bullet in the cobra background; no existing command changes.
