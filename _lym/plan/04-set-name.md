# Card 4 — set-name

**Edit:**
- `plan:internal/machinecli#nameDeps`
- `plan:internal/machinecli#productionDeps`
- `plan:internal/machinecli#commandWith`
- `internal/machinecli/doc.go`

**Create:**
- `plan:internal/machinecli#setName` -> `func setName(cmd *cobra.Command, deps nameDeps, name string)`
- `plan:internal/machinecli#hubsToSweep` -> `func hubsToSweep(deps nameDeps) ([]string, error)`
- `plan:internal/machinecli#checkCollision` -> `func checkCollision(hubs []string, own []string, name string) (localOnly []string, err error)`
- `plan:internal/machinecli#rewriteNames` -> `func rewriteNames(file machinename.File, name, hostname string) (next machinename.File, droppedEmpty bool)`
- `plan:internal/machinecli#TestName_Set` -> `func TestName_Set(t *testing.T)`
- `internal/machinecli/name_integration_test.go`
- `plan:internal/machinecli#TestName_CollisionCheck` -> `func TestName_CollisionCheck(t *testing.T)`

**Uses:**
- `internal/machinename#ReadFile`
- `internal/machinename#CheckName`
- `internal/state#WriteJSON`
- `internal/hubregistry#Live`
- `internal/standalonestate#UserStateDir`
- `internal/lymcwd#ResolveWorktree`
- `internal/fabricengine#NewBolt`
- `internal/fabricengine#Bolt.Pull`
- `internal/fabricengine#EnvSyncOptions`
- `internal/fabricengine#NewMutations`
- `internal/fabricengine#BoardDir`
- `internal/boardengine#ReadOwners`
- `internal/hubforge#CopyHub`
- `internal/hubforge#CloneSecond`
- `internal/testkit/boardkit#InstallReplay`
- `plan:internal/machinecli#TestName_Show`

**Intent:** Let `lym machine name <name>` set the name: validate it, refuse a name another machine's owners hold, and rewrite the machine file keeping every name (D5).
The owner sweep is the next card's; after this card the verb writes the file alone.

`nameDeps` gains `Hubs`, filled from `hubregistry.Live` over `standalonestate.UserStateDir()` and returning the registered hubs' paths, and `Cwd`, the working directory `productionDeps` is given.
`commandWith` routes the leaf with an argument to `setName`.

`setName` runs, in order, and writes nothing on any refusal:
- It refuses a name `machinename.CheckName` refuses, printing the rule as the way forward.
- It refuses to write when the seam's `FilePath` resolves no directory, or returns a relative variable's error, naming the variables the per-user config directory reads.
- It reads the file at the seam's path with `machinename.ReadFile`.
  A file whose name fails `CheckName` keeps its `names` and drops its name.
  A file that does not decode is replaced: it counts as an empty file, and the output says its names are lost.
- This machine's own names are the file's valid name, else the hostname rule's name when no file exists, every non-empty `names` entry, and the hostname rule's name from `HostnameName`.
  A hostname that yields no name adds none.
- `hubsToSweep` returns `Hubs`'s paths plus the hub holding `Cwd` when `lymcwd.ResolveWorktree` places it in one, each once.
- `checkCollision` pulls each hub's Bolt through `fabricengine.NewBolt(fabricengine.BoardDir(hub)).Pull` with `EnvSyncOptions` and `NewMutations(hub)`.
  It then reads the hub's owners with `boardengine.ReadOwners`.
  A hub whose pull returns an error or a skip is checked against its local Bolt and returned in `localOnly`, which the output names as checked locally only.
  A name that a board records as a beam project's owner or as the prime owner, and that is not one of this machine's names, is refused.
  The refusal names the hub, the entry or the prime, and the owner, as another machine's name.
  When the refused name was one of the file's names lost to an undecodable file, the refusal also names the way forward D5 gives:
  writing `{"name": "<name>"}` to the file's path by hand.
- `rewriteNames` builds the file to write from the file read:
  `name` becomes the new name;
  the old valid name and the hostname rule's name are each appended to `names` when absent;
  every empty entry is dropped, reported through `droppedEmpty`;
  the new name is then moved to the end of `names`.
- It writes the result through `state.WriteJSON` at the seam's file and lock paths only when it differs from the file read, so a run with the name already set and `names` unchanged writes nothing.
  The output prints the name, the former names, whether the file was written, any dropped empty entry, any lost names and the hubs checked locally only.

`internal/machinecli/doc.go` gains the set form, the collision check and its bounds, and the file rewrite.

`TestName_Set`, tier0 beside `TestName_Show` and driving `commandWith` with fake deps, no hubs and a file path under `t.TempDir()`, covers the file rows of the acceptance criteria:
- no file on a machine whose hostname rule names `osl-1046`, renamed to `osl46`, writes `osl46` with `names` `osl-1046` then `osl46`;
- an invalid name is refused and writes nothing;
- no directory and a relative variable are each refused and write nothing;
- a file holding an invalid name keeps its non-empty `names`, adds the hostname rule's name when absent and writes the new name;
- an undecodable file writes the new name with `names` holding the hostname rule's name then `osl46`, and says the names are lost;
- a hand-written `{"name": "osl46"}` records the hostname rule's name in `names`;
- a `names` holding an empty entry drops it and says so;
- a re-run with the name set and `names` unchanged leaves the file's bytes and modification time as they were.

`internal/machinecli/name_integration_test.go` is tier1.
Its `TestName_CollisionCheck` builds hubs under its temporary directory with `hubforge.CopyHub`, after `boardkit.InstallReplay`.
A second clone from `hubforge.CloneSecond` records owners at origin only, and the test hands the hub's path in through `Hubs`.
Its rows:
- `neve`, recorded at origin only as a beam project's owner, is refused, naming the hub and owner, and the file is unchanged;
- `osl46` over a file whose `name` is hand-broken and whose `names` ends in `osl46` succeeds while the board records `osl46`;
- a former name the board records passes;
- a hub whose origin URL is unreachable is checked locally and named so in the output.

**ImpactSummary:** Extends the new module's own seams and command; nothing outside `internal/machinecli` changes.
