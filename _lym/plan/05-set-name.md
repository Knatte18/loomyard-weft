# Card 5 — `lym machine name <name>` sets the name and sweeps owners

**Create:**
- `internal/machinecli/setname.go`
- `internal/machinecli/setname_test.go`
- `internal/machinecli/name_integration_test.go`
- `plan:internal/machinecli#fileState` -> `type fileState struct`
- `plan:internal/machinecli#readFileState` -> `func readFileState(d deps, path string) (fileState, error)`
- `plan:internal/machinecli#nextFile` -> `func nextFile(state fileState, name string) (next machinename.File, notes []string, changed bool)`
- `plan:internal/machinecli#checkCollision` -> `func checkCollision(d deps, hubs []hub, name string, own []string) (localOnly []string, err error)`
- `plan:internal/machinecli#setName` -> `func setName(d deps, cwd, name string) (fields map[string]any, clean bool, err error)`
- `plan:internal/machinecli#TestNextFile` -> `func TestNextFile(t *testing.T)`
- `plan:internal/machinecli#TestNameSet` -> `func TestNameSet(t *testing.T)`
- `plan:internal/machinecli#TestNameIntegration_RenameSweepsOwners` -> `func TestNameIntegration_RenameSweepsOwners(t *testing.T)`

**Edit:**
- `plan:internal/machinecli#deps`
- `plan:internal/machinecli#productionDeps`
- `plan:internal/machinecli#nameCmd`

**Extends:**
- `internal/machinecli/doc.go`
- `docs/overview.md`

**Uses:**
- `plan:internal/machinecli#sweptHubs`
- `plan:internal/machinecli#sweepHubs`
- `plan:internal/machinecli#recoveries`
- `plan:internal/machinecli#hub`
- `plan:internal/machinename#HostnameName`
- `internal/machinename#File`
- `internal/machinename#ReadFile`
- `internal/machinename#CheckName`
- `internal/state#WriteJSON`
- `internal/fabricengine#Bolt.Pull`
- `internal/fabricengine#EnvSyncOptions`
- `internal/fabricengine#NewMutations`
- `internal/boardengine#ReadOwners`
- `internal/boardengine#Owners`
- `internal/lymcwd#CwdFrom`
- `internal/hubforge#CopyHub`
- `internal/hubforge#CloneSecond`
- `internal/hubforge#Hub.BoardDir`
- `internal/testkit/boardkit#InstallReplay`
- `internal/boardengine#SetOwners`

**Intent:** Make `lym machine name <name>` validate the name, check it against every swept board, write the machine file and run the sweep (D5, D6).
`nameCmd` drops card 3's refusal of a name and calls `setName` with the cwd from `lymcwd.CwdFrom`.

`deps` gains two seams, and `productionDeps` fills them:

- `Pull func(boardDir string) (skipDetail string, err error)`: `fabricengine.NewBolt(boardDir).Pull` with `fabricengine.EnvSyncOptions()`;
  a non-empty skip comes back as its detail.
- `ReadOwners func(boardDir string) (boardengine.Owners, error)`: `boardengine.ReadOwners`.

`setName` runs these steps in order and stops at the first refusal, having written nothing:

1. `machinename.CheckName(name)`: an invalid name is refused with the rule's own error, which names the way forward.
   Reached by `lym machine name Bad_Name`; `TestNameSet`'s invalid-name row.
2. `d.FilePath()`: a relative variable is refused with `FilePath`'s error.
   With no directory resolved it refuses: the machine file has no directory to live in;
   way forward: set `HOME` to an absolute path, or `XDG_CONFIG_HOME`, or `LOCALAPPDATA` on Windows.
   Reached by a `FilePath` seam answering not ok, and one answering an error; `TestNameSet`'s two rows.
3. `readFileState`: the file through `machinename.ReadFile`, never `machinename.Name`, and the hostname rule's name through `d.HostnameName`.
   `fileState` holds the decoded file, whether it exists, whether it decoded, whether its `name` passes `CheckName`, and the hostname rule's name, empty when the hostname yields none.
   A file that exists and does not decode is a state, not an error; any other read error is returned.
   This machine's own names are: the current name, which is the file's valid `name`, or the hostname rule's name when no file exists;
   every non-empty entry of a decoded file's `names`; and the hostname rule's name when there is one, over a bad file too.
   An invalid `name` of a decoded file is not among them.
4. `sweptHubs(d, cwd)`, then `checkCollision`: for each hub one `d.Pull`, then one `d.ReadOwners`.
   A hub whose pull returns an error or a skip is read from its local Bolt and named in `localOnly`, which the output lists as checked locally only.
   A failed `ReadOwners` is returned as it is.
   The name is refused when any board records it as an owner, the prime's or any entry's, done entries included, and it is not one of this machine's own names:
   `the name <name> belongs to another machine: the board of <hub path> records it as the owner of <entry or the prime>`.
   Its way forward: choose another name; or, when the name was this machine's and the machine file lost it, write `{"name": "<name>"}` to the file's path by hand.
   Reached by a valid new name a swept board records as an owner; `TestNameSet`'s collision rows and the integration test's origin-held owner.
5. `nextFile`, pure, gives the file to write, the notes for the output and whether it differs from what is on disk:
   `name` becomes the new name;
   `names` starts from the decoded file's `names`, or empty for no file and for a file that does not decode;
   the old valid name and the hostname rule's name are each appended when absent;
   every empty entry is dropped, with a note saying how many;
   the new name is then moved to the end.
   A file that does not decode adds a note that it was replaced and its names are lost.
   A decoded file with an invalid `name` adds a note that the invalid name was dropped.
6. The file is written with `state.WriteJSON` at the path and lock `d.FilePath` gave, only when `changed` is true.
   A run with the name already set therefore writes only when the rewrite changes `names`.
7. `sweepHubs(d, hubs, name, replace)`, always, so a re-run finishes a sweep a hub failed.
   `replace` is every entry of the written `names` other than the new name.

The output's fields: `name`, `file`, `names`, `written`, `notes`, `checked_locally_only`, `hubs` with each hub's sweep, and `recoveries`.
With `clean` true the envelope is `output.Ok`.
Otherwise it is `output.ErrFields` with the same fields, an error naming the hubs that failed or lost entries and the re-run as the way forward, and a non-zero exit.

What this lets through: a process that renames the machine and rewrites owners on every recorded hub's board.
Bounds: it writes only the file at the path `FilePath` gives and only owners recorded under this machine's former names;
a name a board records for another machine is refused before any write;
an owner pushed to origin between the pull and the file write, and an owner only origin holds on a hub whose pull failed, are not seen by the check,
and the replace set still keeps the write off them.

Tests:

- `TestNextFile` in `internal/machinecli/setname_test.go`, tier0 and parallel, a table asserting the whole file, the notes and `changed`:
  `osl46` on `osl-1046` with no file gives `names` `osl-1046` then `osl46`;
  a hand-written file holding only `osl46`, named `osl46` again, gains the hostname rule's name and reads changed;
  the name already set and last in `names` reads unchanged;
  an empty `names` entry is dropped with its note;
  an invalid `name` keeps the non-empty `names` and adds the hostname rule's name;
  a file that does not decode gives the hostname rule's name then `osl46`;
  no hostname name adds none;
  a rename back to a former name moves it to the end.
- `TestNameSet` in the same file, tier0 and parallel, over `newCommand` with fake `deps` and a file under `t.TempDir()`.
  Refusal rows assert the envelope, the exit code, the file's bytes unchanged and no `SetOwners` call:
  an invalid name; no config directory; a relative variable; a board recording the name under no name of this machine; a prime owner colliding.
  Passing rows assert the file on disk, the replace set each hub's write got and the envelope:
  a plain rename; a hub whose pull failed, listed as checked locally only and still swept;
  a hand-broken `name` whose `names` ends in `osl46` while a board records `osl46`, which succeeds;
  the name already set with nothing to change, which writes no file and still sweeps;
  a hub whose sweep fails, exiting non-zero with the other hub swept, and a second run with the same name sweeping it;
  the recovery lines and the lift-after-sweep advice present after a rename.
- `TestNameIntegration_RenameSweepsOwners` in `internal/machinecli/name_integration_test.go`, tier1 and parallel, with `boardkit.InstallReplay`.
  It builds a hub with `hubforge.CopyHub` and a second clone with `hubforge.CloneSecond`, both under the test's temporary directory.
  Through `boardengine.SetOwners` on the first clone it records the prime and an open beam under `osl-1046`, and a done beam under `osl-1046`.
  From the second clone it records a second open beam under `osl-1046` and an open beam under `far`, so only origin holds those two owners before the verb runs.
  It runs `newCommand` with production board seams, a machine file path under the temporary directory, the hub as the one registry hub, and `osl-1046` as the hostname rule's name.
  `lym machine name far` is refused, naming the hub and the beam, and writes nothing: the check sees an owner only origin held.
  `lym machine name osl46` then leaves origin reading `osl46` for the prime and both open beams, `osl-1046` for the done beam and `far` for the other machine's beam.
  The same test runs the cwd seam's production function from a directory inside the hub and from a directory outside any, asserting the hub path and the miss.

No test writes a file outside its temporary directory: every one hands in the machine file's path and its hubs through `deps` (D8).

`internal/machinecli/doc.go` gains the set path: the steps, the refusals, the file rule, the repair of a bad file and the bounds D5 and D6 state on shared and lost names.
`docs/overview.md`'s `machine` entry gains the set form and the sweep.

**ImpactSummary:** Turns `lym machine name <name>` from a refusal into the rename: it writes the per-user machine file and one owner commit per swept hub, and nothing else calls the new functions.

**Commit:** 5: machine: lym machine name sets the name and sweeps owners
