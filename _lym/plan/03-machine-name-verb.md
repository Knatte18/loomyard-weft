# Card 3 — machine-name-verb

**Create:**
- `internal/machinecli/doc.go`
- `plan:internal/machinecli#Command` -> `func Command() *cobra.Command`
- `plan:internal/machinecli#RunCLI` -> `func RunCLI(out io.Writer, args []string) int`
- `plan:internal/machinecli#RunCLIIn` -> `func RunCLIIn(cwd string, out io.Writer, args []string) int`
- `plan:internal/machinecli#Deps` -> `type Deps struct`
- `plan:internal/machinecli#productionDeps` -> `func productionDeps(cwd string) Deps`
- `plan:internal/machinecli#runWith` -> `func runWith(deps Deps, out io.Writer, args []string) int`
- `plan:internal/machinecli#hubsToSweep` -> `func hubsToSweep(deps Deps) ([]string, error)`
- `plan:internal/machinecli#Shown` -> `type Shown struct`
- `plan:internal/machinecli#show` -> `func show(deps Deps) Shown`
- `plan:internal/machinecli#SetResult` -> `type SetResult struct`
- `plan:internal/machinecli#setName` -> `func setName(deps Deps, name string) (SetResult, error)`
- `plan:internal/machinecli#hubBoard` -> `type hubBoard struct`
- `plan:internal/machinecli#readBoards` -> `func readBoards(deps Deps, hubs []string) ([]hubBoard, error)`
- `plan:internal/machinecli#checkCollision` -> `func checkCollision(name string, own []string, boards []hubBoard) error`
- `plan:internal/machinecli#rewriteFile` -> `func rewriteFile(file machinename.File, name, hostnameName string) (machinename.File, bool)`
- `internal/machinecli/testmain_test.go`
- `internal/machinecli/name_test.go`
- `plan:internal/machinecli#TestMain` -> `func TestMain(m *testing.M)`
- `plan:internal/machinecli#TestName` -> `func TestName(t *testing.T)`

**Edit:**
- `cmd/lym#newRoot`
- `cmd/lym/seamsignature_test.go`
- `docs/overview.md`

**Extends:**
- `pattern/PATTERN-cli-cobra.md`

**Uses:**
- `internal/machinename#FilePath`
- `internal/machinename#ReadFile`
- `internal/machinename#Names`
- `internal/machinename#CheckName`
- `internal/boardengine#ReadOwners`
- `internal/fabricengine#Bolt.Pull`
- `internal/fabricengine#NewBolt`
- `internal/fabricengine#BoltDir`
- `internal/fabricengine#BoardDir`
- `internal/hubregistry#Live`
- `internal/standalonestate#UserStateDir`
- `internal/lymcwd#Resolve`
- `internal/state#WriteJSON`

**Intent:**
Add the module `machine`, package `internal/machinecli`, with `lym machine name [<name>]` showing the name and setting it with D5's collision check and file write; the owner sweep follows on card 4.

Module shape per `PATTERN-cli-cobra`: `Command()` returns the `machine` group, `RunE: clihelp.GroupRunE`, holding the leaf `name [<name>]` with `cobra.MaximumNArgs(1)`, a lower-case `Short`, a `Long` with examples under the byte cap, and the operator audience annotation.
Every `RunE` checks `clihelp.ShouldAbort` first; results go through `output.Ok` and refusals through `output.Err` with `clihelp.SetExit`.
`RunCLI` and `RunCLIIn` follow `internal/hibernatecli`'s seams; `RunCLIIn` seeds the cwd the verb reads to find the working directory's hub.
The module has no engine package, and is no standalone `cliwire` module, since it runs from any directory, inside a hub or not.

`Deps` is the seam D8 names: `FilePath func() (file, lock string, ok bool, err error)`, `HostnameName func() (string, error)`, `LiveHubs func() ([]string, error)`, `Cwd string`, `HubOf func(cwd string) (hub string, ok bool)`, `Pull func(hub string) (fabricengine.BoltSkip, string, error)` and `ReadOwners func(hub string) (boardengine.Owners, error)`.
`productionDeps` fills them from `machinename.FilePath`, `machinename.HostnameName`, `hubregistry.Live` over `standalonestate.UserStateDir` (hub paths of the live entries), the given cwd, `lymcwd.Resolve`'s `HubPath` (no hub when it errors), `fabricengine.NewBolt(fabricengine.BoltDir(hub)).Pull` with default `SyncOptions` and a `fabricengine.NewMutations(hub)` record, and `boardengine.ReadOwners(fabricengine.BoardDir(hub))`.
Read how the existing owner callers (`internal/hubgeom/dobbygeom.go`) pass a hub's board directory and match them.
`hubsToSweep` returns the live hubs plus the cwd's hub when absent from them, deduplicated and sorted; a registry read error is returned.

No argument: `show` writes nothing and reports `name`, `source` (`file` when the file exists, else `hostname`), `path` (empty with no config directory) and `former`, read through `machinename.Names`.
A bad file reports the file's error and `path` in place of `name` and `former`; the verb still exits 0, since showing is no write.
A relative config variable is reported the same way, as `FilePath`'s error.

With a name, `setName` runs D5's steps in order and writes nothing until the collision check passes:
1. `machinename.CheckName` refuses an invalid name with its rule as the way forward; reached by the test row `InvalidNameIsRefused` (`Osl_46`).
2. `FilePath` returning `ok` false is refused naming `LOCALAPPDATA`, `XDG_CONFIG_HOME` and `HOME` as the variables to set; reached by the row `NoConfigDirIsRefused`, whose `FilePath` seam reports `ok` false.
   A `FilePath` error (a relative variable) is returned as is.
3. The file is read through `machinename.ReadFile`, never `machinename.Name`, so a bad file is repaired.
   A file that decodes keeps its `names` and drops an invalid `name`; a file that does not decode is replaced and its names are lost, and the result says so.
4. This machine's names are the file's valid current name, its non-empty `names` entries and `HostnameName`'s name when it yields one; a hostname error adds none.
5. `readBoards` pulls each hub of `hubsToSweep` through `Pull`, then reads its owners through `ReadOwners`.
   A pull that errors or returns a non-empty skip leaves the hub checked against its local Bolt, and the result names it as checked locally only.
   A `ReadOwners` error is returned naming the hub.
6. `checkCollision` refuses a name that a board records as its prime owner or a beam project's owner and that is not one of this machine's names.
   The refusal names the board's hub and the owner, says the name belongs to another machine, and names the way forward: choose another name.
   For a name this machine held under a file lost to a decode failure, it also names writing `{"name": "<name>"}` to the file's path by hand.
   Reached by the row `NameOwnedElsewhereIsRefused`.
7. `rewriteFile` builds the new file: `name` becomes the new name; the old valid name and the hostname rule's name are appended to `names` when absent; every empty entry is dropped, which the result reports; the new name moves to the end of `names`.
   The file is written through `state.WriteJSON` with `FilePath`'s lock path, only when the rewrite changes it, so a re-run with the name already set writes nothing new.

`SetResult` carries the name, the path, `names` after the write, whether the file was written, the dropped empty entries, a lost-file note and the hubs checked locally only.

`TestName` in `internal/machinecli/name_test.go` (tier0) is one table over `runWith` with `Deps` faked in memory.
Each row hands in a file path under `t.TempDir()`, a fixed hostname name and fake hubs whose `Pull` and `ReadOwners` answer from maps, so no row writes the developer's machine file or reaches a Bolt.
The rows run through `runWith`, which builds the command over the given deps; `RunCLIIn` calls it with `productionDeps(cwd)`, so every row keeps `t.Parallel`.
Rows: the show with a good file, with no file, and with a bad file; `osl46` over `osl-1046` writing `names` `osl-1046` then `osl46`; `InvalidNameIsRefused`; `NoConfigDirIsRefused`; `NameOwnedElsewhereIsRefused` for `neve`, from a board's owners, writing nothing.
Then: an invalid `name` with `names` kept plus the hostname rule's name; an undecodable file giving `names` hostname name then `osl46`; a hand-broken `name` whose `names` ends in `osl46` while a board records `osl46`, which succeeds; a file holding only `{"name":"osl46"}` recording the hostname rule's name in `names`; an empty `names` entry dropped and reported; and a pull skip reported as checked locally only.
Each row asserts the exit code, the envelope's fields and the file's bytes after the run, the unchanged bytes for a refusal.
`internal/machinecli/testmain_test.go` is untagged and holds `TestMain` calling `tmuxkit.Main`, as the other cli packages' do.

`cmd/lym#newRoot` mounts `machinecli.Command()` and adds `machine` to the `Long` text's module list.
`cmd/lym/seamsignature_test.go` gains `machinecli.RunCLI` and `machinecli.RunCLIIn` in its two pin lists.
`docs/overview.md` gains the package's tree line beside `internal/hubregistry/` and a module bullet beside the hibernate one.
`pattern/PATTERN-cli-cobra.md`'s Package naming deviations gain `machinecli`, which calls the functions of `machinename`, `boardengine`, `fabricengine`, `hubregistry` and `state`, with no engine package of its own.
`internal/machinecli/doc.go` documents the module: its verb, its deps seam, the steps above and the bounds D5 states.

**ImpactSummary:** a new module mounted under the root; cmd/lym's root text, seam pins and the overview doc gain one entry each.

**Commit:** 3: lym machine name shows and sets the machine name with a collision check across hubs
